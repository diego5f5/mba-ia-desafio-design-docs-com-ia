# Da Reunião ao Documento: Design Docs do Sistema de Webhooks

Entrega do desafio "Da Reunião ao Documento: Design Docs Gerados por IA" do MBA. O enunciado original está no [repositório base](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia).

## Sobre o desafio

O ponto de partida era uma empresa com um OMS (Order Management System) em produção que decidiu, numa reunião de uns 55 minutos, construir um sistema de webhooks para avisar clientes B2B quando o status de um pedido muda. A reunião teve decisões, discussões, ideias descartadas e coisas empurradas para depois, mas o único registro era a transcrição da call (`TRANSCRICAO.md`).

Minha tarefa foi transformar essa transcrição, junto com o código da aplicação, num pacote de design docs que um time conseguisse pegar e começar a implementar: PRD, RFC, FDD, sete ADRs e um tracker que liga cada item à sua origem. A IA foi a ferramenta principal de produção, e o meu papel foi dirigir o trabalho, revisar o que ela entregava e corrigir o que estava errado ou inventado. Não mexi em nada do código (`src/`, `prisma/`, `tests/`). Ele serviu só como referência.

## Ferramentas de IA utilizadas

- **Claude Code (modelo Claude Opus 5.5)**, rodando no terminal dentro do repositório. Foi a ferramenta principal: leu o código e a transcrição direto dos arquivos, gerou os documentos e rodou as verificações. Trabalhar com o repositório aberto fez muita diferença, porque a IA conseguia conferir nomes de arquivo, métodos e classes em vez de chutar.
- **Prompts de PRD e FDD disponibilizados pelo professor**, usados como base de formato e de checklist. Os dois são prompts de entrevista, então adaptei para a IA responder a entrevista sozinha usando a transcrição e o código como "entrevistado" (explico abaixo).
- **Um script Python curto, gerado pela própria IA**, para validar o tracker e as citações. Não é uma ferramenta de IA em si, mas foi a forma que encontrei de usar a IA para checar o trabalho dela mesma.

## Workflow adotado

Segui a ordem sugerida no enunciado, com uma etapa de exploração antes:

1. **Exploração.** Pedi para a IA ler a transcrição inteira e os arquivos centrais do código (`prisma/schema.prisma`, `order.service.ts`, `order.status.ts`, erros, middlewares, logger, rotas, `server.ts`, `app.ts`, testes). Antes de escrever qualquer documento, pedi uma lista separando decisões fechadas, requisitos, itens descartados, itens adiados e detalhes secundários, sempre com timestamp. Essa lista virou a base de tudo.
2. **Templates.** A IA perguntou se eu tinha os templates do curso. Passei os prompts de PRD e FDD. Para RFC e ADR, que eu não tinha, combinei de usar MADR nos ADRs (é o que o enunciado cita) e um formato clássico de RFC.
3. **ADRs.** Sete ADRs, um por decisão: as seis principais da reunião e uma sétima sobre o payload como snapshot, que era uma decisão secundária com alternativas reais discutidas.
4. **RFC.** Consolidei a proposta em cima dos ADRs, com as alternativas descartadas e as questões em aberto, sem entrar em tabela, endpoint ou código.
5. **FDD.** O documento mais longo. Modelo de dados, fluxos, contratos, matriz de erros, observabilidade e a seção de integração com o código existente.
6. **PRD.** Por último entre os grandes. Com os outros prontos, virou uma consolidação no nível de produto.
7. **Tracker.** Montado varrendo os documentos prontos, item por item.
8. **Verificação.** Um script conferiu se cada `[hh:mm] Nome` citado existe mesmo na transcrição, se cada caminho de código existe e os percentuais do tracker.
9. **README.** Este arquivo, escrito no fim.

Para a IA não ficar me fazendo pergunta a cada passo, combinei que ela respondesse as entrevistas dos templates no meu lugar, mas só com o que estivesse na transcrição ou no código, e marcando como hipótese qualquer coisa que não tivesse origem.

## Prompts customizados

**Prompt de filtragem da transcrição**, usado antes de qualquer documento. A ideia era impedir que coisa descartada virasse requisito:

```text
Leia TRANSCRICAO.md inteira e os arquivos de src/ e prisma/ que tratam de pedidos,
erros, autenticação, logger e bootstrap da aplicação.

Monte uma lista com cinco grupos, e para cada item cite o timestamp e quem falou
no formato [hh:mm] Nome:
1. Decisões fechadas (alguém disse "decidido", "anotado" ou houve concordância explícita)
2. Requisitos funcionais pedidos
3. Itens explicitamente descartados, com o motivo
4. Itens adiados ou deixados em aberto
5. Detalhes técnicos secundários (formato, headers, timeouts, limites)

Regras:
- Não misture grupos. Se algo foi sugerido e depois descartado, vai no grupo 3.
- Se uma fala foi corrigida depois na reunião, fique com a versão final e aponte a correção.
- Aponte qualquer ambiguidade ou contradição que você encontrar, sem resolver sozinho.
- Para cada item que toca o código, diga o arquivo real que ele afeta.
```

**Adaptação dos prompts de entrevista do professor (PRD e FDD)**. Os prompts originais fazem uma pergunta por vez. Colei o prompt do professor inteiro e acrescentei este bloco no começo:

```text
Use o prompt abaixo como formato e checklist, mas NÃO me entreviste.
Faça a entrevista internamente, respondendo cada pergunta como se fosse eu,
usando apenas:
- a TRANSCRICAO.md (cite [hh:mm] Nome ao lado de cada informação que vier dela)
- o código do repositório (cite o caminho do arquivo)

Se uma pergunta da entrevista não tiver resposta na transcrição nem no código,
escreva "hipótese" e explique de onde veio a suposição. Não invente números.
Pule a etapa de exportar JSON.
Siga exatamente o esqueleto de saída do template, sem travessões.
Não repita o que já está nos ADRs e no RFC: faça link para eles.
```

**Prompt dos ADRs**:

```text
Escreva um ADR por decisão, no formato MADR em português, com as seções
Status, Contexto, Decisão, Alternativas Consideradas e Consequências
(positivas, negativas e uma linha de trade-off explícito).

As alternativas precisam ter sido discutidas na reunião. Se não houver nenhuma,
proponha uma plausível e diga claramente que ela não foi discutida.
No contexto, cite os arquivos e funções reais do código envolvidos
(ex: OrderService.changeStatus em src/modules/orders/order.service.ts).
Nome do arquivo: docs/adrs/ADR-NNN-titulo-em-kebab-case.md.
```

**Prompt de verificação do tracker**, usado no fim:

```text
Escreva e rode um script que:
1. Leia docs/TRACKER.md e, para cada linha com Fonte TRANSCRICAO, confira se o
   par [hh:mm] Nome existe como fala em TRANSCRICAO.md.
2. Para cada linha com Fonte CODIGO, confira se o caminho existe no repositório.
3. Calcule a porcentagem de linhas TRANSCRICAO e o total de linhas CODIGO.
4. Varra todos os .md de docs/ e procure citações [hh:mm] Nome inexistentes,
   caminhos de src/, tests/ ou prisma/ que não existem.
Me mostre só o que falhou.
```

## Iterações e ajustes

Foram **quatro ciclos principais**: exploração e filtragem; ADRs e RFC; FDD e PRD com revisão crítica; tracker com verificação automática e correções finais. Os ajustes que mais pesaram:

1. **Quantas tentativas são "5 tentativas"?** A reunião diz "5 tentativas" e lista cinco intervalos (1m, 5m, 30m, 2h, 12h). Se fossem 5 envios no total, só caberiam 4 intervalos e a janela seria de umas 2h36, não os "quase 15 horas" que o Diego falou. Pedi para a IA fazer a conta e registrar a interpretação (envio inicial mais 5 retentativas) no ADR-003, com a justificativa, em vez de escolher uma leitura em silêncio.

2. **Código de erro que nunca chegaria no cliente.** A primeira versão colocava a validação de https só no schema Zod, como a Sofia sugeriu, e usava o código `WEBHOOK_INVALID_URL`. Olhando o `validate.middleware.ts`, qualquer falha de Zod vira `VALIDATION_ERROR`, então aquele código nunca apareceria. A regra passou para o service, o schema continua validando o formato da URL, e a explicação ficou no FDD.

3. **Status HTTP errado na classe de erro.** Na seção de integração a IA disse que `WebhookInvalidUrlError` seguiria o modelo de `InsufficientStockError`, que estende `UnprocessableEntityError`. Isso daria 422, e o contrato dizia 400. Corrigi para estender `BadRequestError`, que já aceita código customizado.

4. **`customer_id` vindo do JWT.** O Marcos fala primeiro que o `customer_id` vem do JWT, e minutos depois o Bruno e a Larissa corrigem: o JWT é do usuário operador, e o `customer_id` vai no body ou no path. Conferi se nenhum documento ficou com a versão antiga. Como "body ou path" não foi decidido, virou questão em aberto no RFC, e o FDD propõe body e query string seguindo o que `order.schemas.ts` já faz.

5. **Filtro por `PENDING` que nunca dispararia.** Lendo `order.service.ts` e `order.status.ts`, a criação do pedido não passa por `changeStatus` e nenhuma transição leva a `PENDING`. Um webhook assinando `PENDING` nunca receberia nada, então o FDD restringe o filtro aos outros cinco status e explica o motivo.

6. **Onde checar o limite de 64KB.** Checar o tamanho na inserção da outbox faria a mudança de status dar rollback por causa de um payload grande, o que não faz sentido para a operação. A checagem foi para o worker: o evento não é enviado, não é truncado e vai direto para a DLQ, que é o "erra" que a Sofia pediu.

7. **Rotação de secret num webhook de saída.** "A antiga fica válida por 24 horas" é natural quando o cliente assina. Aqui quem assina somos nós. Pedi para a IA explicar o que isso significa na prática, e a proposta foi mandar duas assinaturas no `X-Signature` durante a carência, marcada como ponto para a revisão da Sofia.

8. **Ajustes da verificação automática.** O script mostrou que os arquivos novos propostos na reunião (`src/worker.ts`, `src/modules/webhooks/...`) apareciam como caminhos inexistentes. Deixei explícito em cada documento que são arquivos a criar. Também troquei um diagrama ASCII do RFC por Mermaid, que renderiza no GitHub e fica mais legível.

## Como navegar a entrega

Ordem sugerida de leitura:

1. [`docs/PRD.md`](docs/PRD.md): o problema, o público, o escopo e o que fica de fora. Comece por aqui para entender o porquê.
2. [`docs/RFC.md`](docs/RFC.md): a proposta técnica em nível de arquitetura, as alternativas descartadas e as questões em aberto.
3. [`docs/adrs/`](docs/adrs/README.md): uma decisão por arquivo.
   - [ADR-001 Outbox no MySQL](docs/adrs/ADR-001-outbox-no-mysql.md)
   - [ADR-002 Worker separado com polling](docs/adrs/ADR-002-worker-separado-com-polling.md)
   - [ADR-003 Retry com backoff e DLQ](docs/adrs/ADR-003-retry-com-backoff-e-dlq.md)
   - [ADR-004 HMAC-SHA256 com secret por endpoint](docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md)
   - [ADR-005 At-least-once com X-Event-Id](docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md)
   - [ADR-006 Reuso dos padrões do projeto](docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md)
   - [ADR-007 Payload enxuto como snapshot](docs/adrs/ADR-007-payload-enxuto-como-snapshot-na-insercao.md)
4. [`docs/FDD.md`](docs/FDD.md): como construir. Modelo de dados, fluxos, contratos, erros, observabilidade e integração com o código atual.
5. [`docs/TRACKER.md`](docs/TRACKER.md): de onde veio cada item, com timestamp da reunião ou caminho do código.

A transcrição original continua em [`TRANSCRICAO.md`](TRANSCRICAO.md), sem alterações.
