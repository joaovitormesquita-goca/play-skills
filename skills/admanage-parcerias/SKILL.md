---
name: admanage-parcerias
description: Liga anúncio de parceria (branded content) por creator nas linhas de um draft do AdManage, casando o @handle do nome do anúncio com a lista de parcerias aprovadas do Instagram da marca, confirmando os casos parecidos numa rodada só e aplicando o padrão do perfil para quem não tem parceria. Use quando um lote de creators, influs ou afiliados precisar de parceria ("liga as parcerias", "casa os creators com as parcerias", "quem não tem collab?", "configura partnership"). Não use para pedir parceria nova a um creator: isso é o piloto que faz.
---

# Parcerias por creator

Leia antes o perfil da marca (Instagram da marca, parceria padrão, exceções, handles confirmados) e siga a skill `regras-do-play`.

**Nunca envie pedido de parceria.** Isso fala com o creator em nome da marca e não tem volta. Quem não tem parceria vai para o piloto numa lista (§5).

## 1. Ler a lista de parcerias (uma vez por lote)

AdManage: `list_partnership_permissions` com `instagramId` = Instagram da marca e **`workspaceId` do perfil**.

- **Sempre com `workspaceId`.** Sem ele, o AdManage pode pegar um token antigo e devolver a lista cortada. Exemplo da Gocase (04/11/2026): sem `workspaceId` voltaram 500 linhas com erro; com ele, 508 completas.
- **Confira a resposta:** se vier `fetchIssues` ou `meta.partial: true`, a lista está incompleta. Não conclua que um creator está sem parceria; avise o piloto.
- **Erro de token** (código 190, "session has been invalidated", ou "Token expired"): a conexão da Meta no AdManage venceu. O piloto reconecta em AdManage › Settings › Authentication › "Re-Authenticate Facebook", depois "Refresh permissions" e "Save changes".
- **A resposta é grande** (centenas de linhas, cada uma com URL repetida) e não tem filtro por handle. Chame **uma vez por lote**, nunca por creator. Se o contexto estiver apertado, peça ao piloto o export CSV de parcerias do AdManage (tela de Partnership ads › Export to CSV) e use o arquivo.
- Use só `permission_status = Approved`. Campos que importam: `creator_username`, `creator_ig_id` e `creator_page_id` (pode vir vazio, e tudo bem).

## 2. Casar o handle

O handle sai do nome do anúncio, na posição que o padrão de nome do perfil indica.

1. **Handles confirmados do perfil** primeiro (o @ no nome às vezes não é o do Instagram).
2. **Exato:** igual, ou igual depois de normalizar (minúsculas, sem ponto, sublinhado e acento).
3. **Parecido:** um contido no outro com 6 letras ou mais, ou grafia muito próxima. Traga os 3 melhores candidatos.
4. **Nenhum.**

**Todo parecido e todo "nenhum" vão numa única rodada de perguntas antes de aplicar.** Errar põe o rosto de outra pessoa num anúncio pago. Vizinhos enganam: nomes que diferem por uma letra costumam ser pessoas diferentes.

O que o piloto confirmar, sugira registrar em "Handles confirmados" do perfil.

## 3. Quem não tem parceria

Siga o perfil:
- **parceria padrão** (um creator ou perfil da marca que aceita todos), quando houver;
- **exceções**, por exemplo tipos de conteúdo que sobem sem parceria ou casos em que o padrão não combina com o rosto do vídeo;
- sem regra no perfil: pergunte na mesma rodada do §2.

## 4. Aplicar

Em cada linha com parceria, o `partnershipAd`:

| Campo | Valor |
|---|---|
| `enabled` | `true` |
| `attributionMode` | o do perfil (`DYNAMIC` se não houver) |
| `creatorId` | `creator_ig_id` (numérico; nunca o @) |
| `creatorPageId` | `creator_page_id`, se houver |
| `brandPageId` / `brandInstaId` | a página e o Instagram da marca **daquela linha** |
| `swapOrder` | `false`, salvo pedido do piloto |

Ao criar o draft, vai em `ads[].partnershipAd` (skill `admanage-draft`). Num draft que já existe, leia uma linha com parceria para ver o formato do campo e aplique com `update_draft_rows`, todas as linhas numa chamada.

Depois, confira linha a linha: handle → parceiro (ou "sem") e modo. Peça ao piloto para olhar o **Preview** de um anúncio no AdManage antes de lançar.

## 5. Entrega

- Tabela: handle → parceria (exata, normalizada, confirmada, padrão, sem) → nº de anúncios.
- Em bloco de código, os handles **sem parceria**, para o piloto pedir a parceria pelo Instagram ou pelo Business Suite se quiser.

## Erros conhecidos no lançamento

- **"PAGE AND INSTAGRAM ACCOUNT DON'T MATCH"** numa linha de parceria: o problema está na conta do creator (página do Facebook ligada a outro Instagram). Relançar não resolve. Pergunte se a linha sobe com a parceria padrão ou sem parceria.
