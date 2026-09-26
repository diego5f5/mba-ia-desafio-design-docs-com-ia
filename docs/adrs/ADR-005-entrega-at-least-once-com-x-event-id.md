# ADR-005: Garantia de entrega at-least-once com deduplicação por X-Event-Id

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Diego (Eng. Sênior, Plataforma), Larissa (Tech Lead), Sofia (Segurança), Marcos (PM)
**Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Contexto

Com outbox, worker e retry (ADRs 001 a 003), existem situações em que o mesmo evento pode sair mais de uma vez. Por exemplo: o cliente processou a requisição, mas a resposta demorou mais de 10 segundos e contamos como falha. Ou o worker enviou com sucesso e caiu antes de marcar a linha como entregue. Ou um ADMIN fez replay de um evento da DLQ que o cliente, na verdade, já tinha recebido.

A reunião precisava decidir qual garantia oferecer ao cliente e como ele trata duplicidade ([09:24] Diego, [09:25] Bruno).

## Decisão

1. A plataforma garante entrega **at-least-once**. O cliente pode receber o mesmo evento mais de uma vez e precisa estar preparado para isso ([09:24] Diego).
2. Todo envio leva o header **`X-Event-Id`** com um **UUID gerado no momento em que o evento entra na outbox**. Esse id é único por evento e não muda entre retentativas nem em replay ([09:25] Diego). O mesmo valor aparece no campo `event_id` do payload ([09:43] Diego).
3. A deduplicação é **responsabilidade do cliente**, usando o `event_id` ([09:25] Diego).
4. O PM vai documentar esse comportamento com destaque no portal do desenvolvedor ([09:26] Marcos).

## Alternativas Consideradas

**Exactly-once.** Descartado porque exigiria coordenação dos dois lados, a nossa plataforma e o sistema do cliente, e ficaria muito mais complexo. At-least-once com um id de evento resolve a grande maioria dos casos e é o que Stripe e GitHub fazem ([09:25] Diego).

**At-most-once, sem retry.** Não chegou a ser defendido na reunião, mas é a alternativa natural a considerar. Contradiz a política de retry já decidida (ADR-003) e deixaria o cliente sem saber de mudanças sempre que houvesse qualquer instabilidade.

## Consequências

**Positivas**
- Nenhum evento se perde por falha transitória. Na dúvida, reenviamos.
- O worker fica simples: não precisa de protocolo de confirmação em duas fases.
- O modelo é conhecido pelos desenvolvedores dos clientes, porque é o mesmo de provedores grandes.

**Negativas**
- Joga responsabilidade para o cliente, como a Sofia apontou ([09:25] Sofia). Integrações mal feitas podem processar o mesmo evento duas vezes.
- Exige documentação clara no portal, e isso depende do PM.

**Trade-off:** preferimos uma garantia simples e padrão de mercado, com a deduplicação do lado de quem recebe, a um protocolo exactly-once que custaria muito mais para construir e operar.
