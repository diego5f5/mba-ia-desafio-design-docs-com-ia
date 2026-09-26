# ADR-007: Payload enxuto, renderizado como snapshot na inserção da outbox

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)
**Relacionados:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Contexto

Duas perguntas ficaram para o fim da reunião e definem o que exatamente vai na linha da outbox:

1. O que vai dentro do payload ([09:43] Marcos)?
2. A outbox guarda o payload já pronto ou guarda só o `order_id` e monta o JSON na hora do envio ([09:51] Bruno)?

A segunda pergunta ganha peso por causa do retry (ADR-003). Um evento pode ser enviado até 15 horas depois de criado, e o pedido pode mudar nesse intervalo.

## Decisão

1. **O payload é renderizado na inserção**, dentro da transação do `changeStatus`, e gravado pronto na outbox. Ele reflete o estado do pedido no momento em que o status mudou ([09:52] Larissa, [09:52] Diego, [09:52] Bruno).
2. **O payload é enxuto**: JSON com `event_id`, `event_type` (`order.status_changed`), `timestamp` em ISO 8601, `order_id`, `order_number`, `from_status`, `to_status`, `customer_id` e campos básicos do pedido como `total_cents`. **Os itens do pedido não vão**. Se o cliente precisar de detalhes, consulta `GET /orders/:id` ([09:43] Diego, [09:44] Bruno).
3. Todo payload tem teto de **64KB**. Se passar, o evento não é enviado e vira erro. Não truncamos ([09:23] Sofia, [09:24] Diego, [09:24] Larissa).

## Alternativas Consideradas

**Guardar só o `order_id` e montar o payload na hora do envio.** Colocada pelo Bruno ([09:51] Bruno). Descartada porque, se o pedido mudar depois, o evento de uma transição antiga chegaria com dados novos, o que gera casos esquisitos para o cliente ([09:52] Larissa).

**Payload completo, com itens.** Descartado para não inflar o payload. O detalhe já está disponível na API de pedidos ([09:43] Diego).

**Truncar payloads grandes.** Descartado pela Sofia. Se o evento chegou nesse tamanho, tem algo errado, e o correto é falhar ([09:23] Sofia).

## Consequências

**Positivas**
- O evento é fiel ao momento da transição, mesmo depois de retries e replays.
- Retry e replay não fazem consultas extras em `orders`. O worker só lê a outbox.
- A outbox não precisa de chave estrangeira para `orders`, então a exclusão de um pedido (permitida em `OrderService.delete` para `PENDING` e `CANCELLED`) não quebra eventos já registrados.
- Payload pequeno, bem longe do teto de 64KB.

**Negativas**
- Mudar o formato do payload não afeta eventos que já estão na outbox. Durante uma migração convivem dois formatos.
- Uma linha por evento e por webhook ocupa mais espaço do que guardar só o `order_id`.
- O cliente que precisar dos itens faz uma chamada extra à API.

**Trade-off:** gastamos um pouco mais de armazenamento para ter eventos imutáveis e previsíveis, e deixamos o detalhe do pedido na API que já existe.
