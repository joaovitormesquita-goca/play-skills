# play-skills

Plugin de skills com as práticas de mídia do Gogroup (Gocase e marcas gobeaute). Com ele, o piloto de cada marca faz tarefas com o Claude (subir anúncios, mexer em orçamento, ler testes) já sabendo como fazer, quais regras seguir e quais práticas o grupo usa.

O plugin não traz ferramentas novas. Ele usa os conectores que já existem: Meta Ads, AdManage e Plaisson.

## Como funciona

| Camada | Quem mexe | O que guarda |
|---|---|---|
| Este plugin | João (os pilotos propõem) | Método, regras de segurança, como usar cada conector |
| Perfil da marca | O piloto da marca | Contas, páginas, slug no Plaisson, padrão de nomes, réguas da marca |
| Skills pessoais | Cada piloto, na própria conta | Atalhos e fluxos dele, sem afetar ninguém |

As skills não fixam métricas. Réguas como gasto mínimo, CTR, ROAS e dias de corte mudam de marca para marca. A skill descreve o método e diz onde ler o valor da marca: no perfil, ao vivo no sistema ou perguntando ao piloto.

## Começar

1. Instale o plugin pela biblioteca da organização no claude.ai.
2. Ligue os conectores que você usa: Meta Ads, AdManage e Plaisson.
3. Peça ao Claude "monta o perfil da minha marca". A skill `perfil-da-marca` lê o que der nos conectores e pergunta o resto.
4. Guarde o perfil onde você trabalha:
   - **Projeto no claude.ai ou no Cowork:** cole o arquivo no conhecimento do projeto da marca.
   - **Pasta local (Claude Code ou Cowork):** deixe o `perfil-marca-<slug>.md` na pasta.

## Skills

| Skill | Para que serve |
|---|---|
| `perfil-da-marca` | Cria, atualiza ou lê o perfil da marca que as outras skills usam |
| `regras-do-play` | Regras de operação que valem para todas as marcas: o que pede ordem, panorama antes de lote, conferir depois, o que nunca sobe |
| `dashroi` | Como ler gasto, receita e ROI no dashRoi sem cair nas armadilhas, e qual ROI responde qual pergunta |
| `plaisson` | Como o Plaisson funciona (agendamento, motores, regras, gates) e como ler e mexer nele com segurança |

O plano das próximas skills está em [docs/plano-de-skills.md](docs/plano-de-skills.md).

## Propor mudança

Abra uma issue neste repositório dizendo o que mudar, em qual skill e por quê, com um exemplo real. O João implementa, publica uma versão nova na biblioteca da organização e registra a mudança no `CHANGELOG.md`.

O que vale só para a sua marca vai no perfil da marca, não no plugin.

## Estrutura

```
.claude-plugin/plugin.json      nome e versão do plugin
skills/<nome>/SKILL.md          uma pasta por skill
skills/<nome>/references/       modelos e material de apoio da skill
CHANGELOG.md                    o que mudou em cada versão
CLAUDE.md                       regras para escrever skills neste repo
```
