---
name: subir-conteudo-humanizado
description: Processo de ponta a ponta para subir conteúdo humanizado como anúncios Meta pelo AdManage — vídeos de creators (UGC), de afiliados do TikTok Shop (TTS) e de influenciadores — do link da pasta ao draft pronto (produto, campanha, parceria, copy, página) e, só com ordem, ao lançamento. Use quando o piloto mandar um link de pasta de conteúdo e disser "sobe esses vídeos de creators", "sobe os TTS", "sobe o conteúdo dos influs", "monta o draft desse conteúdo", ou pedir o processo completo. Não use para estáticos de estúdio ou de IA sem rosto, nem para abastecer campanhas com vencedores de teste.
---

# Subir conteúdo humanizado

Este processo orquestra as skills de bloco. Leia o perfil da marca (skill `perfil-da-marca`) e as regras (skill `regras-do-play`) antes de começar. Passos que só montam ou editam draft não pedem licença; lançar pede ordem explícita.

## Variantes

O perfil diz, para cada uma: padrão de nome, onde está o handle, destino e política de parceria.

| Variante | O que muda |
|---|---|
| **Creator (UGC)** | Parceria por creator; destino costuma ser a campanha humanizada do produto |
| **Afiliado TikTok Shop (TTS)** | Costuma ir sem parceria e numa campanha própria de vídeos gerais; o produto sai do nome do arquivo |
| **Influenciador** | Separe os **Big Influs** (campanha própria de influ) dos influs por produto (campanha do produto). Parceria só quando o influ liberou |

Lote com mais de uma variante: trate cada uma como um lote.

## Passos

| # | Passo | Skill | Para para perguntar? |
|---|---|---|---|
| 1 | Pegar o link da pasta (do piloto, ou de onde o perfil diz que o conteúdo é postado) | — | se houver mais de um link candidato |
| 2 | Listar a mídia, contar, tirar duplicados e o que já está no ar | `admanage-midia` | — |
| 3 | Produto de cada vídeo | `produto-do-anuncio` | duvidosos, numa rodada |
| 4 | Campanha e ad sets por produto e variante | `campanhas-por-produto` | produto sem campanha ativa: sai? |
| 5 | Tirar do lote os produtos que saíram (antes da parceria, para não perguntar de handle descartado) | — | — |
| 6 | Parcerias por handle (pule se a variante vai sem parceria) | `admanage-parcerias` | parecidos e sem parceria, numa rodada |
| 7 | Copy e link por produto | `admanage-copy-e-link` | ambíguos e alertas |
| 8 | Página | `capacidade-de-paginas` | se não couber ou o perfil pedir |
| 9 | **Panorama** e uma única rodada com as dúvidas que sobraram (regras-do-play §2) | — | sim, se houver dúvida |
| 10 | Criar o draft com nomes definitivos (tag de fluxo do perfil já no nome) e conferir | `admanage-draft` | — |
| 11 | Lançar, **só com ordem explícita** | `admanage-lancar` | sim |
| 12 | Avisar quem mandou o conteúdo, só com ok do piloto | conector de chat da marca | sim |

Junte as perguntas dos passos 3, 4, 6 e 7 numa rodada só sempre que der.

## Panorama (passo 9)

Antes de criar o draft:
- o funil da contagem (arquivos → sem duplicados → sem já subidos → produtos com campanha → anúncios);
- a tabela **produto → campanha → ad sets → nº de vídeos → nº de anúncios**;
- parcerias (com, padrão, sem) e página;
- o que ficou de fora e por quê.

## Logo antes de lançar

- Liste a pasta de novo: pode ter entrado arquivo depois do panorama.
- Confira que o lote não está no ar (skill `admanage-lancar` §1).
- Respeite a ordem dos lotes que o piloto pediu.

## Entrega

- Nº do draft (e do batch, se lançou).
- Tabela produto → campanha → nº de anúncios.
- Parcerias: handle → parceria (ou sem), e em bloco de código os handles sem parceria.
- Página → nº de anúncios.
- O que ficou de fora e por quê.
- Lembrete para o piloto olhar o Preview de um anúncio de parceria antes de lançar.
