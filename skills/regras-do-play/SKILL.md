---
name: regras-do-play
description: Regras de operação de mídia do Gogroup que valem para qualquer marca e qualquer tarefa — o que pede ordem explícita, como mostrar um lote antes de executar, como conferir depois, o que nunca sobe (condição comercial vencida, produto fora de linha), como contar criativos sem duplicar e como não errar ROI. Use SEMPRE antes de mexer em campanha, anúncio, orçamento ou draft (Meta, AdManage, Plaisson), e quando o piloto perguntar "posso fazer X direto?", "qual é a regra para…", "como vocês fazem…". As outras skills do play-skills seguem estas regras.
---

# Regras do Play

Estas regras valem para todas as marcas. O que muda de marca para marca (réguas, nomes, contas, preferências de autonomia) fica no perfil da marca: leia-o antes, com a skill `perfil-da-marca`. Se o perfil der mais autonomia ou mais cautela do que o padrão abaixo, vale o perfil.

## 1. O que pede ordem explícita

Só faça com ordem clara do piloto **para aquela entidade ou aquele lote**:
- lançar um draft, publicar ou ativar campanha, ad set ou anúncio;
- mudar orçamento ou lance;
- arquivar ou apagar qualquer coisa;
- enviar mensagem ou pedido (de parceria, por exemplo) em nome do piloto.

Pedidos como "não executa ainda", "só planeja", "monta sem lançar" e "vou revisar antes" travam a execução até nova ordem. Uma ordem não se estende ao lote seguinte.

**Para tirar do ar, pause.** Pausar se desfaz; arquivar e apagar não.

## 2. Antes de um lote

1. **Panorama em tabela** (por produto, campanha ou tipo), com o total e o que vai acontecer com cada grupo.
2. **Uma única rodada de dúvidas**, com todos os ambíguos juntos. Escale só o que precisa do piloto; o resto, decida pelo método e diga o que decidiu.
3. **Mostre o funil da contagem** a cada filtro (ex.: 99 anúncios → 24 sem marcador → 22 com produto identificado). Número que muda sem explicação gera desconfiança no lote inteiro.
4. **Preview antes de confirmar.** Se a ferramenta tem modo de prévia (ex.: Plaisson `schedule_budget_batch` sem `confirm`), rode antes. Nunca confirme com erro na prévia.

## 3. Logo antes de executar

- **Confira o status ao vivo** das campanhas e ad sets de destino. O piloto pode ter desligado algo hoje.
- **Liste a pasta de origem de novo**: pode ter entrado arquivo depois do panorama.
- **Confira na Meta se o lote já está no ar**, para não subir duplicado.
- **Respeite a ordem dos lotes** que o piloto pediu.

## 4. Depois de executar

- **Releia o que mudou.** A Meta passa por `IN_PROCESS` antes do status final: confira de novo após ~30 s. Agendamentos devem aparecer como `pending`.
- **Feche a conta:** quantos foram, quantos falharam e por quê, o que ficou para o piloto.
- Relance ou refaça **só o que falhou**, nunca o lote inteiro.

## 5. O que nunca sobe

- **Condição comercial que não está vigente no site:** preço diferente, cupom que não existe mais, promoção datada que acabou, "só hoje", frete grátis que era teste. Mencionar um brinde só no texto do anúncio não impede; mostrar no vídeo ou na imagem, sim.
- **Produto fora de linha ou fora do site.**
- **Conteúdo sazonal vencido** (data comemorativa que passou).

Na dúvida sobre a condição vigente, pergunte ao piloto. Na dúvida sobre um vídeo, procure um jeito de usar (cortar, cobrir) antes de descartar.

## 6. Contar e nomear criativos

- **Feed + story do mesmo criativo é um anúncio só**, com posicionamentos. Confirme o par pela imagem, não só pelo nome.
- **O mesmo criativo com nomes diferentes conta como um conteúdo.** Sufixos e marcas de cópia como `— Cópia`, `(1)`, `_Sub_NN`, `V2`, versões com e sem headline, e o marcador de "já promovido" do perfil.
- **Rascunho nasce com o nome definitivo.** Renomear campanha ou ad set depois quebra a UTM e a leitura de atribuição.
- **Padrões de nome e tags de fluxo vêm do perfil.** Lote fora do padrão: pergunte antes de lançar se mantém.

## 7. Dados e ROI

- **Status sempre pela Meta; histórico pelo dashRoi** (skill `dashroi`). O dashRoi atrasa e não tem o dia corrente completo.
- **Dado tem data.** Lista, cache ou tabela de outra conversa envelhece: releia ao vivo antes de agir.
- **ROI é sempre Σ receita ÷ Σ gasto.** Nunca faça média de ROIs.
- **Compare dias com o mesmo corte horário** (ex.: até as 14h nos dois).
- **Diga qual ROI está usando** (atribuição, produto, ROAS da Meta). Eles respondem perguntas diferentes: ver a skill `dashroi`.
- **Uma variável por vez** ao testar uma mudança (verba, cupom, criativo).

## 8. Armadilhas de ferramenta que valem para todas as marcas

- **Mudar orçamento de campanha ativa pelo conector da Meta** (`ads_update_entity` com `daily_budget`) pode deixar a campanha pausada. O valor vai em centavos. Avise antes e confira o status depois.
- **Renomear anúncio pela Meta** exige publicar a mudança: use `publish_as_active=false` para não ativar o que estava pausado.
- **Não varra a conta inteira na Meta** (dá erro e é lento). Ache os IDs pelo dashRoi e consulte a Meta por ID.

## Formato da resposta

Curta. Primeiro o resultado ou a decisão pendente, depois a tabela. Só os alertas que pedem ação do piloto. Listas que o piloto vai copiar (IDs, handles, nomes) vão em bloco de código.
