# ADR-004: Assinatura HMAC-SHA256 com secret por endpoint e rotação com 24h de carência

**Status:** Aceito
**Data:** 26/09/2026
**Decisores:** Sofia (Segurança), Larissa (Tech Lead), Diego (Eng. Sênior, Plataforma), Bruno (Eng. Pleno, Pedidos)
**Relacionados:** [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Contexto

Os eventos carregam dados de pedidos e vão para endpoints fora da nossa infraestrutura. O cliente precisa conseguir confirmar que a requisição saiu da gente e que ninguém alterou o payload no caminho ([09:19] Sofia). O webhook é só de saída, da plataforma para o cliente ([09:02] Marcos, [09:03] Sofia).

Já tivemos um cliente que vazou uma secret em log da aplicação dele ([09:22] Diego), então a estratégia precisa limitar o estrago de um vazamento e permitir troca sem parar a integração.

## Decisão

1. Cada requisição é assinada com **HMAC-SHA256 sobre o corpo do request**, e a assinatura vai no header `X-Signature` ([09:20] Sofia, [09:22] Sofia).
2. **Cada endpoint de webhook tem a sua própria secret.** Não existe secret global da plataforma ([09:21] Sofia). A configuração do webhook guarda url, secret, customer_id e estado ativo ([09:21] Bruno).
3. A secret é **gerada pela plataforma e devolvida ao cliente na criação** do webhook ([09:31] Marcos).
4. A secret é **rotacionável pela API**. Depois da rotação, a secret antiga continua válida em paralelo por **24 horas**, e depois disso deixa de valer ([09:21] Sofia).
5. A URL cadastrada precisa ser **https**. Cadastro com http é recusado com erro de validação ([09:23] Sofia).

Como a assinatura é gerada do nosso lado, "a antiga continua válida" se traduz assim na implementação: durante as 24 horas de carência, o `X-Signature` leva duas assinaturas, uma com a secret nova e outra com a antiga, e o cliente aceita se qualquer uma bater. O formato exato está no FDD e passa pela revisão da Sofia antes do deploy ([09:46] Sofia).

## Alternativas Consideradas

**Secret global da plataforma.** Descartada pela Sofia: se vaza uma, vaza tudo ([09:21] Sofia).

**Rotação com troca imediata, sem carência.** Não atende o caso real de o cliente precisar de tempo para atualizar os sistemas dele. A carência de 24h existe justamente para isso ([09:21] Sofia).

**Outros algoritmos de assinatura.** A pergunta sobre o algoritmo foi feita ([09:20] Bruno) e a escolha ficou em SHA-256 por ser o padrão de mercado, com biblioteca disponível em qualquer stack séria dos clientes ([09:20] Sofia).

## Consequências

**Positivas**
- O cliente consegue validar origem e integridade com ferramentas padrão.
- Um vazamento fica restrito a um endpoint e se resolve com rotação, sem downtime.
- Não depende de nenhuma biblioteca nova. O módulo `crypto` do Node já faz HMAC.

**Negativas**
- A secret precisa ficar guardada de forma recuperável no banco, porque é necessária para assinar cada envio. Não dá para guardar só um hash, como fazemos com `passwordHash` no modelo `User`.
- Por 24 horas convivem duas secrets por endpoint, o que deixa o envio e a verificação do cliente um pouco mais complexos.
- A assinatura cobre só o corpo. A proteção contra replay com o `X-Timestamp` fica a cargo do cliente, se ele quiser ([09:44] Diego).

**Trade-off:** aceitamos o custo de guardar e rotacionar uma secret por endpoint em troca de isolar vazamentos e dar ao cliente uma verificação padrão de mercado.
