# Tracker de Rastreabilidade

Esta tabela liga cada item registrado no PRD, no RFC, no FDD e nos ADRs à sua origem. Para a transcrição, a localização é o timestamp e o nome de quem falou, no formato `[hh:mm] Nome`, seguindo o `TRANSCRICAO.md`. Para o código, é o caminho do arquivo no repositório.

Quando um item tem mais de uma origem, a linha aponta a fala ou o arquivo principal, e as demais origens aparecem no próprio documento. Itens marcados como hipótese nos documentos também estão listados, com a origem que motivou a hipótese.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-CTX-01 | docs/PRD.md | Contexto | Três clientes B2B (Atlas, MaxDistribuição, Nova Cargo) pediram notificação de mudança de status | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-02 | docs/PRD.md | Problema | Clientes fazem polling em GET /orders, o que deixa a integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-03 | docs/PRD.md | Problema | Atlas pode migrar para concorrente se a feature não sair até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-CTX-04 | docs/PRD.md | Contexto | Usuários da API representam o cliente e cadastram os webhooks | TRANSCRICAO | [09:32] Marcos |
| PRD-CTX-05 | docs/PRD.md | Contexto | Feature implantada no OMS existente (Express, MySQL, Prisma) | CODIGO | package.json |
| PRD-CTX-06 | docs/PRD.md | Cenário de uso | Cliente com manutenção de horas não pode perder notificações | TRANSCRICAO | [09:16] Diego |
| PRD-CTX-07 | docs/PRD.md | Contexto | A plataforma não tem mecanismo de notificação externa | CODIGO | src/app.ts |
| PRD-OBJ-01 | docs/PRD.md | Objetivo / métrica | Entrega em menos de 10 segundos no p95 | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | docs/PRD.md | Objetivo / métrica | 3 de 3 clientes com webhook ativo em produção | TRANSCRICAO | [09:00] Marcos |
| PRD-OBJ-03 | docs/PRD.md | Objetivo / métrica | 100% das mudanças de status com webhook interessado geram evento | TRANSCRICAO | [09:40] Bruno |
| PRD-OBJ-04 | docs/PRD.md | Objetivo / métrica | Janela de retentativas de cerca de 15 horas | TRANSCRICAO | [09:17] Diego |
| PRD-OBJ-05 | docs/PRD.md | Objetivo / métrica (hipótese) | Queda nas chamadas de polling a GET /orders, linha de base a medir | TRANSCRICAO | [09:00] Marcos |
| PRD-ESC-01 | docs/PRD.md | Escopo | CRUD de webhooks com filtro de status | TRANSCRICAO | [09:33] Bruno |
| PRD-ESC-02 | docs/PRD.md | Escopo | Documentação de integração no portal do desenvolvedor | TRANSCRICAO | [09:40] Marcos |
| PRD-FORA-01 | docs/PRD.md | Fora de escopo | Email de aviso de falha adiado para próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-FORA-02 | docs/PRD.md | Fora de escopo | Dashboard visual é projeto do time de frontend | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-03 | docs/PRD.md | Fora de escopo | Rate limiting de saída fica para observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-FORA-04 | docs/PRD.md | Fora de escopo | Webhooks de entrada não fazem parte, só saída | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-05 | docs/PRD.md | Fora de escopo | Ordem global e múltiplos workers | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-06 | docs/PRD.md | Fora de escopo | Arquivamento de eventos após cerca de 30 dias | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-07 | docs/PRD.md | Fora de escopo | Itens do pedido fora do payload | TRANSCRICAO | [09:43] Diego |
| PRD-FORA-08 | docs/PRD.md | Fora de escopo | Endurecer permissões do CRUD fica para depois | TRANSCRICAO | [09:37] Sofia |
| PRD-FR-01 | docs/PRD.md | Requisito Funcional | Cadastrar webhook com URL e status, secret gerada e devolvida | TRANSCRICAO | [09:31] Marcos |
| PRD-FR-01a | docs/PRD.md | Restrição | customer_id não vem do JWT, vai na requisição | TRANSCRICAO | [09:32] Larissa |
| PRD-FR-02 | docs/PRD.md | Requisito Funcional | Listar webhooks de um cliente | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-03 | docs/PRD.md | Requisito Funcional | Editar webhook (PATCH) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-04 | docs/PRD.md | Requisito Funcional | Remover webhook (DELETE) | TRANSCRICAO | [09:33] Bruno |
| PRD-FR-05 | docs/PRD.md | Requisito Funcional | Filtro de eventos por lista de status | TRANSCRICAO | [09:33] Marcos |
| PRD-FR-05a | docs/PRD.md | Decisão | Filtro aplicado na inserção: sem interessado, nada é gravado | TRANSCRICAO | [09:34] Bruno |
| PRD-FR-06 | docs/PRD.md | Requisito Funcional | Notificar toda mudança de status aos webhooks interessados | TRANSCRICAO | [09:40] Bruno |
| PRD-FR-07 | docs/PRD.md | Requisito Funcional | Identificador de evento para deduplicação | TRANSCRICAO | [09:25] Diego |
| PRD-FR-08 | docs/PRD.md | Requisito Funcional | Assinar notificações com HMAC-SHA256 | TRANSCRICAO | [09:20] Sofia |
| PRD-FR-09 | docs/PRD.md | Requisito Funcional | Rotacionar secret com 24 horas de convivência | TRANSCRICAO | [09:21] Sofia |
| PRD-FR-10 | docs/PRD.md | Requisito Funcional | Retentar entregas que falharam | TRANSCRICAO | [09:15] Diego |
| PRD-FR-11 | docs/PRD.md | Requisito Funcional | DLQ com reprocessamento manual por administrador | TRANSCRICAO | [09:18] Diego |
| PRD-FR-12 | docs/PRD.md | Requisito Funcional | Histórico de entregas com resultado, payload, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-NFR-01 | docs/PRD.md | Requisito Não Funcional | Verificação da fila a cada 2 segundos, atraso mínimo aceito | TRANSCRICAO | [09:10] Larissa |
| PRD-NFR-02 | docs/PRD.md | Requisito Não Funcional | Timeout de 10 segundos por chamada | TRANSCRICAO | [09:42] Diego |
| PRD-NFR-03 | docs/PRD.md | Requisito Não Funcional | Nenhuma chamada HTTP dentro da transação de pedidos | TRANSCRICAO | [09:04] Bruno |
| PRD-NFR-04 | docs/PRD.md | Requisito Não Funcional | Worker em processo separado da API | TRANSCRICAO | [09:11] Diego |
| PRD-NFR-05 | docs/PRD.md | Requisito Não Funcional | URL obrigatoriamente https | TRANSCRICAO | [09:23] Sofia |
| PRD-NFR-06 | docs/PRD.md | Requisito Não Funcional | CRUD liberado para qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| PRD-NFR-07 | docs/PRD.md | Requisito Não Funcional | Replay exige papel ADMIN | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-08 | docs/PRD.md | Requisito Não Funcional | Limite de 64KB por notificação | TRANSCRICAO | [09:24] Larissa |
| PRD-NFR-09 | docs/PRD.md | Requisito Não Funcional | Ordem garantida só por pedido e com um único worker | TRANSCRICAO | [09:13] Larissa |
| PRD-NFR-10 | docs/PRD.md | Requisito Não Funcional | Logs com o logger já existente (Pino) | CODIGO | src/shared/logger/index.ts |
| PRD-NFR-11 | docs/PRD.md | Requisito Não Funcional | Auditoria de quem fez o replay | TRANSCRICAO | [09:36] Sofia |
| PRD-NFR-12 | docs/PRD.md | Requisito Não Funcional | Tipo de evento order.status_changed em JSON | TRANSCRICAO | [09:43] Diego |
| PRD-DEP-01 | docs/PRD.md | Dependência | Revisão de segurança da Sofia com pelo menos dois dias úteis | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-02 | docs/PRD.md | Dependência | Documentação no portal pelo PM | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-03 | docs/PRD.md | Dependência | Confirmação do prazo de fim de novembro com a Atlas | TRANSCRICAO | [09:45] Marcos |
| PRD-DEP-04 | docs/PRD.md | Dependência | Segundo processo Node em produção com acesso ao mesmo banco | TRANSCRICAO | [09:11] Bruno |
| PRD-DEP-05 | docs/PRD.md | Dependência | Clientes precisam de endpoint https, validar HMAC e deduplicar | TRANSCRICAO | [09:25] Diego |
| PRD-RISK-01 | docs/PRD.md | Risco | Atraso faz a Atlas migrar para concorrente | TRANSCRICAO | [09:00] Marcos |
| PRD-RISK-01a | docs/PRD.md | Mitigação | Estimativa de três sprints com revisão incluída | TRANSCRICAO | [09:46] Larissa |
| PRD-RISK-02 | docs/PRD.md | Risco | Cliente processar evento duplicado | TRANSCRICAO | [09:25] Sofia |
| PRD-RISK-03 | docs/PRD.md | Risco | Vazamento de secret de cliente | TRANSCRICAO | [09:22] Diego |
| PRD-RISK-04 | docs/PRD.md | Risco | Cliente fora do ar por mais de 15 horas | TRANSCRICAO | [09:17] Marcos |
| PRD-RISK-05 | docs/PRD.md | Risco | Rajada de notificações sem rate limiting | TRANSCRICAO | [09:38] Diego |
| PRD-RISK-06 | docs/PRD.md | Risco | Notificações fora de ordem para um mesmo pedido | TRANSCRICAO | [09:12] Diego |
| PRD-TEST-01 | docs/PRD.md | Estratégia de teste | Testes de integração com banco real em Vitest e Supertest | CODIGO | tests/orders.test.ts |
| PRD-TEST-02 | docs/PRD.md | Estratégia de teste | Revisão manual de segurança sobre HMAC e secret | TRANSCRICAO | [09:46] Sofia |
| RFC-META-01 | docs/RFC.md | Metadado | Revisores são os participantes da reunião | TRANSCRICAO | [09:50] Larissa |
| RFC-CTX-01 | docs/RFC.md | Contexto | Mudança de status acontece em transação pesada no changeStatus | CODIGO | src/modules/orders/order.service.ts |
| RFC-PROP-01 | docs/RFC.md | Proposta | Evento gravado na outbox dentro da transação de changeStatus | TRANSCRICAO | [09:06] Diego |
| RFC-PROP-02 | docs/RFC.md | Proposta | Worker separado, polling 2s, instância única | TRANSCRICAO | [09:09] Diego |
| RFC-PROP-03 | docs/RFC.md | Proposta | Retry 1m/5m/30m/2h/12h e DLQ com replay por ADMIN | TRANSCRICAO | [09:17] Larissa |
| RFC-PROP-04 | docs/RFC.md | Proposta | Secret por endpoint, HMAC-SHA256, https obrigatório | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-05 | docs/RFC.md | Proposta | At-least-once com X-Event-Id, X-Webhook-Id e X-Timestamp | TRANSCRICAO | [09:44] Diego |
| RFC-PROP-06 | docs/RFC.md | Proposta | Módulo novo reaproveitando padrões do projeto | TRANSCRICAO | [09:30] Larissa |
| RFC-ALT-01 | docs/RFC.md | Alternativa descartada | Disparo síncrono no changeStatus | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | docs/RFC.md | Alternativa descartada | Redis Streams ou fila dedicada | TRANSCRICAO | [09:07] Larissa |
| RFC-ALT-03 | docs/RFC.md | Alternativa descartada | Trigger no banco para acordar o worker | TRANSCRICAO | [09:09] Bruno |
| RFC-ALT-04 | docs/RFC.md | Alternativa descartada | Retry indefinido ou só 3 tentativas | TRANSCRICAO | [09:16] Bruno |
| RFC-ALT-05 | docs/RFC.md | Alternativa descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-06 | docs/RFC.md | Alternativa descartada | Montar payload na hora do envio | TRANSCRICAO | [09:51] Bruno |
| RFC-QA-01 | docs/RFC.md | Questão em aberto | Rate limiting de saída | TRANSCRICAO | [09:39] Diego |
| RFC-QA-02 | docs/RFC.md | Questão em aberto | Aviso por email em falhas repetidas | TRANSCRICAO | [09:37] Marcos |
| RFC-QA-03 | docs/RFC.md | Questão em aberto | Escala horizontal do worker (particionar por order_id ou lock) | TRANSCRICAO | [09:13] Diego |
| RFC-QA-04 | docs/RFC.md | Questão em aberto | Arquivamento da outbox | TRANSCRICAO | [09:08] Diego |
| RFC-QA-05 | docs/RFC.md | Questão em aberto | Endurecer permissões do CRUD | TRANSCRICAO | [09:37] Sofia |
| RFC-QA-06 | docs/RFC.md | Questão em aberto | customer_id no body ou no path | TRANSCRICAO | [09:32] Larissa |
| RFC-IMP-01 | docs/RFC.md | Impacto | Estimativa de três sprints | TRANSCRICAO | [09:46] Larissa |
| RFC-IMP-02 | docs/RFC.md | Impacto | Nenhuma dependência nova em package.json | CODIGO | package.json |
| ADR-001 | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Outbox no MySQL, atômica com a mudança de status | TRANSCRICAO | [09:08] Larissa |
| ADR-001-CTX | docs/adrs/ADR-001-outbox-no-mysql.md | Contexto | changeStatus roda em $transaction e valida com canTransition | CODIGO | src/modules/orders/order.service.ts |
| ADR-001-IDX | docs/adrs/ADR-001-outbox-no-mysql.md | Decisão | Índices em status e created_at | TRANSCRICAO | [09:08] Diego |
| ADR-001-TO | docs/adrs/ADR-001-outbox-no-mysql.md | Trade-off | Redis descartado como overengineering para time pequeno | TRANSCRICAO | [09:07] Diego |
| ADR-002 | docs/adrs/ADR-002-worker-separado-com-polling.md | Decisão | Worker em processo separado com polling de 2s | TRANSCRICAO | [09:10] Larissa |
| ADR-002-EP | docs/adrs/ADR-002-worker-separado-com-polling.md | Decisão | Entry point src/worker.ts espelhando src/server.ts e npm run worker | TRANSCRICAO | [09:11] Larissa |
| ADR-002-SRV | docs/adrs/ADR-002-worker-separado-com-polling.md | Contexto | Hoje só existe o entry point da API com shutdown por sinal | CODIGO | src/server.ts |
| ADR-002-PRISMA | docs/adrs/ADR-002-worker-separado-com-polling.md | Decisão | Instância própria de PrismaClient, mesma DATABASE_URL | TRANSCRICAO | [09:30] Bruno |
| ADR-002-DB | docs/adrs/ADR-002-worker-separado-com-polling.md | Restrição | PrismaClient criado por processo em database.ts | CODIGO | src/config/database.ts |
| ADR-002-ALT | docs/adrs/ADR-002-worker-separado-com-polling.md | Alternativa | Trigger descartada por falta de LISTEN/NOTIFY no MySQL | TRANSCRICAO | [09:09] Diego |
| ADR-003 | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | 5 retentativas em 1m/5m/30m/2h/12h | TRANSCRICAO | [09:17] Larissa |
| ADR-003-DLQ | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | DLQ em tabela webhook_dead_letter com payload, motivo e timestamp | TRANSCRICAO | [09:18] Diego |
| ADR-003-REPLAY | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Replay manual via POST /admin/webhooks/dead-letter/:id/replay | TRANSCRICAO | [09:35] Diego |
| ADR-003-ROLE | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Decisão | Replay com requireRole('ADMIN') existente | CODIGO | src/middlewares/auth.middleware.ts |
| ADR-003-ALT | docs/adrs/ADR-003-retry-com-backoff-e-dlq.md | Alternativa | Retry indefinido e 3 tentativas descartados | TRANSCRICAO | [09:16] Diego |
| ADR-004 | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | HMAC-SHA256 sobre o corpo, header X-Signature | TRANSCRICAO | [09:22] Sofia |
| ADR-004-SECRET | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Decisão | Secret por endpoint, nunca global | TRANSCRICAO | [09:21] Sofia |
| ADR-004-HASH | docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md | Trade-off | Secret não pode ser só hash, diferente do passwordHash de User | CODIGO | prisma/schema.prisma |
| ADR-005 | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Decisão | At-least-once com dedup pelo X-Event-Id | TRANSCRICAO | [09:26] Larissa |
| ADR-005-TO | docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md | Trade-off | Responsabilidade de dedup vai para o cliente | TRANSCRICAO | [09:25] Sofia |
| ADR-006 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Reuso máximo: AppError, Pino, error middleware, módulos, Zod | TRANSCRICAO | [09:30] Larissa |
| ADR-006-MOD | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Módulo em src/modules/webhooks com a estrutura padrão | TRANSCRICAO | [09:27] Bruno |
| ADR-006-ERR | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Erros herdam de AppError com prefixo WEBHOOK_ | CODIGO | src/shared/errors/app-error.ts |
| ADR-006-ERR2 | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Prefixo WEBHOOK_ para todo o módulo | TRANSCRICAO | [09:29] Larissa |
| ADR-006-MW | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Restrição | Middleware de erro já trata AppError, Zod e Prisma | CODIGO | src/middlewares/error.middleware.ts |
| ADR-006-PUB | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | publishWebhookEvent(tx, order, fromStatus, toStatus) em vez de injetar repository | TRANSCRICAO | [09:41] Bruno |
| ADR-006-UUID | docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md | Decisão | Ids em UUID como no resto do projeto | TRANSCRICAO | [09:51] Larissa |
| ADR-007 | docs/adrs/ADR-007-payload-enxuto-como-snapshot-na-insercao.md | Decisão | Payload renderizado como snapshot na inserção | TRANSCRICAO | [09:52] Larissa |
| ADR-007-FMT | docs/adrs/ADR-007-payload-enxuto-como-snapshot-na-insercao.md | Decisão | Campos do payload e ausência de items | TRANSCRICAO | [09:43] Diego |
| ADR-007-DEL | docs/adrs/ADR-007-payload-enxuto-como-snapshot-na-insercao.md | Consequência | Pedido pode ser apagado em PENDING e CANCELLED, então a outbox não tem FK para orders | CODIGO | src/modules/orders/order.service.ts |
| FDD-CTX-01 | docs/FDD.md | Contexto | Só changeStatus gera evento. create já nasce em PENDING | CODIGO | src/modules/orders/order.service.ts |
| FDD-CTX-02 | docs/FDD.md | Restrição | Nenhuma transição leva a PENDING, filtro aceita só 5 status | CODIGO | src/modules/orders/order.status.ts |
| FDD-CTX-03 | docs/FDD.md | Suposição (hipótese) | Resposta 2xx conta como sucesso, o resto é falha | TRANSCRICAO | [09:42] Diego |
| FDD-OBJ-01 | docs/FDD.md | Objetivo técnico | p95 abaixo de 10s entre commit e primeira entrega | TRANSCRICAO | [09:02] Marcos |
| FDD-OBJ-02 | docs/FDD.md | Invariante | Rollback de status desfaz o evento e vice versa | TRANSCRICAO | [09:06] Diego |
| FDD-DADOS-01 | docs/FDD.md | Modelo de dados | Tabela webhooks com url, secret, customer_id e ativo | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-02 | docs/FDD.md | Modelo de dados | Estados da outbox: pendente, processando, entregue, falhou | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-03 | docs/FDD.md | Modelo de dados | Padrão de modelo Prisma com UUID Char(36) e @@map | CODIGO | prisma/schema.prisma |
| FDD-CONF-01 | docs/FDD.md | Parâmetro | WEBHOOK_BATCH_SIZE com default 20 como hipótese para "batch pequeno" | TRANSCRICAO | [09:08] Diego |
| FDD-CONF-02 | docs/FDD.md | Parâmetro | Novas variáveis no envSchema com default | CODIGO | src/config/env.ts |
| FDD-FLUXO-01 | docs/FDD.md | Fluxo | Cadastro com secret gerada pela plataforma | TRANSCRICAO | [09:31] Marcos |
| FDD-FLUXO-02 | docs/FDD.md | Fluxo | Criação do evento na outbox dentro de changeStatus, rollback se falhar | TRANSCRICAO | [09:40] Bruno |
| FDD-FLUXO-03 | docs/FDD.md | Fluxo | Worker processa em ordem de created_at, um de cada vez | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-04 | docs/FDD.md | Fluxo | Retry com backoff pelo nextAttemptAt | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-05 | docs/FDD.md | Fluxo | Envio para DLQ com payload, motivo e data | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-06 | docs/FDD.md | Fluxo | Replay recoloca na outbox como pendente | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-07 | docs/FDD.md | Fluxo | Rotação com secret anterior válida por 24h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-08 | docs/FDD.md | Fluxo | Shutdown do worker por SIGINT e SIGTERM como no server.ts | CODIGO | src/server.ts |
| FDD-CONTRATO-01 | docs/FDD.md | Contrato | POST /api/v1/webhooks | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-02 | docs/FDD.md | Contrato | GET /api/v1/webhooks com customerId na query, igual order.schemas.ts | CODIGO | src/modules/orders/order.schemas.ts |
| FDD-CONTRATO-03 | docs/FDD.md | Contrato | PATCH /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-04 | docs/FDD.md | Contrato | DELETE /api/v1/webhooks/:id | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-05 | docs/FDD.md | Contrato | POST /api/v1/webhooks/:id/rotate-secret | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-06 | docs/FDD.md | Contrato | GET /api/v1/webhooks/:id/deliveries, até 100 por página | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-06a | docs/FDD.md | Contrato (hipótese) | deadLetterId no histórico para permitir o replay | TRANSCRICAO | [09:18] Diego |
| FDD-CONTRATO-07 | docs/FDD.md | Contrato | POST /api/v1/admin/webhooks/dead-letter/:id/replay | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-08 | docs/FDD.md | Contrato | Headers de saída X-Event-Id, X-Signature, X-Timestamp, Content-Type | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-08a | docs/FDD.md | Contrato | Header X-Webhook-Id | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-08b | docs/FDD.md | Contrato | Payload com event_id, event_type, timestamp, order_id, order_number, status, customer_id, total_cents | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-08c | docs/FDD.md | Contrato | total_cents e order_number vêm do modelo Order | CODIGO | prisma/schema.prisma |
| FDD-CONTRATO-09 | docs/FDD.md | Contrato | Assinatura da função publishWebhookEvent | TRANSCRICAO | [09:41] Bruno |
| FDD-CONTRATO-10 | docs/FDD.md | Contrato | Respostas paginadas com paginated() | CODIGO | src/shared/http/response.ts |
| FDD-ERRO-01 | docs/FDD.md | Erro | WEBHOOK_INVALID_URL para URL sem https | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-02 | docs/FDD.md | Erro | WEBHOOK_NOT_FOUND | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-03 | docs/FDD.md | Erro | WEBHOOK_SECRET_REQUIRED no worker | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-04 | docs/FDD.md | Erro | WEBHOOK_PAYLOAD_TOO_LARGE, sem envio e sem truncar | TRANSCRICAO | [09:23] Sofia |
| FDD-ERRO-05 | docs/FDD.md | Erro | WEBHOOK_DELIVERY_TIMEOUT após 10s | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-06 | docs/FDD.md | Erro | WEBHOOK_MAX_ATTEMPTS_EXCEEDED leva à DLQ | TRANSCRICAO | [09:15] Diego |
| FDD-ERRO-07 | docs/FDD.md | Restrição | validate converte qualquer ZodError em VALIDATION_ERROR, então a regra de https vai para o service | CODIGO | src/middlewares/validate.middleware.ts |
| FDD-ERRO-08 | docs/FDD.md | Restrição | NotFoundError fixa NOT_FOUND e BadRequestError aceita código | CODIGO | src/shared/errors/http-errors.ts |
| FDD-RES-01 | docs/FDD.md | Resiliência | Circuit breaker fora, comportamento em rajadas será observado | TRANSCRICAO | [09:39] Larissa |
| FDD-RES-02 | docs/FDD.md | Fallback | Sem canal alternativo, email adiado | TRANSCRICAO | [09:37] Larissa |
| FDD-RES-03 | docs/FDD.md | Fallback | Cliente pode consultar GET /orders/:id | TRANSCRICAO | [09:43] Diego |
| FDD-OBS-01 | docs/FDD.md | Observabilidade | Sem biblioteca nova, logs Pino como base de métricas | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | docs/FDD.md | Observabilidade | redactPaths ganha secret e previousSecret | CODIGO | src/shared/logger/index.ts |
| FDD-OBS-03 | docs/FDD.md | Observabilidade | requestId do request logger usado para correlação | CODIGO | src/middlewares/request-logger.middleware.ts |
| FDD-OBS-04 | docs/FDD.md | Observabilidade | Sinal de eventos por webhook por minuto para decidir rate limiting | TRANSCRICAO | [09:38] Diego |
| FDD-DEP-01 | docs/FDD.md | Dependência | Node 20, fetch nativo e node:crypto | CODIGO | package.json |
| FDD-DEP-02 | docs/FDD.md | Dependência | MySQL 8.0 | CODIGO | docker-compose.yml |
| FDD-DEP-03 | docs/FDD.md | Compatibilidade | tests/setup.ts precisa limpar as tabelas novas | CODIGO | tests/setup.ts |
| FDD-DEP-04 | docs/FDD.md | Compatibilidade | Build já inclui src/**/*.ts | CODIGO | tsconfig.build.json |
| FDD-RISK-01 | docs/FDD.md | Risco | Evento ultrapassado por outro do mesmo pedido durante backoff | TRANSCRICAO | [09:13] Larissa |
| FDD-RISK-02 | docs/FDD.md | Risco | Worker parado sem ninguém perceber | TRANSCRICAO | [09:11] Diego |
| FDD-RISK-03 | docs/FDD.md | Risco | Secret vazada | TRANSCRICAO | [09:22] Diego |
| FDD-RISK-04 | docs/FDD.md | Risco | Secrets guardadas em texto, entra na revisão de segurança | TRANSCRICAO | [09:46] Sofia |
| FDD-RISK-05 | docs/FDD.md | Risco | Rajada de eventos para um cliente | TRANSCRICAO | [09:38] Diego |
| FDD-INT-01 | docs/FDD.md | Integração | changeStatus chama publishWebhookEvent após orderStatusHistory.create | CODIGO | src/modules/orders/order.service.ts |
| FDD-INT-02 | docs/FDD.md | Integração | Rotas montadas em buildApiRouter | CODIGO | src/routes/index.ts |
| FDD-INT-03 | docs/FDD.md | Integração | Dependências montadas em buildControllers | CODIGO | src/app.ts |
| FDD-INT-04 | docs/FDD.md | Integração | requireRole('ADMIN') como em user.routes.ts | CODIGO | src/modules/users/user.routes.ts |
| FDD-INT-05 | docs/FDD.md | Integração | Nova factory createTestWebhook | CODIGO | tests/helpers/factories.ts |
| FDD-INT-06 | docs/FDD.md | Integração | Scripts worker e worker:start | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-07 | docs/FDD.md | Integração | Arquivos do módulo, incluindo webhook.worker.ts | TRANSCRICAO | [09:28] Bruno |
