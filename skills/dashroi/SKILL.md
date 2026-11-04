---
name: dashroi
description: Como ler gasto, receita e ROI das marcas do Gogroup no dashRoi (banco de mídia e ROI do grupo) sem cair nas armadilhas — fuso, IOF, marca errada, dupla contagem, kits, dia corrente incompleto, campanha nova que ainda não aparece — e qual ROI responde qual pergunta (atribuição, produto, mira, marginal, ROAS da Meta). Use sempre que for consultar ou interpretar números de mídia de uma marca ("qual o ROI do produto X", "quanto gastou ontem", "ROI da campanha", "isso está pagando?", "por que o ROI caiu?"), escrever SQL no dashRoi, ou quando outra skill do play-skills mandar ler dados no dashRoi.
---

# dashRoi

O dashRoi é o banco de mídia e ROI do Gogroup (Supabase). Tem gasto da Meta e do Google por campanha, ad set, anúncio e hora, receita do site e receita atribuída, para várias marcas.

## 1. Qual caminho usar

Nesta ordem, conforme o que o piloto tem ligado:

1. **Conector do dashRoi** (ferramentas de alto nível, já tratam fuso, IOF e marca):

   | Pergunta | Ferramenta |
   |---|---|
   | Quais marcas existem e qual o slug | `list_brands` |
   | ROI e gasto por produto num período (série diária opcional) | `get_product_roi` (`granularity: "day"`) |
   | Hoje hora a hora contra ontem, 7 dias ou uma data | `get_intraday` |
   | Anúncios em teste pareados com o desempenho fora de teste | `get_creatives` |
   | Quanto do gasto e da receita está mapeado a produto | `get_unmapped_spend` |
   | Se a coleta de cada fonte está em dia | `get_etl_health` |

2. **SQL direto** (Supabase `execute_sql`), quando a pergunta não cabe nas ferramentas. Use os trechos de `references/sql-base.md`.
3. **Nenhum dos dois:** diga ao piloto que falta o conector e pergunte se ele tem o número de outra fonte.

O slug da marca vem do perfil da marca ou de `list_brands`. Em SQL, a marca é um `brand_id`: ache pelo slug (ver `references/sql-base.md`), nunca pelo final de um ID de conta ou anúncio.

## 2. Armadilhas

1. **Sempre filtre pela marca.** As tabelas de gasto têm várias marcas juntas. Sem o filtro, gasto e COS inflam.
2. **`hour_start` está em UTC.** Converta para `America/Sao_Paulo`. Para filtrar um dia, use o intervalo UTC equivalente; converter dentro do `WHERE` (`::date`) impede o índice e a consulta estoura o tempo.
3. **IOF.** As tabelas de gasto da Meta (`facebook_data`, `facebook_ad_spend`) guardam gasto **sem** IOF. As views de ROI e o `get_product_roi` já aplicam. Use **com** IOF para ROI de produto e plano; **sem** IOF para ROI de atribuição e para orçamento na Meta. Exemplo: em 2026 o fator foi 1,1383 para datas a partir de 01/01/2026 — confirme se mudou. No Google não se aplica.
4. **Não some fontes que medem a mesma coisa.** `facebook_data` + `facebook_ad_spend` para a mesma campanha é dupla contagem. GA4 + receita de atribuição de outra ferramenta (ex.: Plausible) também.
5. **Kits.** Um anúncio pode estar ligado a 2 ou 3 produtos. Divida o gasto do anúncio pelo número de produtos dele, senão o gasto duplica.
6. **Receita atribuída se junta pelo nome da campanha normalizado** (minúsculas, espaços colapsados) **ou pelo ID**: às vezes a UTM grava o ID da campanha no lugar do nome. Use as duas formas. Campanhas homônimas dividem a receita errado: atribua à que teve gasto.
7. **Campanha de live** pode gravar o nome no `adset_name` e deixar `campaign_name` vazio na receita atribuída.
8. **Dia corrente.** O dashRoi atrasa, às vezes por campanha (uma campanha pode estar horas atrás das outras), e a receita do GA4 pode estar vazia hoje. Para hoje, confira o frescor (`get_etl_health` ou o trecho de frescor) e use a Meta para o status e o gasto do dia.
9. **Campanha ou ad set recém-criado não aparece** até a próxima coleta. Busque o ID no AdManage (`list_adsets`) ou na Meta (`ads_get_ad_entities`).
10. **Data de criação do anúncio** (`dim_meta_ad.meta_created_time`) guarda só o dia, como meia-noite BRT. Para a hora exata, use a Meta.
11. **Horas de veiculação:** conte só horas com `spend > 0`.
12. **Conta pelo `account_id`**, nunca pelo prefixo ou sufixo do ID da campanha.
13. **A receita mora em tabelas diferentes por família de marca** (`brands.family`). O gasto da Meta é comum a todas; a receita do site e a atribuída, não:

    | Família | Receita do site | Receita atribuída |
    |---|---|---|
    | `gocase` | `site_sales` (sem `brand_id`) | `plausible_revenue`, `ga4_revenue` (sem `brand_id`, por nome de campanha) |
    | `gobeaute` | `beaute_sales` / `vw_beaute_sales_effective` | `beaute_attribution_utm` / `v_beaute_utm_campanha` (por `utm_campaign`) |
    | `jump` | `jump_sales` (diária, por loja e moeda) | — |

    Para comparar marcas, use as camadas comuns: `vw_brand_product_daily` (receita por produto, com moeda da marca) e as views de ROI, ou o `get_product_roi`. Marcas `jump` vendem em outras moedas: confira a moeda antes de somar. (Tabelas conferidas em 04/11/2026.)

## 3. Qual ROI responde qual pergunta

| ROI | Fórmula | Responde | IOF |
|---|---|---|---|
| **De atribuição** (de campanha) | receita atribuída à campanha ÷ gasto da campanha | A mídia está **comprando** essa receita? | sem |
| **De produto** | receita do site do produto (todos os canais) ÷ gasto de mídia mapeado nele | Este produto está **pagando** a mídia dele? | com |
| **Mira** | definido no planejamento do dia da marca | Qual é o alvo de hoje? | com |
| **Marginal** | Δreceita ÷ Δgasto numa janela, contra a mesma janela de outro dia | O dinheiro novo se pagou? | — |
| **ROAS da Meta** | `purchase_value ÷ spend` (pixel) | Atribuição da própria Meta, a mais conservadora; bom para comparar criativos entre si | — |

- O ROI de produto é sempre mais alto, porque inclui receita orgânica, de marca e de CRM. Não é defeito: é outra pergunta.
- **A divergência é informação.** ROI de produto alto com ROI de atribuição baixo = o produto vende por outros canais e a mídia pega carona; subir verba ali não compra a receita que o ROI de produto sugere.
- **Regra de decisão:** o ROI de produto aponta o candidato; o ROI de atribuição autoriza ou veta.
- **A fonte da atribuição muda por família** (armadilha 13): Plausible e GA4 na Gocase, UTM nas gobeaute. Em texto para o grupo, diga "ROI de atribuição", não o nome da ferramenta.
- Para comparar janelas de níveis diferentes (dia parcial × dia fechado), pondere **fatores** (ROI do item ÷ ROI de referência na mesma janela), nunca ROIs brutos.
- Receita atribuída zero com gasto real pode ser subatribuição de um dia: olhe 7 dias antes de condenar.

## 4. Ao responder

- Diga a janela (datas BRT), a marca, qual ROI e se o gasto está com ou sem IOF.
- Se o dia corrente entrou, diga até que hora os dados estão.
- Não traga o dado bruto inteiro: resuma e mostre só o que importa para a decisão.
