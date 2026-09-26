# Architectural Decision Records

Este diretório guarda os ADRs da feature de Webhooks de Notificação de Pedidos. Cada decisão fica em um arquivo separado, no formato `ADR-NNN-titulo-em-kebab-case.md`, seguindo uma variante em português do MADR (Status, Contexto, Decisão, Alternativas Consideradas e Consequências).

| ADR | Decisão | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-outbox-no-mysql.md) | Padrão Outbox no MySQL existente | Aceito |
| [ADR-002](ADR-002-worker-separado-com-polling.md) | Worker em processo separado lendo a outbox por polling de 2s | Aceito |
| [ADR-003](ADR-003-retry-com-backoff-e-dlq.md) | Retry com backoff 1m/5m/30m/2h/12h e DLQ em tabela separada | Aceito |
| [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint e rotação com 24h de carência | Aceito |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com deduplicação por X-Event-Id | Aceito |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões existentes do projeto | Aceito |
| [ADR-007](ADR-007-payload-enxuto-como-snapshot-na-insercao.md) | Payload enxuto renderizado como snapshot na inserção | Aceito |
