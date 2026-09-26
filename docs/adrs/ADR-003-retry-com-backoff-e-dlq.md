# ADR-003: Retry com backoff (1m/5m/30m/2h/12h) e DLQ em tabela separada

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos), Marcos (PM), Sofia (Segurança)
**Relacionados:** [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Contexto

O endpoint do cliente pode estar lento, fora do ar ou em manutenção. Precisamos definir quantas vezes tentamos de novo, com qual intervalo e o que acontece quando desistimos ([09:14] Larissa).

Um dado de realidade pesou na discussão: já tivemos cliente com indisponibilidade de duas horas por manutenção planejada ([09:16] Diego).

## Decisão

1. **Backoff com 5 tentativas e intervalos fixos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas** ([09:17] Diego, [09:17] Larissa). A janela total entre a primeira falha e a última tentativa fica em quase 15 horas, o que o PM considerou aceitável ([09:17] Marcos).
   - Interpretação adotada: são 5 **retentativas** depois do envio inicial. É o que fecha com a conta do Diego (1 + 5 + 30 + 120 + 720 minutos, cerca de 14h36, "quase 15 horas"). Se fossem 5 envios no total, só caberiam 4 intervalos e a janela seria de cerca de 2h36.
2. Esgotadas as tentativas, o evento vira **falha permanente e vai para a DLQ** ([09:15] Diego).
3. A DLQ é uma **tabela separada, `webhook_dead_letter`**, com o payload, o motivo da falha e o timestamp ([09:18] Diego). A linha correspondente na outbox fica com status de falha.
4. O reprocessamento é **manual, por um endpoint administrativo** `POST /admin/webhooks/dead-letter/:id/replay`, que recoloca o evento na outbox como pendente ([09:18] Diego, [09:35] Diego).
5. O replay exige **role ADMIN**, reaproveitando o `requireRole` de `src/middlewares/auth.middleware.ts`, e registra quem fez o replay para auditoria ([09:36] Sofia, [09:36] Larissa).
6. Uma tentativa que não recebe resposta em **10 segundos** conta como falha e entra no ciclo de retry ([09:42] Diego).

## Alternativas Consideradas

**Retry indefinido com backoff.** Descartado porque o evento pode ficar pendurado para sempre se o cliente sumir ([09:15] Diego).

**3 tentativas, mais agressivo.** Proposto pelo Bruno ([09:16] Bruno). Descartado porque uma indisponibilidade de manhã esgotaria as três tentativas em uns 30 minutos, e já temos histórico de manutenção de duas horas ([09:16] Diego).

**Marcar como "failed" dentro da própria outbox, sem tabela de DLQ.** Colocado em discussão pela Larissa ([09:17] Larissa). Preterido porque a tabela separada deixa a leitura da outbox principal mais limpa e funciona como evidência para debug e reprocessamento ([09:18] Diego).

## Consequências

**Positivas**
- Cobre indisponibilidades de algumas horas sem intervenção manual.
- Todo evento termina em um estado final conhecido: entregue ou na DLQ.
- A DLQ dá material concreto para investigação e um caminho auditável de reprocessamento.

**Negativas**
- Uma falha pode levar quase 15 horas para chegar na DLQ, e durante esse tempo o evento fica em retry.
- Eventos que estão em backoff podem ser ultrapassados por eventos mais novos do mesmo pedido. A ordem por pedido só vale enquanto as entregas dão certo de primeira.
- O replay depende de alguém agir. Não existe aviso automático ao cliente, porque email ficou para uma próxima fase ([09:37] Larissa).

**Trade-off:** preferimos um teto de tentativas com janela longa e reprocessamento manual a um retry infinito. Ganhamos previsibilidade e um fim claro para cada evento, e aceitamos que parte das falhas vai precisar de um ADMIN.
