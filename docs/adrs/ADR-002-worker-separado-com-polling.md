# ADR-002: Worker em processo separado lendo a outbox por polling

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos), Marcos (PM)
**Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Contexto

Com a outbox definida (ADR-001), falta decidir como os eventos saem dela. O requisito de negócio é que o cliente seja avisado em menos de 10 segundos, que é o que os clientes consideram "tempo real" ([09:02] Marcos).

Hoje a aplicação tem um único entry point, `src/server.ts`, que sobe a API Express e trata `SIGINT` e `SIGTERM` para desligar com calma. Não existe nenhum processo em background no projeto.

## Decisão

1. O worker lê a outbox por **polling em loop, a cada 2 segundos**, pegando os eventos pendentes mais antigos em lotes pequenos ([09:09] Diego, [09:08] Diego). No pior caso a latência mínima fica em 2 segundos, e isso foi aceito ([09:10] Larissa, [09:10] Marcos).
2. O worker roda como **processo separado da API** ([09:11] Diego). Se a API reiniciar, o worker não cai junto. O entry point novo, a ser criado, é `src/worker.ts`, espelhando o `src/server.ts`, com um script `npm run worker` ([09:11] Larissa). A lógica de processamento fica dentro do módulo, em `src/modules/webhooks/webhook.worker.ts`, também novo ([09:28] Bruno).
3. O worker usa o mesmo banco e a mesma `DATABASE_URL`, mas com uma instância própria de `PrismaClient`, porque o client é por processo ([09:11] Bruno, [09:30] Bruno). Na prática, `src/worker.ts` importa `prisma` de `src/config/database.ts`, o que já cria uma instância nova naquele processo.
4. Nesta fase roda **um único worker**. Com isso a ordem de envio segue o `created_at` da outbox e, na prática, fica ordenada por pedido. Não existe garantia de ordem global. Isso fica documentado como limitação conhecida ([09:12] Diego, [09:13] Larissa).

## Alternativas Consideradas

**Trigger no banco para ser mais reativo.** Sugerido pelo Bruno ([09:09] Bruno). O MySQL não tem um mecanismo de listener como o `LISTEN/NOTIFY` do Postgres. A trigger só executa SQL e não avisa processo externo. Para acordar o worker seria preciso improvisar, escrevendo em arquivo ou chamando um endpoint ([09:09] Diego). Descartado porque o polling de 2s já cumpre a meta de 10s.

**Worker rodando dentro do processo da API.** Descartado porque um restart ou deploy da API derrubaria o processamento junto ([09:11] Diego).

**Vários workers em paralelo.** Adiado. Para escalar no futuro seria preciso particionar por `order_id` ou usar lock pessimista, e isso foi tratado como problema do futuro ([09:13] Diego).

## Consequências

**Positivas**
- Implementação simples, sem dependência nova e sem truque no banco.
- Latência previsível: 2s de intervalo mais o tempo do envio, bem abaixo dos 10s pedidos.
- API e worker podem ser reiniciados e publicados de forma independente.

**Negativas**
- Consulta ao banco a cada 2 segundos, mesmo quando não há nada para enviar.
- Um segundo processo para subir, monitorar e manter vivo em produção.
- Com um worker só, a vazão é limitada e não há redundância. Se o worker parar, os eventos se acumulam na outbox até ele voltar (sem perda, mas com atraso).

**Trade-off:** trocamos reatividade instantânea e escala horizontal por simplicidade operacional. Os 2 segundos de latência mínima foram aceitos explicitamente pelo PM.
