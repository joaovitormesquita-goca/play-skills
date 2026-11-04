---
name: plaisson
description: Como o Plaisson (automação de orçamento das campanhas Meta do Gogroup) funciona e como ler e mexer nele com segurança — agendamento (virada e boost) × motores (intra, descida, subidas, desligar/religar, pods), como uma regra casa, cooldown, gates de COS e de ROI de produto, avisos do preview, logs e as armadilhas que travam campanha em silêncio. Use quando o piloto perguntar "por que o Plaisson subiu/baixou essa campanha?", "o que o intra fez hoje?", "essa regra está pegando?", "agenda o orçamento", "o motor está ligado?", ou antes de qualquer mudança de regra, motor ou orçamento pelo conector do Plaisson. Para montar o plano de verba do dia, use as skills de orçamento.
---

# Plaisson

O Plaisson grava orçamento nas campanhas da Meta das marcas do Gogroup. Ele faz duas coisas diferentes:

- **Agendamento:** grava os valores que o piloto decidiu (ex.: virada à noite, boost de manhã). No log aparece com `source = "agendado"`.
- **Motores:** robôs com regras próprias que mexem no orçamento ao longo do dia. No log aparecem com `source = "robot"`. Mudança feita por pessoa aparece como `human`.

Cada marca (às vezes cada conta) tem um **slug** no Plaisson, com regras independentes. O slug vem do perfil da marca ou de `list_brands`.

> **Esta skill descreve o mecanismo, não os valores.** Limiares, faixas, multiplicadores, horários e travas mudam e se leem sempre ao vivo: `get_rules(slug)` para as regras e travas; `get_brand_dashboard(slug)` para os motores ligados (`toggles`) e o campo `automated` de cada campanha. Nunca responda de memória.

## 1. Ler antes de concluir

| Pergunta | Ferramenta |
|---|---|
| Quais marcas e slugs | `list_brands` |
| Campanhas, orçamento, produto (`mista` = multiproduto), `automated`, motores ligados, gasto e receita do dia | `get_brand_dashboard(slug)`. A saída é grande: resuma. `fresh: true` lê ao vivo na Meta, com cota (ex.: em 2026, 1 leitura/min e 60/dia por marca) |
| Regras, travas por produto, multiplicadores, cooldown, gate de COS | `get_rules(slug)` |
| Quem mexeu em quê, por qual motor e com qual ROI | `get_logs(slug, range)` com `hoje`, `ontem`, `7d`, `30d` |
| Lotes agendados e resultado por item | `list_budget_schedules`, `get_budget_schedule(id)` |
| Play masters retidos para aprovação | `get_held_play_masters` |

Antes de explicar o comportamento de uma campanha, confira se ela está `automated` e quais motores estão ligados na marca. Motor ligado só em algumas campanhas é comum.

## 2. Os motores

| Motor | O que faz |
|---|---|
| `intra` | Sobe e desce orçamento conforme a performance, de hora em hora numa janela do dia |
| `descida` | Só cortes, numa janela da noite. Usa o mesmo conjunto de regras do `intra`, filtrando `Ação = Baixar` |
| `subidas` | Subida de uma passada, uma vez por dia |
| `desligar` / `religar` | Pausa e despausa campanha por performance |
| pods | Governam orçamento fora do rateio e podem reverter uma mudança no tick seguinte |

Os horários de cada motor estão na configuração da marca.

- **`intra` e `descida` compartilham as regras.** Mexer numa regra de corte muda as duas janelas.
- **Cada motor tem seu gate por produto** (ex.: `produtos` para o `intra`, `produtos_subidas` para o `subidas`). Ler o gate do motor errado leva a diagnóstico errado.
- **O `intra` escolhe a campanha pelo ROI de atribuição e pede licença ao ROI de produto** (ver skill `dashroi`).

## 3. Como uma regra casa

Uma linha do conjunto de regras dispara quando **todas** as condições valem, com limites **inclusivos**:

```
ação ∈ {Subir, Baixar}
piso         ≤ orçamento atual da campanha ≤ teto
roiHoraPiso  ≤ ROI(h−1) ≤ roiHoraTeto
roiHoraPiso  ≤ ROI(h−2) ≤ roiHoraTeto
dayRoiMin    ≤ ROI acumulado do dia ≤ dayRoiMax
```

- **As duas últimas horas cheias precisam passar.** Exigir um ROI nas duas aperta bem mais do que exigir numa hora só.
- **ROI acumulado do dia** = 00h até a última hora cheia, Σ receita ÷ Σ gasto.
- **Célula vazia = sem limite.** **`Valor` vazio vira 0**: a regra dispara, consome o match e não muda nada. É a forma mais silenciosa de travar uma campanha.
- **`Ação = Desligar` não roda no `intra`**, mesmo aparecendo no dropdown.
- **Na aba de descida, colunas de piso e teto podem ser de gasto do dia**, não de orçamento. Confira o cabeçalho.
- **First-match-wins:** as regras são avaliadas por `Número` crescente; a primeira que passa vence. Regra sobreposta com número maior fica morta.

**Como o valor vira orçamento:** `efetivo = Valor × multiplicador da janela horária`. Subir: `orçamento × (1 + efetivo)`. Baixar: `orçamento × (1 − efetivo)`. É percentual sobre o orçamento atual, arredondado para baixo, e depois passa pelos limites fixos do app. Na tela o `Valor` aparece em %; no banco é fração (`0.6` = 60%).

## 4. O que decide sem estar nas regras

- **Cooldown:** se a campanha mudou de orçamento nas últimas N horas, **por robô ou por pessoa**, a decisão do motor é descartada. Um boost agendado zera o relógio das campanhas que ele tocou.
- **Gate de COS:** só bloqueia `Subir`, a partir de uma hora configurada. COS desconhecido bloqueia (fail-closed).
- **Gate de ROI de produto:** sem regra cadastrada para o produto, ou sem ROI disponível, **não bloqueia**. Campanha `mista` costuma passar sempre.
- Reprovar num gate pula para a próxima regra candidata.
- Campanha abaixo do piso da menor regra de `Baixar` não é cortável; só sobe.
- **Marca com regras vindas de planilha** (`configSource = 'sheet'`): um sync periódico reescreve tudo, e o que se grava pelo conector some. Confira antes de editar regra.

## 5. Agendamento: o essencial

- `schedule_budget_batch` com `slug`, data, hora e as linhas `id<TAB>valor`. Um lote por conta.
- **Sempre rode sem `confirm` primeiro.** O preview traz orçamento atual, delta e avisos. Só confirme com `temErro: false` e com ordem do piloto.
- Antes de confirmar, `list_budget_schedules` para não duplicar o lote. Depois, confira que os itens ficaram `pending`.
- **Modos:** `orcamento` grava o valor como orçamento; `alvo_gasto` trata o valor como meta de gasto do dia e calcula o orçamento pela curva de gasto, só numa janela do dia (o orçamento gravado sai acima do alvo, e isso é o método funcionando). `alvo_gasto` usa gasto em cache, a menos que se passe `fresh: true` (repita na confirmação).
- **Avisos do preview:**
  - **Campanha ABO** (orçamento indeterminado): derruba o lote inteiro. Tire do lote e faça nos ad sets.
  - **Campanha governada por pod:** o pod reverte no tick seguinte. Tire do lote.
  - **Resolve como `adset`:** normal, não derruba o lote.
  - **Orçamento vitalício:** o Plaisson não agenda; o item falha.
  - Item perto do `religar` da noite: confira de manhã se o valor ficou de pé.

O passo a passo de virada e boost fica nas skills de orçamento.

## 6. Logs

- `get_logs` **trunca em ~2.000 registros.** Para estudo de vários dias, peça janelas menores.
- O log traz `engine`, `source` e o ROI que justificou a mudança. Para "por que subiu?", leia a linha da mudança e confira a regra que casou.
- Entre duas leituras na mesma noite, o orçamento pode cair sozinho: é o `descida` ou um pod.

## 7. Mudar configuração

Ligar ou desligar motor (`set_ads_engine_toggle`), mudar campo de regra (`set_rule_field`), limiar (`set_brand_threshold`), multiplicador (`set_multiplier`) ou papel de campanha (`set_campaign_*`):
- **só com ordem explícita** de quem é dono da decisão naquela marca (anote no perfil);
- mostre o antes e o depois e o efeito esperado;
- depois de mexer em faixas, procure regras sobrepostas (first-match-wins);
- registre a data e o motivo da mudança, para medir depois.
