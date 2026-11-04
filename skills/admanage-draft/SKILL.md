---
name: admanage-draft
description: Cria, lê, edita e confere drafts do AdManage pelo conector (sem navegador) — montar o draft a partir da mídia, trocar copy, link, nome, página, ad set ou parceria em várias linhas de uma vez, e conferir que ficou certo. Use sempre que for mexer em linhas de um draft em qualquer processo ("monta o draft", "ajusta o draft", "troca o link/a página/o ad set dessas linhas", "confere o draft 12345", "tira essas linhas"). Não use para lançar (skill `admanage-lancar`) nem para decidir o conteúdo das linhas (as skills de produto, campanha, copy, parceria e página decidem; esta aplica).
---

# Draft do AdManage

Leia antes o perfil da marca e siga a skill `regras-do-play`. Criar e editar draft não muda nada na Meta: não precisa de ordem. Lançar precisa (skill `admanage-lancar`).

Todas as chamadas são do conector do AdManage. Passe o `workspaceId` do perfil onde a ferramenta aceitar.

## 1. Criar

1. **Identidade:** `get_launch_context` (`accountId`) mostra a página e o Instagram padrão da conta. Não confie no padrão: ele pode ser uma página cheia ou a errada para o lote. Passe sempre `page` e `insta` em cada anúncio, com o que a skill `capacidade-de-paginas` decidiu.
2. **Ad sets:** objetos `{value, label}` do `list_adsets` (skill `campanhas-por-produto`).
3. **`create_draft`** com `title` = padrão de nome de draft do perfil e um item em `ads[]` por anúncio:
   - `adAccountId`, `adSets`, `page`, `insta`;
   - `adName` = nome definitivo (renomear depois quebra a UTM);
   - `media` da skill `admanage-midia` (`type: "multi"` para feed + story);
   - copy e link da skill `admanage-copy-e-link` (`title`, `description`, `adDescription`, `link`, `cta`, `urlTags` e as variações);
   - `partnershipAd` da skill `admanage-parcerias`, quando houver.
4. Um anúncio em N ad sets vira N anúncios na Meta. Diga o total (linhas × ad sets) antes de criar.
5. **Lote grande:** acima de ~200 anúncios, divida em vários drafts. Exemplo da Gocase (09/10/2026): 815 de uma vez travaram e nada foi criado; lotes de 100 a 200 funcionaram.

## 2. Ler

- **Sempre com `fields`.** `get_draft` sem `fields` traz cada linha inteira (uns 5 KB por linha por causa dos ad sets): um draft de 90 linhas passa de 400 KB.
- Comece por `fields: ["title", "state.rows.id", "state.rows.customName"]` e acrescente só os campos de que precisa.
- O nome efetivo do anúncio na linha é `customName`. A parceria fica em `partnershipAd` e em `dynamicData`.
- Para saber o nome exato de um campo, leia **uma** linha inteira uma vez (`get_draft` com o `fields` dessa linha) e use os nomes que vierem. Não adivinhe.

## 3. Editar

1. Leia os campos que vai mudar, só das linhas afetadas, e guarde os valores antigos. É o seu desfazer.
2. Monte **todos** os patches e mande **numa chamada só** de `update_draft_rows` (`rowPatches: [{rowId, fields}]`). Ela devolve o draft inteiro e não aceita `fields`, então cada chamada a mais custa o draft todo de novo.
3. Campo comum a todas as linhas: `globalDefaults`.
4. Para tirar linhas, monte um draft novo só com as que ficam (`create_draft`), em vez de editar o original. O original vira backup.

## 4. Conferir

Depois de criar ou editar, `get_draft` com `fields` dos campos que mudaram e compare com o que devia estar. Feche a conta:

| Conferência | Esperado |
|---|---|
| Linhas | o número do funil |
| Linhas sem mídia ou sem ad set | 0 |
| Nome repetido no mesmo ad set | 0 (é quase sempre o mesmo vídeo duas vezes) |
| Página e Instagram | os decididos para cada linha |
| Parceria | a decidida para cada handle |
| Total de anúncios | Σ (linhas × ad sets) |

Diferença: corrija numa nova chamada única e confira de novo.

## 5. Draft montado pelo piloto

Se o piloto montou o lote na tela do AdManage, peça para ele clicar em **Save**: antes disso o draft só existe na aba dele. Depois, ache com `list_drafts` e siga pela leitura (§2).

Não apague drafts do piloto. Drafts intermediários que você criou podem ser apagados com `delete_draft` se o piloto quiser; avise quais são no fim.
