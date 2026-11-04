---
name: admanage-midia
description: Põe a mídia de um lote (vídeos ou imagens) no formato que o AdManage aceita para montar um draft — pasta do Dropbox, URL pública ou anúncios que já existem na Meta (por ad ID, da mesma conta ou de outra) — com o nome do arquivo virando o nome do anúncio, pares feed + story juntos e duplicados fora. Use no começo de qualquer subida ("pega essa pasta do Dropbox e monta o draft", "sobe esses vídeos/estáticos", "puxa esses ad IDs da outra conta", "reaproveita o post ID", "tem .MOV e .mp4 repetido"). Não use para escolher campanha, copy ou parceria (skills `campanhas-por-produto`, `admanage-copy-e-link`, `admanage-parcerias`) nem para lançar (`admanage-lancar`).
---

# Mídia para o draft do AdManage

Leia antes o perfil da marca (skill `perfil-da-marca`) e siga a skill `regras-do-play`.

O resultado desta skill é uma lista de mídias prontas para o `create_draft` (skill `admanage-draft`): para cada anúncio, o nome, a URL (ou o ID do criativo de origem), o tipo (vídeo ou imagem) e, nos pares feed + story, as duas mídias com largura e altura.

## 1. De onde vem a mídia

### Pasta do Dropbox (o caso mais comum)

1. Peça o **link compartilhado da pasta** (`dropbox.com/scl/fo/...`). Um caminho de espaço de time (`dropbox.com/work/...` ou `/home/...`) não é link compartilhado. Criar um link muda o compartilhamento da pasta: só com ok do piloto.
2. Liste com AdManage: `browse_dropbox` (`sharedLink`, `workspaceId` do perfil). Cada arquivo vem com um `launchUrl`.
3. **Use o `launchUrl` exatamente como veio** no `media[].url`. Ele passa pela integração do Dropbox do AdManage, de onde a Meta consegue baixar.
4. **Nunca use** link de preview (`dropbox.com/preview/...`), link `dl.dropboxusercontent.com` nem URL montada à mão. A Meta recusa: "O objeto promovido pelo anúncio é inválido" ou "Não foi possível baixar sua imagem". Exemplo da Gocase (27 e 28/10/2026): 241 de 241 e 168 de 212 anúncios recusados assim.
5. **Não renomeie nem mova arquivos depois de listar.** O `launchUrl` guarda o caminho do arquivo; se ele muda, o lançamento falha com "Dropbox download failed". Renomeie antes e liste de novo.
6. O `launchUrl` leva um token que expira em algumas horas. Se o lançamento não for logo, suba as mídias para a biblioteca do AdManage com `upload_media_from_url` (o `launchUrl` como `url`, o nome do arquivo como `displayName`) e use a URL `media.admanage...` que volta. Ela não expira. Para pastas grandes, `import_dropbox_folder` faz o mesmo em lote (até 200 arquivos por chamada).

Pasta com subpastas: navegue com `browse_dropbox` (`path`) ou use o `search`. A subpasta costuma dizer o produto ou o tipo (ex.: `TTS/<produto>`), e isso ajuda a skill `produto-do-anuncio`. Subpastas como `_reserva` ou `não subir` ficam de fora.

### URL pública ou arquivo do piloto

- `upload_media_from_url` com `displayName` = nome que o anúncio deve ter. Sem `displayName`, o nome vira um código ilegível.
- Arquivo que está só na máquina do piloto: peça que ele ponha numa pasta do Dropbox ou do Drive conectado ao AdManage. A skill não lê caminho local.

### Anúncios que já existem (por ad ID)

1. AdManage: `get_source_ad_setup` com `sources` = `[{adId, accountId}]` (a conta de **origem** de cada anúncio) e `fields: "lightweight"`. Vem a copy, o link, a identidade e um `media` pronto para o `create_draft`.
2. Se a mídia vier com URL da CDN da Meta (`fbcdn`), ela expira: passe por `upload_media_from_url` com o nome do anúncio de origem como `displayName`.
3. **Post ID:** por padrão, crie anúncio novo (não passe `effectiveStoryId` e use `recreateWithPostId: false`), a menos que o perfil diga outra coisa. Reaproveitar o post (mantém comentários e curtidas) só com pedido do piloto e só se a página do post serve no destino.
4. **Identidade:** se o anúncio de origem era de parceria, a linha vem com `partnershipInfo` e a identidade pode ser a do creator. Use a página e o Instagram da marca (perfil) e refaça a parceria com a skill `admanage-parcerias`.
5. **CTA de origem:** confira. Já veio `WATCH_LIVE_VIDEO`, que o lançamento recusa; a skill `admanage-copy-e-link` define o CTA.

Para copiar **todos os anúncios de um ad set** para outro (inclusive de outra conta), existe `copy_meta_ads_cross_account`. Ele cria anúncios direto na Meta (pausados por padrão), então pede ordem explícita.

## 2. Nome do anúncio

O nome do arquivo (ou do anúncio de origem) vira o nome do anúncio. Confira contra o padrão de nome do perfil **antes** de montar o draft. Lote fora do padrão: pergunte se mantém ou se corrige (regras-do-play §6). Nunca invente tag de fluxo que não é do processo.

## 3. Duplicados

Mostre o funil da contagem: arquivos na pasta → depois de cada filtro → anúncios.

- **Mesmo vídeo em `.MOV` e `.mp4`, com o mesmo nome:** fica um (o `.mp4`, salvo outra regra no perfil). Os dois sairiam em dobro.
- **Mesmo nome com sufixo `(1)`, `— Cópia`, `V2`:** pode ser o mesmo vídeo. Confira pelo tamanho ou pela imagem antes de descartar.
- **Já está no ar?** A pasta costuma acumular lotes. Procure o nome no dashRoi (`dim_meta_ad`, nome normalizado; skill `dashroi`) ou na Meta, na campanha de destino. Campanhas que o perfil marca como "não contam como já subido" (testes, por exemplo) não impedem.

## 4. Feed + story

Feed (4:5) e story (9:16) do mesmo criativo são **um anúncio só** (`type: "multi"`, as duas mídias no `media[]`).

- Passe `width` e `height` (vêm no retorno do `upload_media_from_url`) em cada mídia. Sem eles, o AdManage pode trocar feed por story.
- **Confirme o par pela imagem, não só pelo nome.** O agrupamento automático do AdManage já juntou criativos diferentes com nomes parecidos. AdManage: `view_image` mostra até 6 imagens por chamada (para vídeo, use a miniatura).
- Nomes diferentes (`X - Feed.png` e `X.png`) não se juntam sozinhos: junte você e use o nome sem sufixo.

## 5. Entrega

Uma tabela curta: quantos arquivos, quantos ficaram fora (e por quê), quantos anúncios, quantos pares feed + story. A lista de mídias segue para a skill `admanage-draft`.
