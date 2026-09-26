### PRD: OMS (Order Management System) Webhooks de Notificação de Pedidos

Versão: 1.0
Data: 26/09/2026
Responsável: Diego Ferreira

Documentos relacionados: [RFC](RFC.md), [FDD](FDD.md), [ADRs](adrs/README.md), [Tracker](TRACKER.md)

---

### Resumo

Clientes B2B do OMS precisam saber quando o status dos seus pedidos muda sem ter que ficar consultando a nossa API. Esta feature cria webhooks de saída: o cliente cadastra uma URL, escolhe quais status quer acompanhar e passa a receber uma chamada HTTP assinada em menos de 10 segundos a cada mudança. A entrega tem retentativas automáticas, histórico consultável e reprocessamento manual para os casos que falharem de vez. O pedido formal veio de três clientes, e um deles já sinalizou que pode migrar para um concorrente se isso não sair até o fim do trimestre.

---

### Contexto e problema

Público-alvo
- Clientes B2B que integram os sistemas deles com o OMS pela API, começando por Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos).
- Usuários da nossa API que representam esses clientes e vão cadastrar e manter os webhooks ([09:32] Marcos).
- Administradores da plataforma, que reprocessam entregas que falharam de vez ([09:36] Sofia).

Cenários de uso chave
- O cliente quer saber na hora que um pedido foi enviado ou entregue, para atualizar o sistema dele, sem polling ([09:00] Marcos, [09:33] Marcos).
- O cliente cadastra mais de um endpoint e precisa saber qual cadastro recebeu cada chamada ([09:44] Sofia).
- O sistema do cliente fica fora do ar por algumas horas, por exemplo em manutenção planejada, e ele não pode perder as notificações desse período ([09:16] Diego).
- O cliente suspeita que a secret vazou e precisa trocá-la sem derrubar a integração ([09:21] Sofia, [09:22] Diego).
- O cliente quer conferir o que foi enviado para ele, com sucesso ou falha, resposta e tempo de resposta ([09:34] Marcos).

Onde essa feature será implantada
- No OMS existente (sistema já em produção, Node.js com TypeScript, Express e MySQL via Prisma), como um novo módulo da API e um processo worker separado que usa o mesmo banco ([09:11] Diego, [09:27] Bruno).

Problemas priorizados
- Os clientes fazem polling periódico no `GET /orders` para descobrir mudanças de status. Isso deixa a integração deles lenta e cara. Impacto: experiência ruim e custo para o cliente. Prioridade alta ([09:00] Marcos).
- Risco comercial: a Atlas Comercial sinalizou que pode migrar para um concorrente se a feature não for entregue até o fim do trimestre. Impacto: perda de cliente B2B. Prioridade alta ([09:00] Marcos).
- A plataforma não tem nenhum mecanismo de notificação externa. Toda integração depende de o cliente perguntar. Impacto: não existe base para comunicação em tempo real. Prioridade média.

---

### Objetivos e métricas

| Objetivo | Métrica | Meta |
| --- | --- | --- |
| Avisar o cliente em "tempo real" quando o status do pedido muda | Tempo entre a mudança de status e a entrega da notificação, com o endpoint do cliente saudável | p95 abaixo de 10 segundos ([09:02] Marcos) |
| Atender os clientes que fizeram o pedido formal | Clientes com webhook ativo recebendo eventos em produção | 3 de 3 (Atlas Comercial, MaxDistribuição e Nova Cargo) ([09:00] Marcos) |
| Não perder notificação quando o status muda | Percentual de mudanças de status, com webhook interessado, que geram evento registrado | 100% ([09:40] Bruno) |
| Suportar indisponibilidade temporária do cliente sem intervenção manual | Janela coberta pelas retentativas automáticas antes de o evento ir para a fila de falhas | Cerca de 15 horas, com 5 retentativas ([09:17] Diego, [09:17] Marcos) |
| Reduzir a dependência de polling dos clientes | Chamadas ao `GET /orders` feitas pelos três clientes | Queda após a adoção. A linha de base ainda precisa ser medida (hipótese) |

---

### Escopo

Incluso
- Cadastro, listagem, edição e remoção de webhooks pela API, com filtro dos status que o webhook quer receber ([09:31] Marcos, [09:33] Bruno).
- Notificação de toda mudança de status de pedido feita pela plataforma, para os webhooks ativos do cliente que assinaram aquele status ([09:40] Bruno).
- Assinatura das notificações com secret exclusiva por endpoint, gerada pela plataforma, e rotação da secret com 24 horas de convivência ([09:21] Sofia).
- Retentativas automáticas e fila de falhas definitivas (DLQ) ([09:15] Diego).
- Reprocessamento manual da DLQ, restrito a administradores e auditado ([09:35] Diego, [09:36] Sofia).
- Histórico de entregas por webhook ([09:34] Marcos).
- Documentação de integração no portal do desenvolvedor, a cargo do PM ([09:26] Marcos, [09:40] Marcos).

Fora de escopo
- Aviso por email ao cliente quando o webhook dele falha repetidamente. Adiado para a próxima fase, depois de medir o impacto ([09:37] Marcos, [09:37] Larissa).
- Dashboard visual para o cliente acompanhar os webhooks. É um projeto separado do time de frontend. Nesta fase só existem endpoints ([09:39] Marcos, [09:40] Larissa).
- Rate limiting no envio para o cliente. Fica como ponto em aberto: vamos observar e decidir depois ([09:38] Diego, [09:39] Larissa).
- Webhooks de entrada, do cliente para a plataforma. O cliente só recebe ([09:02] Marcos).
- Garantia de ordem global entre eventos e processamento com vários workers em paralelo ([09:13] Diego, [09:13] Larissa, [09:14] Marcos).
- Arquivamento de eventos antigos já entregues, depois de uns 30 dias ([09:08] Diego).
- Itens do pedido dentro da notificação. O cliente consulta os detalhes na API de pedidos ([09:43] Diego).
- Permissões mais restritas para gerenciar webhooks além de estar autenticado ([09:37] Sofia).

---

### Requisitos funcionais

#### FR-001 Cadastrar webhook
O cliente cadastra um endpoint para receber notificações de pedidos, informando a URL e os status que quer acompanhar.

**Fluxo principal**
- O usuário autenticado envia o cadastro com o cliente (`customer_id`), a URL e a lista de status desejados ([09:31] Marcos).
- A plataforma valida os dados e confirma que o cliente existe.
- A plataforma gera uma secret exclusiva para esse endpoint ([09:21] Sofia).
- O webhook é criado como ativo e a secret é devolvida na resposta ([09:31] Marcos).

**Fluxos alternativos e exceções**
- O `customer_id` é informado na requisição. Ele não vem do token, porque o token é do usuário que opera a API em nome do cliente ([09:32] Bruno, [09:32] Larissa).
- Um mesmo cliente pode ter mais de um webhook, cada um com a sua secret ([09:44] Sofia).

**Erros previstos**
- URL que não usa https é recusada com erro de validação ([09:23] Sofia).
- Cliente inexistente é recusado.
- Requisição sem autenticação é recusada.

**Prioridade:** alta

---

#### FR-002 Listar webhooks de um cliente
O usuário consulta os webhooks cadastrados para um cliente.

**Fluxo principal**
- O usuário autenticado pede a lista informando o cliente ([09:33] Bruno).
- A plataforma devolve os webhooks com URL, status assinados e estado ativo. A secret nunca aparece na listagem. Ela só é exibida na criação e na rotação ([09:31] Marcos).

**Fluxos alternativos e exceções**
- Cliente sem webhooks recebe uma lista vazia.

**Erros previstos**
- Pedido de listagem sem informar o cliente é recusado com erro de validação.

**Prioridade:** alta

---

#### FR-003 Editar webhook
O usuário altera a URL, a lista de status ou o estado ativo de um webhook.

**Fluxo principal**
- O usuário autenticado envia as alterações ([09:33] Bruno).
- A plataforma valida e grava.

**Fluxos alternativos e exceções**
- Desativar o webhook pausa as entregas sem apagar o cadastro nem o histórico ([09:21] Bruno).

**Erros previstos**
- Webhook inexistente.
- Nova URL sem https ([09:23] Sofia).

**Prioridade:** alta

---

#### FR-004 Remover webhook
O usuário remove um webhook que não quer mais usar.

**Fluxo principal**
- O usuário autenticado pede a remoção ([09:33] Bruno).
- A plataforma remove o cadastro e para de enviar eventos para ele.

**Fluxos alternativos e exceções**
- Eventos ainda pendentes para esse webhook deixam de ser enviados.

**Erros previstos**
- Webhook inexistente.

**Prioridade:** média

---

#### FR-005 Filtrar eventos por status
Cada webhook recebe apenas as mudanças para os status que ele assinou.

**Fluxo principal**
- No cadastro ou na edição, o cliente informa os status de interesse, por exemplo só `SHIPPED` e `DELIVERED` ([09:33] Marcos).
- Quando o status de um pedido muda, só os webhooks que assinaram o novo status geram notificação.
- Se nenhum webhook do cliente assinou aquele status, nada é registrado ([09:34] Bruno).

**Fluxos alternativos e exceções**
- O filtro vale a partir do momento em que é alterado. Eventos já registrados não mudam.

**Erros previstos**
- Lista vazia ou com status inexistente é recusada.

**Prioridade:** alta

---

#### FR-006 Notificar mudança de status do pedido
Toda mudança de status de um pedido gera uma notificação HTTP para os webhooks interessados do cliente dono do pedido.

**Fluxo principal**
- Um operador muda o status do pedido na plataforma.
- A notificação é registrada junto com a mudança de status, de forma que uma não existe sem a outra ([09:40] Bruno).
- Em até 10 segundos a plataforma envia ao cliente um JSON com identificador do evento, tipo, data e hora, pedido, número do pedido, status anterior, status novo, cliente e valor total ([09:43] Diego).
- A notificação reflete o pedido no momento da mudança, mesmo que seja enviada mais tarde ([09:52] Larissa).

**Fluxos alternativos e exceções**
- Se o registro da notificação falhar, a mudança de status também não acontece ([09:40] Bruno).
- A notificação não leva os itens do pedido. Quem precisar deles consulta a API de pedidos ([09:43] Diego).

**Erros previstos**
- Notificação com mais de 64KB não é enviada e é tratada como falha ([09:23] Sofia, [09:24] Larissa).

**Prioridade:** alta

---

#### FR-007 Identificar cada evento para deduplicação
Cada notificação leva um identificador único e estável, para o cliente descartar repetições.

**Fluxo principal**
- A plataforma gera um identificador único quando o evento é registrado ([09:25] Diego).
- O identificador vai em todo envio, no header `X-Event-Id` e no corpo.
- O cliente usa esse identificador para ignorar eventos que já processou ([09:25] Diego).

**Fluxos alternativos e exceções**
- O mesmo evento pode chegar mais de uma vez (entrega at-least-once), sempre com o mesmo identificador ([09:24] Diego).

**Erros previstos**
- Nenhum erro para o cliente. A regra é documentada no portal do desenvolvedor ([09:26] Marcos).

**Prioridade:** alta

---

#### FR-008 Assinar notificações
Toda notificação é assinada para que o cliente confirme que ela veio da plataforma e não foi alterada.

**Fluxo principal**
- A plataforma assina o corpo da notificação com HMAC-SHA256, usando a secret do webhook ([09:20] Sofia).
- A assinatura vai no header `X-Signature`, junto com `X-Webhook-Id` e `X-Timestamp` ([09:44] Diego, [09:44] Sofia).
- O cliente recalcula a assinatura do lado dele e compara.

**Fluxos alternativos e exceções**
- Durante uma rotação de secret, as notificações seguem válidas para quem ainda usa a secret anterior, por 24 horas ([09:21] Sofia).

**Erros previstos**
- Nenhuma notificação sai sem assinatura.

**Prioridade:** alta

---

#### FR-009 Rotacionar secret
O cliente pede uma secret nova pela API e tem 24 horas para migrar os sistemas dele.

**Fluxo principal**
- O usuário autenticado pede a rotação da secret de um webhook ([09:21] Sofia).
- A plataforma gera uma secret nova e a devolve.
- A secret anterior continua valendo em paralelo por 24 horas e depois deixa de valer ([09:21] Sofia).

**Fluxos alternativos e exceções**
- Uma nova rotação dentro das 24 horas descarta a secret mais antiga.

**Erros previstos**
- Webhook inexistente.

**Prioridade:** alta

---

#### FR-010 Retentar entregas que falharam
Se o endpoint do cliente não responder com sucesso, a plataforma tenta de novo automaticamente.

**Fluxo principal**
- Uma entrega que falha ou não responde em 10 segundos é marcada para nova tentativa ([09:42] Diego).
- As retentativas acontecem depois de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego).
- Se alguma retentativa tiver sucesso, o evento é dado como entregue.

**Fluxos alternativos e exceções**
- Depois da quinta retentativa sem sucesso, o evento vai para a DLQ (FR-011) ([09:15] Diego).

**Erros previstos**
- Timeout, erro de rede e resposta de erro do cliente são tratados da mesma forma: nova tentativa.

**Prioridade:** alta

---

#### FR-011 Guardar falhas definitivas e permitir reprocessamento manual
Eventos que esgotaram as tentativas ficam guardados e podem ser reenviados por um administrador.

**Fluxo principal**
- O evento que falhou de vez é guardado com o conteúdo, o motivo da falha e a data ([09:18] Diego).
- Um administrador pede o reenvio pela API ([09:18] Diego).
- O evento volta para a fila de envio com o mesmo identificador.
- A plataforma registra quem fez o reenvio ([09:36] Sofia).

**Fluxos alternativos e exceções**
- Se o reenvio falhar de novo, o evento passa pelo ciclo de retentativas outra vez.

**Erros previstos**
- Usuário sem papel de administrador é bloqueado ([09:36] Sofia).
- Item inexistente ou já reprocessado.

**Prioridade:** média

---

#### FR-012 Consultar histórico de entregas
O cliente consulta as notificações enviadas para um webhook, com resultado e detalhes.

**Fluxo principal**
- O usuário autenticado pede o histórico de um webhook ([09:34] Marcos).
- A plataforma devolve os envios mais recentes, até 100 por página, com sucesso ou falha, conteúdo enviado, resposta do cliente e tempo de resposta ([09:34] Marcos).

**Fluxos alternativos e exceções**
- Eventos ainda em retentativa aparecem com o resultado da última tentativa.

**Erros previstos**
- Webhook inexistente.

**Prioridade:** média

---

### Requisitos não funcionais

Performance
- Notificação entregue em menos de 10 segundos após a mudança de status, com o cliente saudável ([09:02] Marcos).
- A fila de envio é verificada a cada 2 segundos. Esse é o atraso mínimo aceito ([09:10] Larissa).
- Cada chamada ao cliente tem timeout de 10 segundos ([09:42] Diego).
- A mudança de status do pedido não pode ficar mais lenta por causa de clientes lentos. Nenhuma chamada HTTP acontece dentro da transação de pedidos ([09:04] Bruno).

Disponibilidade
- O envio roda em processo separado da API, para que reinícios da API não interrompam as entregas ([09:11] Diego).
- Se o processo de envio parar, os eventos ficam guardados e são enviados quando ele voltar. Nada se perde.
- A reunião não definiu meta numérica de uptime. Fica como hipótese seguir o mesmo patamar da API atual.

Segurança e autorização
- Assinatura HMAC-SHA256 com secret exclusiva por endpoint ([09:20] Sofia, [09:21] Sofia).
- URL de destino obrigatoriamente https ([09:23] Sofia).
- Gerenciamento de webhooks exige usuário autenticado, com qualquer papel nesta fase ([09:37] Sofia).
- Reprocessamento da DLQ exige papel de administrador ([09:36] Sofia).
- A secret só aparece na criação e na rotação, nunca em listagens ou logs.
- Revisão de segurança antes do deploy, com pelo menos dois dias úteis ([09:46] Sofia).

Observabilidade
- Logs estruturados com o logger que o projeto já usa, com o identificador do evento em todas as etapas ([09:29] Bruno).
- Acompanhamento de latência de entrega, fila pendente, taxa de sucesso e itens na DLQ. O detalhe está no FDD.

Confiabilidade e integridade de dados
- Registro da notificação e mudança de status acontecem na mesma transação: ou as duas acontecem, ou nenhuma ([09:06] Diego, [09:40] Bruno).
- Entrega at-least-once, com identificador estável para deduplicação ([09:24] Diego).
- Ordem das notificações garantida apenas por pedido, e enquanto houver um único processo de envio ([09:13] Larissa).
- Limite de 64KB por notificação ([09:24] Larissa).

Compatibilidade e portabilidade
- Nenhuma mudança nas rotas atuais da API de pedidos.
- Nenhuma infraestrutura nova. Usa o mesmo MySQL e a mesma stack ([09:07] Diego, [09:11] Diego).
- Notificação em JSON, com o tipo de evento `order.status_changed` ([09:43] Diego).

Compliance
- Todo reprocessamento manual registra quem fez e quando ([09:36] Sofia).
- Falhas definitivas ficam guardadas com o conteúdo e o motivo, como evidência ([09:18] Diego).

Acessibilidade no frontend consumidor
- Não se aplica nesta fase. A feature é só API, e o painel visual está fora de escopo ([09:40] Larissa).

---

### Arquitetura e abordagem

Abordagem
- Novo módulo dentro do monólito do OMS, com os mesmos padrões dos módulos existentes. A comunicação com o cliente é assíncrona, por meio de uma tabela de saída (outbox) no MySQL que é lida por um worker separado ([09:06] Diego, [09:27] Bruno). A proposta completa está no [RFC](RFC.md).

Componentes
- API REST do OMS, com o novo módulo de webhooks para cadastro, rotação, histórico e reprocessamento ([09:27] Bruno).
- Tabelas novas no MySQL existente: configuração de webhooks, outbox de eventos e DLQ ([09:06] Diego, [09:18] Diego, [09:21] Bruno).
- Worker de envio, em processo separado, iniciado por `npm run worker` ([09:11] Larissa).

Integrações
- Serviço de pedidos: a mudança de status registra o evento dentro da própria transação ([09:40] Bruno).
- Endpoints HTTPS dos clientes, que recebem as notificações ([09:23] Sofia).
- Portal do desenvolvedor, com a documentação de integração ([09:40] Marcos).

### Decisões e trade-offs

#### Decisão: Outbox no MySQL em vez de envio síncrono ou fila dedicada
- **Justificativa:** garante que status e notificação andem juntos e não exige infraestrutura nova para um time pequeno ([09:07] Diego). Detalhe no [ADR-001](adrs/ADR-001-outbox-no-mysql.md).
- **Trade-off:** mais carga e uma tabela crescente no banco de produção.

#### Decisão: Worker separado com verificação a cada 2 segundos
- **Justificativa:** o MySQL não avisa processos externos, e 2 segundos cabem folgados na meta de 10 ([09:09] Diego). Detalhe no [ADR-002](adrs/ADR-002-worker-separado-com-polling.md).
- **Trade-off:** atraso mínimo de 2 segundos e um processo a mais para operar.

#### Decisão: 5 retentativas em cerca de 15 horas, depois DLQ com reenvio manual
- **Justificativa:** cobre manutenções de algumas horas, que já aconteceram com clientes, sem deixar eventos pendurados para sempre ([09:16] Diego). Detalhe no [ADR-003](adrs/ADR-003-retry-com-backoff-e-dlq.md).
- **Trade-off:** falhas longas dependem de um administrador para reenviar.

#### Decisão: HMAC-SHA256 com secret por endpoint e rotação com 24 horas
- **Justificativa:** padrão de mercado, e um vazamento fica restrito a um endpoint ([09:21] Sofia). Detalhe no [ADR-004](adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md).
- **Trade-off:** mais complexidade para guardar e rotacionar secrets.

#### Decisão: Entrega at-least-once com identificador de evento
- **Justificativa:** exactly-once exigiria coordenação dos dois lados. Stripe e GitHub usam at-least-once ([09:25] Diego). Detalhe no [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md).
- **Trade-off:** o cliente precisa tratar duplicidade ([09:25] Sofia).

#### Decisão: Reuso dos padrões do projeto e notificação como retrato do momento
- **Justificativa:** manutenção mais simples e eventos fiéis ao momento da mudança ([09:30] Larissa, [09:52] Larissa). Detalhes no [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) e no [ADR-007](adrs/ADR-007-payload-enxuto-como-snapshot-na-insercao.md).
- **Trade-off:** sem ferramentas especializadas de observabilidade, e cliente que quiser os itens faz uma chamada extra.

---

### Dependências

#### Organizacional: Revisão de segurança
A Sofia precisa revisar o código de HMAC e de geração de secret antes do deploy, com pelo menos dois dias úteis reservados no fim do cronograma ([09:46] Sofia, [09:49] Sofia).

#### Organizacional: Documentação no portal do desenvolvedor
O Marcos precisa publicar no portal como integrar via API, validar a assinatura e deduplicar pelo `X-Event-Id`. Sem isso os clientes não conseguem consumir a feature com segurança ([09:26] Marcos, [09:40] Marcos).

#### Organizacional: Alinhamento de prazo com os clientes
O Marcos confirma com a Atlas o prazo de fim de novembro e atualiza os três clientes ([09:47] Marcos, [09:49] Marcos).

#### Técnica: Execução do worker em produção
O ambiente de produção precisa rodar um segundo processo Node (o worker) ao lado da API, com acesso ao mesmo banco ([09:11] Diego, [09:11] Bruno).

#### Externa: Preparo do lado dos clientes
Cada cliente precisa expor um endpoint https, validar a assinatura HMAC e tratar eventos repetidos pelo identificador ([09:23] Sofia, [09:25] Diego).

---

### Riscos e mitigação

#### Atraso na entrega faz a Atlas Comercial migrar para um concorrente
- **Probabilidade:** media
- **Impacto:** perda de um cliente B2B que pediu a feature formalmente ([09:00] Marcos).
- **Mitigação:**
  - Escopo enxuto, com email, dashboard e rate limiting fora desta fase ([09:48] Larissa).
  - Estimativa de três sprints, já incluindo a revisão de segurança ([09:46] Larissa).
  - Prazo confirmado diretamente com a Atlas ([09:47] Marcos).
- **Plano de contingência:** entregar primeiro os requisitos de prioridade alta e deixar histórico de entregas e reprocessamento manual para logo em seguida.

#### Clientes processam a mesma notificação duas vezes
- **Probabilidade:** media
- **Impacto:** ação duplicada no sistema do cliente, por exemplo disparar duas vezes o processo de expedição.
- **Mitigação:**
  - Identificador de evento estável em todas as tentativas ([09:25] Diego).
  - Documentação destacada no portal do desenvolvedor ([09:26] Marcos).
- **Plano de contingência:** apoio direto do time ao cliente afetado, usando o histórico de entregas para identificar os eventos repetidos.

#### Vazamento de secret de um cliente
- **Probabilidade:** media
- **Impacto:** terceiros conseguem forjar notificações para aquele cliente. Já aconteceu de um cliente vazar secret em log ([09:22] Diego).
- **Mitigação:**
  - Secret exclusiva por endpoint ([09:21] Sofia).
  - Rotação pela API com 24 horas de convivência ([09:21] Sofia).
  - Revisão de segurança antes do deploy ([09:46] Sofia).
- **Plano de contingência:** rotação imediata da secret. Em último caso, desativar o webhook.

#### Cliente fora do ar por mais de 15 horas
- **Probabilidade:** baixa
- **Impacto:** eventos vão para a DLQ e o cliente só os recebe depois de um reenvio manual ([09:17] Marcos).
- **Mitigação:**
  - Janela de retentativas longa, dimensionada a partir de casos reais de manutenção ([09:16] Diego).
  - Alerta para itens novos na DLQ.
- **Plano de contingência:** administrador reprocessa os eventos pela API. O cliente também pode consultar o estado atual dos pedidos pela API existente.

#### Rajada de notificações para um mesmo cliente
- **Probabilidade:** media
- **Impacto:** o endpoint do cliente pode ficar sobrecarregado quando muitos pedidos mudam de status ao mesmo tempo ([09:38] Diego).
- **Mitigação:**
  - Acompanhar o volume de eventos por webhook ([09:39] Diego).
- **Plano de contingência:** priorizar a decisão de rate limiting que ficou em aberto.

#### Notificações chegam fora de ordem para um mesmo pedido
- **Probabilidade:** baixa
- **Impacto:** o cliente pode receber um status mais novo antes de um mais antigo quando a primeira entrega falha.
- **Mitigação:**
  - Um único processo de envio, em ordem de registro ([09:12] Diego).
  - Status anterior, status novo e horário em toda notificação ([09:43] Diego).
- **Plano de contingência:** o cliente consulta o pedido na API para confirmar o estado atual.

---

### Critérios de aceitação
Checklist objetivo que define se a feature está pronta.

- Um usuário autenticado consegue cadastrar, listar, editar e remover webhooks de um cliente pela API.
- O cadastro de URL com http é recusado com erro de validação.
- A secret aparece apenas nas respostas de criação e de rotação.
- Uma mudança de status assinada por um webhook ativo gera exatamente uma notificação para esse webhook.
- Uma mudança de status que nenhum webhook do cliente assinou não gera notificação.
- Se o registro da notificação falhar, o status do pedido não muda.
- Com o endpoint do cliente saudável, a notificação chega em menos de 10 segundos no p95.
- Toda notificação tem `X-Event-Id`, `X-Signature`, `X-Timestamp`, `X-Webhook-Id` e `Content-Type: application/json`, e a assinatura confere com HMAC-SHA256 do corpo.
- Um endpoint que sempre falha recebe o envio inicial e mais 5 retentativas nos intervalos de 1m, 5m, 30m, 2h e 12h, e o evento termina na DLQ com o motivo registrado.
- Uma chamada sem resposta em 10 segundos conta como falha.
- Uma notificação acima de 64KB não é enviada.
- Depois de uma rotação, a secret anterior continua funcionando por 24 horas e depois para.
- O reprocessamento da DLQ é bloqueado para quem não é administrador, e quando feito por administrador registra quem fez.
- O histórico de entregas mostra, para cada envio, resultado, conteúdo, resposta e tempo de resposta.
- As rotas atuais de pedidos continuam funcionando sem mudança de contrato.

---

### Testes e validação

Tipos de teste obrigatórios
- Testes unitários para assinatura HMAC, cálculo do próximo horário de retentativa e filtro por status.
- Testes de integração com banco real, no mesmo padrão da suíte atual em Vitest e Supertest, para a mudança de status gerando o evento na mesma transação, inclusive o caso de rollback.
- Testes de integração do worker contra um servidor HTTP local, cobrindo sucesso, erro, timeout, retentativas e DLQ.
- Testes de autorização no reprocessamento (administrador e operador).
- Revisão de segurança manual da Sofia sobre assinatura e geração de secret ([09:46] Sofia).

Estratégia de validação
- TDD para as regras críticas (assinatura, backoff e atomicidade com a mudança de status). Depois, validação ponta a ponta em homologação com um endpoint de teste de um dos três clientes antes de liberar em produção (hipótese: o cliente precisa topar participar).
