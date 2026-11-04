---
name: perfil-da-marca
description: Cria, atualiza ou lê o perfil de uma marca do Gogroup (Gocase, Apice, Barbours, Rituaria, Kokeshi, Lescent, By Samia, Aua, Denavita, Yenzah...). O perfil guarda contas de anúncio, páginas, slug no Plaisson, padrão de nome das campanhas e as réguas da marca, e as outras skills do play-skills leem os dados da marca dali. Use quando o piloto disser "monta o perfil da minha marca", "configura a marca", "atualiza a régua de teste da marca", "qual é a conta/página da marca X", ou quando outra skill precisar de um dado da marca e não houver perfil. Não use para analisar resultado nem para mexer em campanha.
---

# Perfil da marca

O perfil é a única parte específica de cada marca. As skills do plugin descrevem o método. O perfil diz com quais contas, páginas, campanhas e réguas esse método roda nesta marca.

Esta skill **só lê** os conectores. Ela não cria, pausa nem altera nada na Meta, no AdManage ou no Plaisson.

## 1. Achar um perfil que já existe

Antes de criar, procure nesta ordem:
1. Na conversa ou no conhecimento do projeto, um documento com o título "Perfil da marca".
2. Na pasta de trabalho, um arquivo `perfil-marca-<slug>.md`.

Achou e o pedido é só consultar: responda com o dado e diga a data do campo (`atualizado em`). Campo com mais de 60 dias ou vazio: avise o piloto e sugira confirmar.

Achou e o pedido é atualizar: mude só os campos pedidos, ponha a data de hoje e a fonte em cada um, e mostre o antes e o depois.

## 2. Criar ou completar o perfil

Use o modelo em `references/modelo-perfil.md`. Preencha nesta ordem:

**a. Identificar a marca.** Pergunte qual marca, se não estiver claro. Uma marca pode ter mais de uma conta e mais de um slug (ex.: conta principal e complementar). Nesse caso, liste todos.

**b. Ler o que der nos conectores.** Só leitura, e só nos conectores que estiverem ligados:

| Dado | Onde ler |
|---|---|
| Slug(s) da marca e campanhas automatizadas | Plaisson: `list_brands`, depois `get_brand_dashboard(slug)` |
| Contas de anúncio | AdManage: `list_ad_accounts`, ou Meta: `ads_get_ad_accounts` |
| Páginas e Instagram | AdManage: `list_pages`, ou Meta: `ads_get_ad_account_pages` |
| Parcerias de creators | AdManage: `list_partnership_permissions` |
| Templates de copy | AdManage: `list_ad_copy_templates` |
| Grupos de ad sets favoritos | AdManage: `list_favorite_adset_groups` |
| Padrão de nome das campanhas | Os nomes das campanhas ativas em `get_brand_dashboard(slug)`: descreva o padrão com 2 ou 3 exemplos |

Os valores de regras e motores do Plaisson (`get_rules`) **não** vão para o perfil. Eles mudam e se leem sempre ao vivo. No perfil fica só o slug.

Se um conector não estiver ligado, diga qual faltou e pergunte o dado ao piloto.

**c. Perguntar o que não sai dos sistemas.** Uma seção por vez, sem despejar o questionário inteiro de uma vez:
- Para que serve cada conta e cada página (compra, teste, influ...).
- As réguas da marca: quando um teste é aprovado, quando um anúncio é cortado, o ROAS de referência por tipo de campanha.
- O que pede confirmação antes de executar, além do padrão do plugin.
- A parceria padrão quando o creator não tem parceria.

**Nunca invente uma régua nem copie a de outra marca.** Se o piloto não souber, deixe `a definir` no campo.

**d. Toda régua leva valor, data e fonte.** Exemplo: `gasto > R$ 20 e CTR > 2% (piloto, 04/11/2026)`. Sem data e fonte, não entra.

## 3. Guardar

- **Pasta local (Claude Code ou Cowork):** salve `perfil-marca-<slug>.md` na pasta de trabalho.
- **claude.ai (web ou celular):** entregue o perfil como um documento e oriente o piloto a colá-lo no conhecimento do projeto da marca. Assim as próximas conversas desse projeto já o encontram.

No fim, mostre uma tabela com o que veio dos conectores, o que o piloto informou e o que ficou `a definir`.

## Regras

- Nada de token, senha ou chave de API no perfil. IDs de conta, página, campanha e ad set podem entrar.
- Um perfil por marca. Se a marca tem duas contas, as duas entram no mesmo perfil.
- O perfil é do piloto. Proposta de mudança no método vai para o repositório do plugin, não para o perfil.
