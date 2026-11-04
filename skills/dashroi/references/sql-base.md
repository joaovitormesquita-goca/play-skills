# Trechos de SQL do dashRoi

Banco: dashRoi (Supabase). As colunas foram conferidas no `information_schema` em 04/11/2026. Os trechos foram validados rodando na Gocase em 10/10/2026 e 26/10/2026. Em outra família de marca, rode antes e confira se há dado.

Use os parâmetros num CTE no topo, para a consulta rodar como está.

## Marcas

`brands(id, slug, display_name, is_active, family, …)`. Em 04/11/2026, as famílias eram:
- `gocase`: gocase.
- `gobeaute`: rituaria, kokeshi, apice, barbours, lescent, yenzah, denavita, aua, by_samia.
- `jump`: weareknitter, pottd, funem, lifeofcolour.

## Tabelas comuns a todas as marcas

| Tabela | O que tem | Cuidado |
|---|---|---|
| `facebook_data` | Gasto, cliques, alcance e impressões por campanha/ad set e hora; `account_id`, `brand_id` | Sem nome de campanha. Gasto sem IOF |
| `facebook_ad_spend` | Gasto por anúncio e hora: `spend`, `impressions`, `clicks`, `add_to_cart`, `purchases`, `purchase_value`, `currency`, `brand_id` | Nunca some com `facebook_data` |
| `dim_meta_campaign` | `campaign_id`, `campaign_name`, `account_id`, `brand_id` | Fonte do nome da campanha |
| `dim_meta_ad` | `ad_name`, `norm_name`, `adset_id`, `campaign_id`, `meta_created_time`, `brand_id` | Data de criação só com o dia |
| `campaign_products` | Campanha → produto, `is_mixed` | Live costuma ficar sem produto |
| `ad_products` | Anúncio → produto | Kits têm 2 ou 3 linhas |
| `products` | Produtos, `category`, `active`, `brand_id` | `product_type_code` não é chave única |
| `vw_brand_product_daily` | Receita e unidades por produto e dia, `is_mapped`, `revenue_in_brand_currency` | Camada comum de receita |
| `vw_roi_daily*`, `mv_roi_daily` | ROI diário por produto/campanha | Já com IOF. O dia corrente tem a hora parcial |

## Receita por família

| Família | Tabela | Colunas principais | Cuidado |
|---|---|---|---|
| gocase | `site_sales` | `hour_start`, `case_type`, `device`, `revenue`, `units`, `customer_type` | **Sem `brand_id`**: é só da Gocase. Produto via `product_variant_map` |
| gocase | `plausible_revenue`, `ga4_revenue` | `hour_start`, `campaign_name`, `adset_name`, `revenue` | Sem `brand_id` e sem `campaign_id` |
| gobeaute | `beaute_sales`, `vw_beaute_sales_effective` | `brand_id`, `hour_start`, `source_product_id`, `source_product_name`, `revenue`, `units` | Prefira a view `effective` |
| gobeaute | `beaute_attribution_utm`, `v_beaute_utm_campanha` | `brand_id`, `hour_start`, `utm_campaign`, `utm_term`, `revenue`, `transactions`, `campanha_norm` | Junta com a campanha por `campanha_norm` |
| jump | `jump_sales` | `brand_id`, `date` (diária), `storefront`, `currency`, `revenue`, `units`, `currency_divergent` | Moeda varia por loja |

## Parâmetros: marca e dia

```sql
with p as (
  select b.id as brand_id, date '2026-10-09' as dia
  from brands b
  where b.slug = 'gocase'          -- troque pelo slug da marca
),
janela as (
  select brand_id,
         (dia::timestamp at time zone 'America/Sao_Paulo')       as ini_utc,
         ((dia + 1)::timestamp at time zone 'America/Sao_Paulo') as fim_utc
  from p
)
-- uso: where hour_start >= janela.ini_utc and hour_start < janela.fim_utc
--        and brand_id = janela.brand_id
```

## Nome de campanha normalizado

Para juntar gasto com receita atribuída por nome:

```sql
lower(btrim(regexp_replace(campaign_name, '[[:space:]]+', ' ', 'g')))
```

Na Gocase, junte também por `campaign_id::text`, porque às vezes a UTM grava o ID no lugar do nome. Nas gobeaute, a view `v_beaute_utm_campanha` já traz `campanha_norm`.

## Anúncio → produto sem duplicar gasto de kit

```sql
ap as (
  select ad_id, product_id,
         1.0 / count(*) over (partition by ad_id) as peso
  from ad_products
  where brand_id = (select brand_id from p)
)
-- uso: gasto do anúncio × ap.peso
```

## Gasto por anúncio num dia, com nome

```sql
select a.ad_id, d.ad_name,
       sum(a.spend)                                   as gasto_sem_iof,
       sum(a.clicks)                                  as cliques,
       sum(a.purchase_value) / nullif(sum(a.spend),0) as roas_meta,
       count(*) filter (where a.spend > 0)            as horas_com_gasto
from facebook_ad_spend a
join janela j on a.hour_start >= j.ini_utc and a.hour_start < j.fim_utc
left join dim_meta_ad d on d.ad_id = a.ad_id
where a.brand_id = j.brand_id
group by 1, 2
order by gasto_sem_iof desc;
```

## Frescor do gasto por campanha

Mostra a última hora com dado de cada campanha (BRT). Uma campanha muito atrás das outras está atrasada na coleta.

```sql
select a.campaign_id, c.campaign_name,
       max(a.hour_start at time zone 'America/Sao_Paulo') as ultima_hora_brt
from facebook_ad_spend a
left join dim_meta_campaign c on c.campaign_id = a.campaign_id
where a.brand_id = (select id from brands where slug = 'gocase')
  and a.hour_start >= now() - interval '2 days'
group by 1, 2
order by ultima_hora_brt;
```

Pelo conector, `get_etl_health` dá a saúde por fonte, sem SQL.
