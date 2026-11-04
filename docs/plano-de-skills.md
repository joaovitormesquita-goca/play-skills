# Plano de skills do play-skills

atualizado em: 2026-11-04

Este plano lista as skills que saem do contexto que o João já registrou e do que ele fez com o Claude, e define a ordem em que serão escritas.

## Fontes lidas

Todas foram lidas em 04/11/2026:
- **Vault "LLM Vault - Gogroup":** `Processos/` (14), `Eventos/` (3), `Contextos/` (17), `Projetos/` (5), `Decisões/` (29) e `Medições/` (19).
- **Repo `content`:** `CLAUDE.md`, `AGENTS.md`, as skills de mídia de `.claude/skills/` e as 28 memórias do projeto.
- **CLAUDE.md e AGENTS.md** do plaisson, ecom-metrics-hub (dashRoi), sheets e video-ads-sync.
- **Skills de mídia já sincronizadas do claude.ai:** `orcamento-campanhas-plano-do-dia`, `partnership-creators-admanage`, `varredura-promocao-encerrada`, `link-instagram-para-ad-id` e `text-to-sql`.
- **Sessões do Claude Code no repo `content`:** 133 sessões, de 26/08 a 04/11/2026. Muitas são forks da mesma conversa; sem contar os forks, sobram cerca de 70 trabalhos. Foram resumidas por script (pedidos, skills, ferramentas, arquivos) e lidas em 4 lotes.

Os docs do projeto "Vendas/Performance - Gocase" no claude.ai não foram lidos, porque não ficam na máquina. Pelo `AGENTS.md` do vault, eles são espelho do vault.

## O que as sessões mostraram

- **Subir anúncios é de longe o mais frequente.** "Pode lançar" aparece em cerca de 19 sessões só até 10/10, e há draft ou lançamento em cerca de 40 trabalhos.
- **Depois da subida, os mais frequentes são:**
  - `testes-aprovados`, em cerca de 20 sessões;
  - capacidade de páginas;
  - varredura de promoção;
  - cobertura e abastecimento do evento.
- **Quase toda subida foi pelo Chrome**, com centenas de chamadas de `javascript_tool` por sessão. A sessão 97e31a47 (27/10) fez tudo pelo MCP do AdManage (`upload_media_from_url`, `launch_draft`, `get_batch_status`). Isso prova que o caminho da Onda 1 funciona.
- **Apareceram rotinas que não tinham nota no vault:**
  - a marcação `[PLAY]` dos testes promovidos;
  - identificar o produto do anúncio;
  - encontrar criativos históricos;
  - trocar o link de anúncios já no ar;
  - montar campanhas em rascunho;
  - criar templates de copy de evento;
  - revisar fala e tela;
  - organizar as pastas de conteúdo.
- **Pedir "faz como na sessão X" foi frequente.** O Claude teve de reler transcrições para recuperar o método (1695d054, 2f5ebad4, b23b2719). É sinal de que o método ainda não tinha virado skill.
- **Já houve demanda de outra marca.** Em 09/10 (410c45e8), o João pediu um zip das skills genéricas de AdManage para quem toca outra marca.

## Como as skills se organizam

1. **Fundação.** São skills de referência que as outras citam pelo nome: regras do grupo, dashRoi, Plaisson e o perfil da marca. Assim a convenção não se repete em cada skill.
2. **Blocos.** Cada bloco faz uma etapa só (editar draft, aplicar parceria, achar campanha, identificar produto).
3. **Processos.** Orquestram os blocos de ponta a ponta (subir conteúdo humanizado, abastecer campanhas).

O que muda por marca fica no perfil (ver "O que entra no perfil da marca").

## Onda 0: fundação

| Skill | O que ensina | Fontes |
|---|---|---|
| `perfil-da-marca` ✅ | Cria e lê o perfil da marca | — |
| `regras-do-play` ✅ | Regras transversais (lista abaixo) | CLAUDE.md do content; processos Orçamento, Varredura, Abastecimento e Desligar CX; evento Black Month; correções recorrentes nas sessões |
| `dashroi` ✅ | Fuso (UTC → BRT, filtrar por intervalo e não por `::date`); IOF com data de vigência; não somar fontes duplicadas; peso de kit; join por nome normalizado; horas com `spend>0`; atraso por campanha. O dashRoi **não tem o dia corrente**: para hoje, usar a Meta. Campanha ou ad set **recém-criado** não aparece. Frescor do ETL. Os tipos de ROI e o Σ÷Σ. SQL de base com `brand_id` como parâmetro | Contextos do dashRoi e ROIs; `text-to-sql`; sessões ce826370, 606e27a4, 2dc1bcfb |
| `plaisson` ✅ | Agendamento × motores; como uma regra casa; cooldown; gates; pods; ABO; `get_logs` truncado; `fresh` com cota; `configSource=sheet`. Valores sempre ao vivo pelo `get_rules` | Contexto Plaisson e os motores; decisões do intra |

As regras transversais de `regras-do-play`:
- **Ordem explícita** para lançar, ativar, mudar orçamento, arquivar ou apagar. Pedidos como "não executa ainda", "só planeja" e "monta sem lançar" são respeitados.
- **Pausar, não apagar**, quando o objetivo é tirar do ar.
- **Antes de um lote:** panorama em tabela e uma única rodada com todas as dúvidas. Escalar só o que precisa do piloto ("só os que precisam de mim").
- **Mostrar o funil da contagem a cada filtro.** Por exemplo: 99 → 24 → 22. Nas sessões, os números que não batiam geraram cobrança 4 vezes.
- **Preview antes de confirmar**, e nunca confirmar com erro.
- **Conferir depois de agir:** `IN_PROCESS`, agendamentos `pending`, contagem final.
- **Status ao vivo antes de usar uma campanha:** o piloto pode ter desligado hoje.
- **Antes de lançar, listar a pasta de novo** (pode ter entrado arquivo) e **checar na Meta se o lote já está no ar.**
- **Respeitar a ordem dos lotes** que o piloto pediu.
- **Rascunho com nome definitivo**, porque renomear quebra a UTM.
- **Feed + story = um anúncio.** O par se confirma pela imagem, não só pelo nome.
- **Só condição comercial vigente:**
  - preço, promoção datada ou "só hoje" fora do vigente não sobe;
  - produto fora de linha ou fora do site não sobe;
  - sazonal vencido sai.
- **Mesmo criativo com vários nomes conta como um conteúdo.** Exemplos: `[PLAY]`, `— Cópia`, `_Sub_NN`, `(1)`, `V2`, versões com e sem headline.
- **Status sempre pela Meta**, e o dado histórico pelo dashRoi.
- **Mesmo corte horário** ao comparar dias.
- **Nunca fazer média de ROIs.**
- **Dado com data:** reler ao vivo.
- **Mudar orçamento pela Meta pausa a campanha.**
- **Uma variável por vez.**

## Onda 1: subir anúncios (AdManage pelo MCP, sem Chrome)

| Skill | O que faz | Reescrita e detalhes das sessões | Fontes |
|---|---|---|---|
| `produto-do-anuncio` (bloco) | Descobre o produto pelo nome, depois pela pauta e por último pela imagem, com o dicionário de sinônimos do perfil. O ambíguo vai numa rodada única. Monta a tabela Produto \| Qtd antes de lotes grandes | Nome → GoHits `gocreators_briefings` → AdManage `view_image` (no lugar das contact sheets locais). O "- produto X" do nome decide campanha e copy. O InfluHub sozinho não basta para influ | Sessões bea395c2, 863555cf, 97e31a47, 9a5acb5f, 6da7aa05, 1c9ae552; memória do InfluHub |
| `admanage-midia` | Põe a mídia no draft: pasta do Dropbox, URL ou anúncio existente; dedup .MOV/.mp4 e por hash; feed+story como `multi` | `browse_dropbox`, `import_dropbox_folder`, `upload_media_from_url`, `get_source_ad_setup`, `copy_meta_ads_cross_account`. Link de preview do Dropbox nunca. **Post ID desligado por padrão**; exceção: testimonial e hit campeão, com pedido. **Existing Ads instável**: escada de fallback (tentar de novo → lotes menores → por ad_id → mesmo nome pela integração do Dropbox). Limite de 10 MB por upload. Arquivo local: o piloto põe na pasta do Dropbox | `admanage-midia`; sessões 5090e626, 6433d8ab, 863555cf, 912ace78, 0a5ed2c4 |
| `admanage-draft` | Lê, edita e confere as linhas de um draft; cria um draft de backup antes de editar | `get_draft` → `update_draft_rows` → `get_draft` | `admanage-csv` |
| `admanage-parcerias` | Casa o @handle com a parceria (exato, normalizado, parecido) e confirma os ambíguos de uma vez. Também lista quem não aceitou collab, em bloco copiável | `list_partnership_permissions`, `update_draft_rows`. Fallback e exceções vêm do perfil (ex.: fallback, mas não em nome de homem; TTS e afiliados sem collab). Nunca enviar pedido de parceria | `admanage-partnerships`, `partnership-creators-admanage`; sessões bea395c2, 6e202a19, bd317928, 42b05b90 |
| `admanage-copy-e-link` | Copy por produto e link de destino. Template evergreen primeiro; **durante evento, os templates do evento**. Sinaliza preço fixo, promoção vencida e URL no texto diferente do link. Link: PDP pelo SKU ou slug, LP da linha em evento (com cupom se houver), home como fallback; mesma UTM dos lotes anteriores | `list_ad_copy_templates`, `get_ad_copy_template`. Catálogo pelo conector de dados da marca. Regra PDP × LP por produto no perfil | `admanage-copy-templates`; sessões 353c5944, 8b783c12, 4ea99a68, e5611118, 912ace78 |
| `campanhas-por-produto` | Uma campanha própria → ela. Várias → a humanizada ou a "mais necessitada" (pior ROI recente). Nenhuma → a mista ativa. Nenhuma ativa → pergunta, sem jogar numa campanha genérica. **O destino depende do tipo de conteúdo** (estúdio, foto, bait, humanizado, influ, Big Influs) | SQL pelo dashRoi; Plaisson `get_brand_dashboard`; campanha ou ad set novo por AdManage `list_adsets` ou Meta `ads_get_ad_entities` (aconteceu 5+ vezes). Ad set "cliente" × aquisição | `meta-campanhas-por-produto`; sessões 66c12f06, d8008ec2, 63f8dc0f, 6da7aa05 |
| `capacidade-de-paginas` | Escolhe uma página com vaga, divide o lote quando não cabe e exclui páginas que não veiculam no país | **Depende do spike S1.** A política de escolha vem do perfil | `meta-capacidade-paginas`; sessões 796430a4, 1695d054, 6e202a19 |
| `admanage-lancar` | Pré-voo, um único lançamento, acompanhamento e relançamento só do que falhou | `check_launch_conflicts`, `get_launch_context`, `launch_draft`, `get_batch_status`, `get_delivery_errors`. **Lotes grandes divididos** (815 travou). **Dois limites de ad set**: 150 da Meta (arquiva candidatos se o perfil permitir) e 51 do AdManage (pula). Ciclo "o piloto limpa → segue com o próximo ad set". Opção: anúncios ativos com a campanha pausada | `admanage-launch`; sessões a0bb1163, bea395c2, bd317928, 6da7aa05 |
| `subir-conteudo-humanizado` (processo) | Do link ao draft (e ao lançamento, se houver ordem), com variantes creator, TTS e influ. Influ separa Big Influs dos influs por produto | Orquestra os blocos acima. Tag do fluxo no nome antes do lançamento | `creators-play`, `tts-play`; sessões bd317928, 1c9ae552 |

## Onda 2: criativos e testes

| Skill | O que faz | Fontes |
|---|---|---|
| `testes-aprovados` | Contagem por campanha de teste, gasto rápido e link do Gerenciador com os aprovados selecionados. Pontos das sessões:<br>- a régua vem do perfil e pode mudar por formato (foto, estúdio, vídeo) e na hora;<br>- **janela de criação é parâmetro**;<br>- filtros por tag com exclusão; exclui `[PLAY]`, "não" e "não subir" no nome;<br>- **inclui os dados de hoje pela Meta**;<br>- panorama por produto (grandes em linha, o resto agregado, linhas de produto agrupadas, anúncio multiproduto em cada um);<br>- produto com 0 vem com o motivo;<br>- tabela antes do link | `testeatc-aprovados`, processo TESTEATC; sessões 318a97fb, 1695d054, 2f5ebad4, ce826370, 606e27a4 |
| `abastecimento` | Campanhas sem anúncio novo → vencedores por campanha → aprovação → subir ou duplicar → **marcar a origem como promovida** (`[PLAY]`). Na cobertura de evento, a métrica é "campanhas sem anúncio da tag", não "sem a tag no nome". Destino em cascata (produto com ROI abaixo da régua → não-evento de menor ROI → campanha do evento), round-robin, sem duplicar vídeo na mesma campanha, multiproduto em todos os destinos. Exclusões vêm do perfil (ASC de catálogo, Reconhecimento, Humanizada). Painel de cobertura opcional | Processo Abastecimento, decisões e medições GOBLACK; sessões 6433d8ab, 863555cf, 192a12c4, ce826370, 06965275 |
| `marcar-promovidos` (bloco) | Depois de subir um vencedor, renomeia o anúncio de origem na campanha de teste com o marcador do perfil, checando antes se ele já está em compra. A deduplicação seguinte usa essa marca | Meta `ads_update_entity` com `publish_as_active=false`; lote pelo spike S8. Sessões 6433d8ab, ce826370, 606e27a4, 0b6034f4, F21, ea167ede, bea395c2 |
| `renomear-em-massa` (bloco) | De-para → aprovação → renomear: tag de fluxo, correção de padrão de nome. Renomear não reinicia o aprendizado | Meta ou AdManage `update_name` (spike S8). Sessões 9ef08cad, d8008ec2, e64cc154 |
| `trocar-link-no-ar` | Lista os anúncios por link antigo, confirma a contagem e troca a URL. Filtro opcional por ROI | Meta. Sessões 1cc86ac3, 3a0d4637, 4b83778a |
| `criativos-historicos` | **Encontrar** os melhores criativos históricos por produto, campanha ou época (gasto e ROAS com gasto mínimo), ou os que citam o preço novo. Tira o sazonal e o que tem preço vencido e entrega a lista. Religar ou duplicar é um passo opcional, só com ordem: `ads_create_ad` nos ad sets ativos de cliente e de aquisição, com desempate pelo gasto histórico | Sessões 192a12c4, 6cff45a6, 68be3bf2, 0986c212 |
| `revisao-de-criativo` | Fala e tela contra a condição vigente (preço, "só hoje", cupom, promo datada, menção a canal). Devolve, caso a caso, se mantém, cobre, corta ou descarta. Regras: só música não é proibição; na dúvida, encontrar um jeito de usar; cupom de primeira compra pode, cupom comissionado não | **Depende do spike S6** (transcrição remota). A lista do `tts-limpar-meta` entra em `references/`. Sessões 192a12c4, 66c12f06, b340540b, 606e27a4, F14a |
| `varredura-promocao` | Mudou a promoção: nome → pauta → fala e tela, **amostra por conceito** (uma leva de estúdio do mesmo dia é um conceito). Pausa só o que tem o item na imagem ou no vídeo; menção só no texto do anúncio não pausa. Pausar a campanha inteira é uma opção | `varredura-promocao-encerrada`, processo Varredura; sessões 8c931d05, de575be9, fde11d34, 9ad854cf, b529cee8 |
| `achar-anuncio-pelo-link` | Link do Instagram → anúncio(s) → pausar, com ordem. Sem o `xpv_asset_id`, entrega os candidatos e pede confirmação | `link-instagram-para-ad-id`, processo Desligar CX |
| `tiktok-shop` | Hits por GMV e views por produto (destaque × top N), cruzamento com o que já roda na Meta e direito de uso; perguntas soltas (receita de lives × vídeos, hashtag) | Processo Reaproveitamento TTS; sessões F14a, 19ce59e3, b4811dd8. Só para marcas com TikTok Shop |
| `selecao-de-hits` | Faixas de gasto (creator), top GMV (TTS), "outras fontes", cotas por faturamento da linha, recorte por época. Só a parte de dados | Processo Seleção de hits; sessões F21b, a9d90f04 |
| `gasto-por-conceito` | Classifica pelo nome e pelo formato; ROAS da Meta só para comparar conceitos | Medição Gasto por formato e conceito |

## Onda 3: orçamento

| Skill | O que faz | Fontes |
|---|---|---|
| `agendar-orcamento` | Virada e boost no Plaisson: preview, um lote por conta, tirar pods e ABO, checar duplicidade, `alvo_gasto` e conferência | Processo Orçamento diário, contexto do Plaisson, medição Curva de gasto |
| `rateio-por-campanha` | Divide o plano entre campanhas: universo, classes, reservas, âncora, mérito com shrink e clamp, piso, conferência 99–101%. A saída vai para o Sheets, não para XLSX local | Processo Rateio, `orcamento-campanhas-plano-do-dia` |
| `leitura-intraday` | Subaproveitamento: frescor, mesmo corte, pacing contra a mira, gate de ROI, marginal, subatribuição | Processo Leitura intraday |
| `auditar-motores` | Subidas e cortes por dia, regras sombreadas, zona morta, ROI marginal × COS | Medições Linha de base do intra e Plano do Dia vs regra |
| `play-masters-travados` | Decide os retidos por tipo de oferta; diagnóstico de `no_match` (matcher de produto E categoria) | Medição Decisões humanas nos play masters, contexto Subida de vídeos do Dropbox |

## Onda 4: eventos e análises

| Skill | O que faz | Fontes |
|---|---|---|
| `campanhas-em-rascunho` | Parte de uma campanha modelo: troca o produto nos nomes da campanha e dos ad sets, tira os anúncios, põe o orçamento e entrega em rascunho. Quem publica é o piloto. Desliga os ad sets que concorrem com a promo | **Depende do spike S7.** Sessões e5848f12, 4b83778a, ebb67dec, 454cac72, 66c12f06 |
| `templates-de-evento` | Exporta os templates, confere status e título de cada link, procura nas LPs menção a mimo, preço ou evento, propõe LP ou PDP por produto e cria os templates com o prefixo do evento depois do ok. Também cria copy nova imitando uma aprovada | AdManage `create_ad_copy_template` e `update_ad_copy_template`; sessões 1cc86ac3, 3a0d4637, 4b83778a, 978eef4a, d5d00f3b, e91d19fd |
| `xadrez-promocional` | Mecânica, candidatos (receita, investimento, pares, venda sem mídia), faixas de preço, revisão pós-promo e operação da planilha do evento (check, cor, preço, listas) | Processo Xadrez; sessões dd4bb734, 06965275, e5848f12, ea167ede |
| `verba-de-live` | Verba em R$/hora; vitalício com fim programado; começar baixo e subir em degraus; gatilhos | Processo Rotina das lives, decisões das lives de 5h e 6h |
| `resultado-de-live` | As quatro leituras de receita e o que a divergência indica; hora a hora; custo fixo fora do ROI | Contexto Lives da YOOU, medições das lives |
| `comparar-evento` | Evento × evento análogo × linha de base; "produto melhorou" = venda no site subiu; D+1; viés da manhã | Medições 10.10 vs 8.8, Picos e regressão; decisão Série C |
| `ab-de-lp` | A/B por `utm_content` ou por cupom, com o dimensionamento do teste. O Goshorts não mede | Contexto Goshorts; decisão e medição da LP de primeira compra |
| `teto-de-cos` | Teto de COS e ROI mínimo a partir de LB, logística, adm e MC alvo | Processo Leitura da Margem Diária, medição Teto de COS |

Como referência das skills de orçamento, não como skill própria: Projeção de dia pós-pico e Plano do Dia (a lógica da mira, sem o Apps Script).

## Fora das ondas, para avaliar depois

- **`pastas-de-conteudo`:** cria e organiza a árvore de pastas no Dropbox por produto e tipo, com dry-run antes. Apareceu em 8 sessões (a9d90f04, f7b20857, 7a75924c, 7a368a69, entre outras). No MCP, `create_folder` funciona bem, mas `copy`/`move` são lentos (dezenas de `check_job_status`). O João prefere a pasta sincronizada. Vale só se o spike S9 mostrar um caminho aceitável.

## O que fica no repo `content`

- Rotinas só do João, decididas em 04/11/2026:
  - **Fila de entregas do MURAL:** mensagem sem ✅ → subir → reagir e responder na thread.
  - **"O que na pasta ainda não subiu"** e a limpeza do Dropbox depois do launch. Não é necessária no plugin.

- Geração e edição de criativo: piapp, cenas de capinha, overlays de time, Remotion, HyperFrames, Concat, beat-cuts, cards de live, template do copo vibe, Notícia/Adeus.
- Limpeza e render de TTS para a Meta (`tts-limpar-meta`). A lista de regras vai para `revisao-de-criativo`.
- Download de mídia (yt-dlp, gdown, Dropbox local) e transcrição local (whisper).
- `agendar-cupom-live` (Chrome no admin do Spree).
- O que é só da Gocase:
  - precificação de capinhas;
  - as LPs;
  - os IDs de promoção do Spree;
  - as decisões pontuais de cada evento;
  - o Plano do Dia em Apps Script.

## O que entra no perfil da marca

O `modelo-perfil.md` precisa ganhar estas seções:

| Seção | Exemplos da Gocase |
|---|---|
| Dicionário de produtos e sinônimos | Flip = Flip Pro; Mala de Bordo = Trip; Kids = Fun; Tote Puffer ≠ Tote Moon Puffer; produtos fora de linha |
| Padrões de nome por tipo de conteúdo | UGC `@handle - semana - … - produto X`, `PIAPP - {sku} - {data} - {conceito}`, `INFLU - produto - nome`, `{Produto} - [TTS] - {handle}__{id}`; `[MOTOR]` só do fluxo automático |
| Tags de fluxo e de evento | `[TOPGMV]`, `[MIMO-ORG]`, `[GOBLACK]`, `10DO10` |
| Marcador de "já promovido" | `[PLAY]` na campanha de teste |
| Destino por tipo de conteúdo | Estúdio, foto, bait, humanizado, influ, Big Influs |
| Exclusões de cobertura e de "já subido" | Campanhas de teste, ASC de catálogo, Reconhecimento |
| Regra de link por produto | PDP × LP, padrão de UTM |
| Árvore de pastas de conteúdo | TTS/Creator/<NN Produto> |
| Réguas | Teste aprovado por formato, corte, ROI de destino no abastecimento |

As preferências de operação também entram no perfil. São escolhas do piloto, não regras do grupo:

| Opção | O que o João usa hoje |
|---|---|
| Creator sem parceria | Parceria padrão (let.louise), **exceto em nome de homem**; TTS e afiliados sem collab |
| Escolha da página | O Claude escolhe a que tiver vaga e avisa (às vezes o João indica uma) |
| Ad set cheio | Desde 13/10, o Claude marca os candidatos (>20 d, sem conversão, menor gasto) e arquiva. Antes, o João apagava |
| Produto incerto | Copy próxima e link na home |
| Post ID | Desligado por padrão |
| Autonomia | Decidido um ponto, seguir sem checagem extra |
| Formato das respostas | Curto, "como se fosse eu escrevendo"; listas copiáveis em code block |

## Spikes antes de escrever

| # | Pergunta | Bloqueia |
|---|---|---|
| S1 | **Feito em 04/11/2026: não há caminho remoto.** Nenhum conector (Meta oficial, AdManage, Plaisson) expõe `ads_volume` nem contagem por página; o `get_held_destination_options` do Plaisson é sobre campanhas, não páginas. O AdManage `list_pages` só lista as páginas da conta (13 na Principal) com `promotionEligible`. A skill vira: (a) o piloto informa os números da tela `ad_limits` ou escolhe a página; (b) a regra do perfil escolhe por padrão; (c) se o launch falhar por limite (`get_delivery_errors`), trocar a página só das linhas que falharam e relançar | `capacidade-de-paginas` |
| S2 | **Feito em 04/11/2026: funciona.** Draft de teste criado, editado e apagado: `update_draft_rows` mudou título, texto, link, nome (`customName`) e página de 2 linhas numa chamada, e o resultado bateu. Custos: `get_draft` com `fields` (ex.: `state.rows.id`) é barato (88 linhas = só os IDs); sem `fields`, cada linha pesa ~5 KB (ad sets completos), então um draft de 88 linhas passa de 400 KB. **O `update_draft_rows` devolve o draft inteiro e não aceita `fields`**: em draft grande, mande todos os patches numa chamada só e confira depois com `get_draft` + `fields`. O nome efetivo do anúncio é `customName` (não `adName`). Parceria fica em `partnershipAd` e `dynamicData` da linha | `admanage-draft` |
| S3 | **Feito em 04/11/2026: funciona, depois de reconectar a Meta no AdManage** (Settings › Authentication › "Re-Authenticate Facebook" › "Refresh permissions" › "Save changes"; o aviso "Token expired" mostra quando precisa). Passe sempre o `workspaceId`: sem ele, o AdManage caiu num token antigo (`tokenSource: fallback`), deu erro 190 e devolveu a lista cortada em 500 (`partial: true`); com ele, veio completa (508 linhas: 420 Approved, que bate com o export; 54 Pending Approval, 32 Revoked, 2 Canceled). A resposta passa de 200 mil caracteres e não tem paginação nem filtro por handle: a skill lê tudo uma vez por lote e só usa `creator_username`, `creator_ig_id`, `creator_page_id` (vem em ~2/3) e `permission_status` = `Approved`. Se vier `fetchIssues` ou `partial: true`, a lista está incompleta e a skill avisa o piloto em vez de concluir que o creator não tem parceria. No claude.ai essa resposta entra inteira no contexto (~60 a 70 mil tokens): uma chamada por lote, nunca por creator; se o contexto estiver apertado ou a lista vier cortada, a alternativa é o piloto anexar o export CSV de parcerias do AdManage. O conector oficial da Meta (`ads_fb_partner_permission_manager`) não substitui: só lê a parceria do **Facebook** (a doc diz para não deduzir a do Instagram), um creator por chamada, pede `business_id` com `business_management`, e no teste de 04/11 recusou por falta de permissão do conector | `admanage-parcerias` |
| S4 | Quais conectores os pilotos têm: dashRoi, Supabase, Metabase, GoHits, Sheets, Google Chat? | Ondas 1 a 4 |
| S5 | **Respondido em parte em 04/11/2026:** a receita mora em tabelas por família (`site_sales` + Plausible/GA4 na Gocase; `beaute_sales` + `beaute_attribution_utm` nas gobeaute; `jump_sales` na Jump), e a camada comum é `vw_brand_product_daily` (ver skill `dashroi`). Falta: de onde vem o catálogo (PDP) das gobeaute | `dashroi`, `admanage-copy-e-link`, `comparar-evento`, `resultado-de-live` |
| S6 | O GoHits transcreve e lê os vídeos de influ e de afiliado? Quanto tempo leva a fila (nas sessões, ~35 min)? | `revisao-de-criativo`, `varredura-promocao`, variante influ |
| S7 | Criar campanha em rascunho por MCP: o `ads_create_campaign` cria publicada; testar o `duplicate_campaign` e o `duplicate_adset` do AdManage | `campanhas-em-rascunho` |
| S8 | Renomear em lote: o `update_name` do AdManage aceita vários anúncios? Na Meta é uma chamada por anúncio | `marcar-promovidos`, `renomear-em-massa` |
| S9 | Copiar e mover no Dropbox MCP em volume: tempo e um jeito de acompanhar o job | `pastas-de-conteudo` |

## Como escrever cada skill

1. Ler as fontes listadas na linha da skill. Para as sessões, os resumos ficam em `<scratchpad>/sessoes/out/<data>_<id8>.md`. São temporários: o que importar entra na skill.
2. Reescrever seguindo o `CLAUDE.md` deste repo: método e não número, nada da Gocase como regra, só conectores remotos.
3. Os números da Gocase entram só como exemplo, com data. O valor de verdade vem do perfil, do sistema ao vivo ou do piloto.
4. Testar numa tarefa real: uma vez com a Gocase e, quando der, uma vez com uma marca gobeaute.
5. Subir a versão e registrar no `CHANGELOG.md`.
