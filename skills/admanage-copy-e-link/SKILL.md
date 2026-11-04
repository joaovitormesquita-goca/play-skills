---
name: admanage-copy-e-link
description: Escolhe a copy (headline, texto, descrição, CTA) e o link de destino de cada anúncio de um lote no AdManage — template de copy por produto, templates do evento quando há evento, PDP, LP da linha ou home pela regra da marca, UTM igual à dos lotes anteriores — e sinaliza preço fixo, promoção vencida e link no texto diferente do link do anúncio. Use em qualquer subida ("aplica os templates por produto", "ajusta a copy", "qual link vai nesse anúncio?", "coloca a PDP de cada produto", "usa os templates da Black"). Não use para escrever copy nova de campanha do zero sem pedido, nem para trocar link de anúncio que já está no ar (é outra tarefa).
---

# Copy e link por produto

Leia antes o perfil da marca (templates em uso, regra de link, UTM, CTA, copies avulsas aprovadas) e siga a skill `regras-do-play`. A entrada é o produto de cada anúncio (skill `produto-do-anuncio`).

## 1. Achar o template

1. AdManage: `list_ad_copy_templates` com `adAccountId` e `fields: ["id", "name"]` (a lista completa é grande).
2. Candidatos: nome que contém o produto ou um sinônimo do dicionário do perfil.
3. **Evergreen primeiro:** o de nome simples do produto. Fora de evento, evite templates de evento, de promoção, de data comemorativa ou com preço; use um desses só se for o único, e pergunte.
4. **Durante um evento**, os templates do evento vêm primeiro (o piloto ou o perfil diz qual é a tag). Acabou o evento, volte ao evergreen.
5. Produto sem template próprio: o genérico da linha (ex.: "Mochilas"). Não achou: siga a preferência do perfil para produto incerto ou pergunte.
6. Mais de um evergreen para o mesmo produto: pergunte, com o recomendado primeiro.
7. Leia só os escolhidos: `get_ad_copy_template` com `id`.

Mapeie os campos do template para o anúncio (`title` e `headlineVariations`, `description` e `bodyVariations`, `adDescription`, `link`, `cta`, `urlTags`) pelos nomes que o template devolver; confira numa linha antes de aplicar ao lote.

## 2. Sinalizar (sem corrigir por conta própria)

Antes de aplicar, olhe o texto de cada template escolhido e avise:
- **preço escrito no texto** ("a partir de R$ X"): confira se é o preço vigente;
- **promoção, brinde, cupom ou data** que pode ter acabado (regras-do-play §5);
- **descrição que não combina** com o texto;
- **URL dentro do texto diferente do link do anúncio**.

Mudar texto de template é decisão do piloto.

## 3. Link

A regra vem do perfil, por produto ou por linha:
- **PDP** do produto: pelo conector de catálogo da marca (perfil). Confira que a página abre e é do produto certo (o endereço nem sempre segue o nome). Produto fora do site → regras-do-play §5.
- **LP da linha**: em evento, a LP do evento, com o cupom se houver.
- **Home**: quando o produto é incerto ou a regra do perfil manda.

**UTM:** a mesma dos lotes anteriores da mesma campanha (leia de um draft recente com `get_draft` e `fields`, ou do template). UTM diferente quebra a leitura de atribuição.

## 4. Copy avulsa

Sem template, a copy avulsa só entra com aprovação do piloto, no tom dos templates da marca e sem preço. O que for aprovado, sugira registrar no perfil ou virar template (AdManage `create_ad_copy_template`, com ordem do piloto).

## 5. Entrega

Tabela antes de aplicar: produto → template (ou avulsa) → link → alertas. Uma rodada de dúvidas com os ambíguos. Depois, aplique na criação do draft ou com `update_draft_rows` (skill `admanage-draft`).
