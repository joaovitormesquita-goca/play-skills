---
name: capacidade-de-paginas
description: Escolhe em qual página do Facebook cada anúncio de um lote sai, respeitando o limite de anúncios por página da Meta — página padrão do perfil, divisão do lote quando não cabe, páginas que não servem no país — e corrige só as linhas que falharem por limite de página no lançamento. Use antes de lançar lotes grandes, quando o lançamento avisar "Meta ad volume limit reached" ou falhar por limite de página, e quando o piloto pedir "acha uma página com vaga", "divide entre páginas". Não use para mexer na página de anúncio que já está no ar.
---

# Capacidade de páginas

Leia antes o perfil da marca (páginas, página padrão por conta, páginas vetadas, quem escolhe a página) e siga a skill `regras-do-play`.

## Como o limite funciona

- A Meta limita quantos anúncios cada página tem **em veiculação ou em análise**. Pausados não contam.
- O limite muda de página para página (ex.: na Gocase em 01/10/2026 havia páginas de 250, de 1.000 e de 5.000).
- Anúncio de parceria conta na página da marca.
- Outras pessoas lançam na mesma página enquanto você monta o lote: deixe folga.

## 1. Saber quanto cabe

Nenhum conector lê a tela de limites da Meta (`facebook.com/ads/ad_limits`). Então:

1. **Lote pequeno e página padrão com folga conhecida:** siga com a página padrão do perfil.
2. **Lote grande ou página que costuma encher:** peça ao piloto os números da tela de limites (ativos / limite das páginas candidatas) ou que ele escolha a página.
3. **Páginas que podem receber anúncio:** AdManage `list_pages` da conta (campo `promotionEligible`). Tire as que o perfil veta: internacionais, de outro país ou vetadas para aquele produto.

## 2. Decidir

- **Quem escolhe:** o perfil diz. Se o piloto delegou, escolha a página do país com mais folga e só avise qual foi. Se não, pergunte.
- **Não cabe numa página só:** divida. Mova primeiro as linhas **sem** parceria; mova linhas com parceria só se precisar. Não passe do que cabe no destino, com folga.
- O Instagram da marca continua o mesmo; só muda a página.
- **Nunca pause, arquive ou apague anúncios para abrir vaga.** Isso é decisão do piloto.

Aplique a página nas linhas com a skill `admanage-draft` (na criação ou com `update_draft_rows`) e diga o total por página.

## 3. Se o lançamento falhar por limite

Quando o `launch_draft` ou o batch apontar limite de anúncios da página (ex.: "Meta ad volume limit reached for <página> — X/250"):

1. Veja quais linhas falharam e em qual página (skill `admanage-lancar` §4).
2. Troque a página **só dessas linhas** para outra elegível com folga.
3. Relance só essas linhas.

## 4. Entrega

Tabela: página → nº de anúncios → por quê (padrão, folga informada pelo piloto, divisão). Avise se alguma página ficou perto do limite.
