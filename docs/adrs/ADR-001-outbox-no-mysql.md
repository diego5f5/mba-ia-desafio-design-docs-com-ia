# ADR-001: Padrão Outbox no MySQL existente

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)
**Relacionados:** [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-payload-enxuto-como-snapshot-na-insercao.md)

## Contexto

Três clientes B2B precisam saber quando o status dos pedidos deles muda. Hoje eles fazem polling no `GET /orders` ([09:00] Marcos). A primeira pergunta da reunião foi onde disparar essa notificação: direto no service de pedidos, de forma síncrona, ou por algum mecanismo assíncrono ([09:03] Larissa).

A mudança de status já acontece dentro de uma transação pesada. O método `changeStatus` em `src/modules/orders/order.service.ts` roda em `this.prisma.$transaction(...)` e, dentro dela, valida a transição (`canTransition` em `src/modules/orders/order.status.ts`), debita ou repõe estoque, atualiza `orders` e grava em `order_status_history`. Colocar uma chamada HTTP para o cliente dentro disso faria qualquer cliente lento segurar a transação e travar mudanças de status de outros pedidos ([09:04] Bruno). E se o cliente estiver fora do ar, não faz sentido desfazer a mudança de status por causa disso ([09:04] Bruno).

Ao mesmo tempo, precisamos de uma garantia forte: se o status mudou, o evento tem que existir. Se a transação deu rollback, o evento não pode existir.

## Decisão

Usar o padrão Outbox no MySQL que a aplicação já usa ([09:06] Diego, [09:08] Larissa).

Dentro da mesma transação que atualiza `orders` e `order_status_history`, inserimos uma linha na nova tabela `webhook_outbox` com o evento. Um worker separado (ver ADR-002) lê essa tabela e faz as chamadas HTTP. Se a inserção na outbox falhar, a transação inteira dá rollback. Não pode existir status alterado sem evento correspondente ([09:40] Bruno, [09:41] Diego).

A tabela tem índice no campo de status (pendente, processando, falhou, entregue) e em `created_at` ([09:08] Diego). O id segue o padrão do projeto, UUID em `Char(36)` ([09:51] Larissa), igual a todos os modelos de `prisma/schema.prisma`.

O filtro de eventos por webhook é aplicado na hora da inserção. Se nenhum webhook ativo do cliente quer ouvir aquele status, nenhuma linha é criada ([09:34] Bruno, [09:34] Diego).

## Alternativas Consideradas

**Disparo síncrono dentro do `changeStatus`.** Descartado logo no início. Acopla a latência e a disponibilidade do cliente à transação de pedidos, e não tem resposta boa para o caso de cliente fora do ar ([09:04] Bruno, [09:06] Diego).

**Redis Streams (ou fila parecida).** Levantado pela Larissa como alternativa ([09:07] Larissa). Resolveria o problema de desacoplamento, mas exigiria subir e operar infraestrutura nova, e não daria a atomicidade com a transação do MySQL sem um mecanismo extra. Para um time pequeno isso foi considerado overengineering ([09:07] Diego).

## Consequências

**Positivas**
- Atomicidade real entre mudança de status e registro do evento, sem coordenação distribuída.
- Nenhuma infraestrutura nova. Mesmo banco, mesmo Prisma, mesmo backup.
- A outbox também vira a fonte do histórico de entregas que o cliente vai consultar.

**Negativas**
- A transação de `changeStatus` fica um pouco mais longa, com uma leitura dos webhooks do cliente e uma escrita a mais.
- A tabela cresce sem parar. O arquivamento de linhas entregues (algo como 30 dias) ficou fora do escopo desta feature ([09:08] Diego).
- O banco de pedidos passa a carregar também a carga de leitura do worker.

**Trade-off:** aceitamos colocar mais carga e mais uma tabela no MySQL de produção em troca de consistência garantida e de zero infraestrutura nova.
