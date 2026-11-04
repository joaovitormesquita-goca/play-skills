---
name: admanage-lancar
description: Lança na Meta um draft do AdManage já montado, com segurança — pré-voo (nomes repetidos, anúncio já no ar, ad sets e campanhas ativos, página com vaga, tamanho do lote), um único lançamento, tratamento de ad set cheio e de limite de página, acompanhamento do batch, relançamento só do que falhou e relatório. Use SOMENTE quando o piloto pedir explicitamente para lançar este draft ("pode lançar", "sobe pro ar", "lança o draft 12345"). Para montar ou corrigir o draft, use `admanage-draft`.
---

# Lançar draft do AdManage

Lançar cria anúncios reais e gasta verba. Só com ordem explícita do piloto **nesta conversa, para este draft** (regras-do-play §1). Uma ordem não vale para o lote seguinte.

**Nunca chame `launch_draft` duas vezes para o mesmo draft depois de um sucesso.** Se não tiver certeza se o lançamento anterior pegou, procure o batch (`list_batches` ou `get_batch_status`) antes de qualquer nova tentativa.

## 1. Pré-voo

Leia o draft com `get_draft` e `fields` (skill `admanage-draft`) e confira:

1. **Linhas e total:** nº de linhas e Σ (linhas × ad sets) batem com o panorama aprovado.
2. **Mídia e ad set** em todas as linhas.
3. **Nome repetido no mesmo ad set:** quase sempre o mesmo vídeo duas vezes. Resolva antes (skill `admanage-midia` §3).
4. **Já está no ar?** AdManage: `check_launch_conflicts` com os nomes e ad sets. Anúncio com o mesmo nome no mesmo ad set: pergunte antes de seguir.
5. **Destino ativo hoje:** status ao vivo das campanhas e ad sets (`list_adsets` com `campaignId` e `refresh: "true"`, ou a Meta). O piloto pode ter desligado algo hoje.
6. **Página com vaga:** skill `capacidade-de-paginas`.
7. **Tamanho:** acima de ~200 anúncios, divida em drafts menores e lance um por vez.
8. **Pausado ou ativo:** o padrão é ativo. Se o piloto quer os anúncios criados pausados, o campo `launchPaused` vai no draft (precisa ser posto antes).

Mostre o resumo em uma tabela (draft, anúncios, campanhas, página) e lance.

## 2. Lançar

AdManage: `launch_draft` com o `id` do draft.

- **Sucesso:** volta `adBatchId`. Siga para o §4.
- **Erro de validação** (`ok: false`, `category: "validation"`): **nada foi criado**. Leia a mensagem, corrija o draft e lance de novo. Não repita a mesma chamada sem corrigir.
- Se o ambiente bloquear o lançamento (alguns modos do Claude pedem confirmação extra para ações com dinheiro), não contorne: deixe o draft pronto, peça ao piloto para lançar pela tela do AdManage e acompanhe pelo batch.

## 3. Ad set cheio e página cheia

### Ad set cheio

O `launch_draft` recusa quando um ad set passa do limite da Meta e diz quanto passa, por exemplo `"<ad set>" (150/150 current, +1 new, over by 1)`. O limite varia por ad set (já apareceram 150 e 50). Opções, nesta ordem:

1. **Outro ad set ativo da mesma campanha**, se o perfil permitir para aquele tipo de conteúdo.
2. **Arquivar anúncios fracos do ad set cheio**, só se o perfil autorizar e com o critério do perfil (ex.: mais de N dias, sem conversão, menor gasto). Liste os candidatos com nome, idade, gasto e conversões. Sem autorização no perfil, a lista vai para o piloto, que arquiva ou autoriza.
3. **Tirar do draft** as linhas que não cabem e avisar.

Arquivar se faz na Meta. Nunca apague.

### Página cheia

"Meta ad volume limit reached" ou falha por limite de anúncios da página: skill `capacidade-de-paginas`. Troque a página só das linhas afetadas.

## 4. Acompanhar

AdManage: `get_batch_status` com o `adBatchId` e `waitSeconds: 25`, repetindo até `summaryStatus` ser `success` ou `error`. Um lote de ~100 a 200 anúncios leva alguns minutos.

A resposta final traz cada anúncio e pode ser muito grande. Leia o resumo (`totalAds`, `successfulAds`, `failedAds`, `failedDetails`) e não despeje a lista no chat.

## 5. Falhas

Relance **só o que falhou**: monte um draft com essas linhas (skill `admanage-draft`) e lance. Nunca relance o lote inteiro.

| Falha | O que fazer |
|---|---|
| "Meta couldn't create the ad creative… tente novamente mais tarde" (erro genérico, código 100) | Costuma ser passageira: relance uma vez |
| "O tipo de imagem usado não é aceito" com vídeo comum | Já foi passageira: relance uma vez; se repetir, pergunte |
| "Dropbox download failed" | O arquivo foi renomeado ou movido depois de listado: liste de novo e refaça só essas mídias |
| "Não foi possível baixar sua imagem" / "objeto promovido inválido" | Link de mídia errado (preview do Dropbox): skill `admanage-midia` §1 |
| "PAGE AND INSTAGRAM ACCOUNT DON'T MATCH" | Problema na conta do creator: skill `admanage-parcerias`; pergunte se vai com a parceria padrão ou sem |
| Limite de anúncios da página | Skill `capacidade-de-paginas` |

Anúncio criado que não entrega: AdManage `get_delivery_errors` com os IDs (até 50 por chamada). Status `IN_PROCESS` logo depois do lançamento é a revisão da Meta, não erro.

## 6. Relatório

- Batch, anúncios criados / total, relançamentos.
- Tabela: campanha → nº de anúncios; página → nº de anúncios.
- O que falhou e por quê, e o que ficou para o piloto.
- Drafts intermediários que podem ser apagados.

Avisar alguém no chat (ex.: quem mandou o conteúdo) é mensagem em nome do piloto: só com ok dele.
