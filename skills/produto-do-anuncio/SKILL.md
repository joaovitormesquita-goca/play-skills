---
name: produto-do-anuncio
description: Descobre de qual produto é cada anúncio, vídeo ou estático de um lote — primeiro pelo nome, depois pela pauta do creator e, por último, olhando a imagem — usando o dicionário de produtos do perfil da marca, e junta os casos duvidosos numa rodada só. Use antes de escolher campanha e copy em qualquer subida, e quando o piloto perguntar "de qual produto são esses?", "identifica o produto desses anúncios", "quantos são de cada produto?". Não use para decidir a campanha (skill `campanhas-por-produto`).
---

# Produto do anúncio

Leia antes o perfil da marca (dicionário de produtos, padrões de nome) e siga a skill `regras-do-play`. O produto decide a campanha, a copy e o link: errar aqui erra o lote inteiro.

## 1. Pelo nome (resolve a maior parte)

1. O padrão de nome do perfil diz onde o produto está em cada tipo de conteúdo (ex.: um trecho "- produto X" no nome do UGC, o prefixo do arquivo no TTS, o SKU no estático gerado).
2. Normalize com o **dicionário do perfil**: sinônimos e grafias levam ao nome canônico, e a coluna "não confundir com" separa produtos de nome parecido.
3. A subpasta de origem também conta (ex.: `TTS/<produto>`).
4. Produto **fora de linha** no dicionário: o anúncio não sobe (regras-do-play §5). Avise.

## 2. Pela pauta

Se o nome traz o número ou o título da pauta e a marca usa o GoHits:
- GoHits: `gocreators_briefings` (`brand_id`, `search` com o título ou o número) traz os produtos da pauta;
- para anúncios que já existem na Meta (ex.: vencedores de teste), GoHits `gocreators_identify_products` (`brand_id`, até 20 `meta_ad_ids` por chamada) assiste ao vídeo e escolhe entre os produtos do catálogo. Na primeira vez é lento (10 a 30 s por vídeo); itens que voltarem em `pending` vão numa nova chamada só com eles.

Em conteúdo de influenciador, o cadastro de contratos (InfluHub, por exemplo) sozinho não basta: o influ pode falar de mais de um produto. Confirme pela pauta ou pela imagem.

## 3. Pela imagem

Para o que ainda ficou sem produto:
- AdManage: `view_image` com até 6 URLs por chamada (para vídeo, a miniatura);
- compare com a foto do produto no site ou no catálogo;
- produtos de formato parecido (bolsas, garrafas, capinhas) costumam ser confundidos. Na dúvida, não chute: vai para a rodada de perguntas.

## 4. Casos especiais

- **Anúncio com mais de um produto:** liste todos. A skill `campanhas-por-produto` decide o destino.
- **Produto do nome diferente do produto da imagem:** pergunte; não corrija sozinho.
- **Produto que não está no dicionário:** pergunte e sugira acrescentar ao perfil.

## 5. Entrega

Antes de seguir, mostre:
1. o funil (ex.: 84 anúncios → 70 pelo nome → 9 pela pauta → 5 pela imagem);
2. a tabela **Produto | Qtd | Como foi identificado**;
3. **uma única rodada** com todos os duvidosos, cada um com a imagem ou o nome e o palpite.

A saída é o mapa anúncio → produto (o nome canônico do dicionário), usado como chave pelas skills `campanhas-por-produto` e `admanage-copy-e-link`.
