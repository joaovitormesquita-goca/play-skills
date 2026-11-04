---
name: campanhas-por-produto
description: Decide em qual campanha e em quais ad sets ativos da Meta cada produto de um lote deve entrar, pelo tipo de conteúdo — campanha própria do produto, a certa entre várias, campanha mista quando o produto não tem própria — e confirma o status ao vivo. Use em qualquer subida ("pra qual campanha vai esse conteúdo?", "acha as campanhas e ad sets de cada produto", "pega os IDs dos ad sets"). Não use para abastecer campanhas por desempenho (vencedores de teste em campanhas que precisam), que é outra tarefa.
---

# Campanhas e ad sets por produto

Leia antes o perfil da marca (contas, tipos de campanha, destino por tipo de conteúdo, campanhas excluídas) e siga as skills `regras-do-play` e `dashroi`. A entrada é o mapa anúncio → produto (skill `produto-do-anuncio`) e o tipo de conteúdo do lote.

## 1. Candidatas

Campanhas da conta com gasto recente cujo nome bate com o produto (use os sinônimos do dicionário do perfil):
- **dashRoi** (skill `dashroi`): campanhas com gasto nos últimos 7 dias, por nome normalizado. O dashRoi não tem o dia corrente nem campanha recém-criada.
- **Plaisson**, se a marca usa: `get_brand_dashboard` mostra as campanhas e o produto de cada uma.
- **Campanha ou ad set criado hoje:** AdManage `list_adsets` com `refresh: "true"` (e `accountId` ou `campaignId`) ou Meta.

Tire as campanhas que o perfil exclui (testes, catálogo, reconhecimento...).

## 2. Escolher

Para cada produto:

1. **O perfil tem destino para este tipo de conteúdo?** Use (ex.: TTS numa campanha própria de vídeos gerais; Big Influs numa campanha de influ).
2. **Uma campanha própria do produto** → ela.
3. **Várias** → a que bate com o tipo de conteúdo pelo nome (ex.: "Humanizado" para UGC). Sem pista no nome, a que mais recebeu anúncios desse tipo (dashRoi, contagem de anúncios pelo padrão de nome). Ainda empatado, pergunte ao piloto, com uma recomendação.
4. **Nenhuma própria** → a campanha **mista** ativa que já recebe anúncios do produto (ex.: a campanha da linha).
5. **Nenhuma ativa** → pergunte se o produto sai do lote. Nunca jogue numa campanha genérica por conta própria.

Campanha própria sem gasto nos últimos dias está parada: não serve como destino sem ok do piloto.

## 3. Ad sets

AdManage: `list_adsets` com `campaignId`, `status: "ACTIVE"`, `detailsOnly: true` e `fields` curtos. Cada ad set vem com `value` e `label`, que vão direto no draft.

- Quais ad sets usar vem do perfil (ex.: o de aquisição e o de clientes da mesma campanha).
- Campanha com um só ad set ativo → só ele.
- Não descarte ad set por número de anúncios: o limite aparece no lançamento (skill `admanage-lancar`).

## 4. Confirmar ao vivo

Logo antes de montar o draft, confira que cada campanha e ad set escolhido está **ativo hoje** (`list_adsets` com `refresh: "true"`, ou a Meta). O piloto pode ter desligado algo hoje.

## 5. Entrega

Uma tabela única: **produto → campanha (ID) → por quê → ad sets → nº de anúncios**. Marque com ⚠️ só o que foi escolha sua (mista, desempate, linha). Depois de aprovada, siga sem nova checagem, a menos que o perfil peça.

A saída é o mapa produto → ad sets (`{value, label}`), usado pela skill `admanage-draft`.
