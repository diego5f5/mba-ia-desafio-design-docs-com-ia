# ADR-006: Reuso dos padrões existentes do projeto no módulo de webhooks

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Larissa (Tech Lead), Bruno (Eng. Pleno, Pedidos), Diego (Eng. Sênior, Plataforma), Sofia (Segurança)
**Relacionados:** [ADR-002](ADR-002-worker-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-e-dlq.md)

## Contexto

O OMS já tem padrões bem definidos, e o time é pequeno ([09:07] Diego). Olhando o código:

- Cada domínio é um módulo em `src/modules/<dominio>` com `*.controller.ts`, `*.service.ts`, `*.repository.ts`, `*.routes.ts` e `*.schemas.ts` (por exemplo `src/modules/orders/` e `src/modules/customers/`).
- Os erros de negócio herdam de `AppError` (`src/shared/errors/app-error.ts`), com `statusCode` e um `errorCode` em caixa alta. Exemplos em `src/shared/errors/http-errors.ts`: `InsufficientStockError` com `INSUFFICIENT_STOCK` e `InvalidStatusTransitionError` com `INVALID_STATUS_TRANSITION`.
- O `errorMiddleware` em `src/middlewares/error.middleware.ts` já transforma `AppError`, `ZodError` e erros conhecidos do Prisma em respostas JSON no formato `{ error: { code, message, details } }`.
- A validação de entrada usa Zod pelo `validate` de `src/middlewares/validate.middleware.ts`.
- O log é Pino, em `src/shared/logger/index.ts`, com `redact` para campos sensíveis.
- Autenticação e autorização por papel ficam em `src/middlewares/auth.middleware.ts` (`authenticate` e `requireRole`).
- As rotas são montadas em `src/routes/index.ts` e as dependências são montadas à mão em `buildControllers`, no `src/app.ts`.

## Decisão

Reuso máximo do que já existe. O webhook é um módulo como os outros ([09:30] Larissa):

1. Criar o diretório novo `src/modules/webhooks/` com controller, service, repository, routes e schemas ([09:27] Bruno). A lógica do worker fica no mesmo módulo, em `webhook.worker.ts` ([09:28] Bruno), e a função de publicação de evento também.
2. Erros novos herdam de `AppError` e usam o **prefixo `WEBHOOK_`** nos códigos, por exemplo `WEBHOOK_NOT_FOUND`, `WEBHOOK_INVALID_URL` e `WEBHOOK_SECRET_REQUIRED` ([09:28] Bruno, [09:29] Larissa).
3. O `errorMiddleware` não muda. Ele já trata os erros do módulo novo ([09:29] Bruno).
4. O log continua sendo Pino. Nenhuma biblioteca nova de log ([09:29] Bruno).
5. Schemas de entrada em Zod, no mesmo estilo de `order.schemas.ts` ([09:30] Larissa).
6. O endpoint de replay usa o `requireRole('ADMIN')` que já existe, do mesmo jeito que `src/modules/users/user.routes.ts` faz hoje ([09:36] Larissa).
7. Ids em UUID, igual ao resto do projeto ([09:51] Larissa).
8. A integração com pedidos acontece por uma função `publishWebhookEvent(tx, order, fromStatus, toStatus)` que recebe o client da transação atual. Assim o `OrderService` não precisa receber um repository de webhooks inteiro ([09:41] Bruno, [09:41] Diego).

## Alternativas Consideradas

**Estrutura própria para o módulo de webhooks, com uma camada de eventos genérica e bibliotecas novas (logger, cliente HTTP, validação).** É a alternativa plausível para uma feature de "infraestrutura". Descartada porque aumenta a superfície de manutenção para um time pequeno e contraria a decisão explícita de não trazer nada novo ([09:29] Bruno, [09:30] Larissa).

**Injetar um `WebhookRepository` no construtor do `OrderService`.** Levantado pelo Bruno como uma das opções ([09:41] Bruno). Preterido em favor da função que recebe o `tx`, que é mais simples e deixa explícito que a escrita acontece dentro da transação de pedidos ([09:41] Diego).

## Consequências

**Positivas**
- Quem conhece o módulo de pedidos já sabe navegar no módulo de webhooks.
- Respostas de erro com o mesmo formato do resto da API, sem mexer no middleware.
- Zero dependências novas em `package.json`.

**Negativas**
- O `validate` transforma qualquer falha de Zod em `VALIDATION_ERROR`. Se quisermos devolver um código `WEBHOOK_*` específico para uma regra de entrada, essa regra precisa ser checada no service, não no schema. O FDD detalha onde isso acontece.
- O `NotFoundError` existente sempre usa o código `NOT_FOUND`. Para seguir o prefixo `WEBHOOK_`, o módulo precisa de classes próprias, como `WebhookNotFoundError`.
- O padrão atual não traz métricas nem tracing distribuído. A observabilidade do módulo fica limitada ao que conseguimos com logs estruturados e consultas no banco.

**Trade-off:** abrimos mão de ferramentas mais especializadas (fila dedicada, observabilidade mais rica) em troca de consistência com a base de código e menor custo de manutenção.
