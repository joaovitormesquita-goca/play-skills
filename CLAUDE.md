# play-skills

Repositório do plugin de skills de mídia do Gogroup (Gocase e gobeaute). O dono é o João Vitor Mesquita, do Play. Responda sempre em PT-BR.

O porquê do projeto está no vault do João: `~/Documents/LLM Vault - Gogroup/Projetos/Gestão do Play Mobile.md` e `Decisões/2026-11-04 Gestão do Play Mobile vira plugin de skills.md`.

## Regras para escrever uma skill

1. **Método, não número.** Nunca fixe régua de marca (gasto mínimo, CTR, ROAS, dias de corte, limiar de motor). Diga onde ler o valor: o perfil da marca, o sistema ao vivo (ex.: `get_rules(slug)` no Plaisson) ou uma pergunta ao piloto. Número só entra como exemplo, com marca e data.
2. **Nada específico da Gocase como regra geral.** IDs de conta, página, campanha e ad set, nomes de campanha e handles ficam no perfil da marca. Se uma prática só vale para uma marca, ela vai no perfil, não na skill.
3. **Só conectores remotos.** A skill precisa rodar no claude.ai (inclusive no celular), no Cowork e no Claude Code de outra pessoa. Nada de navegador, script local, `tooling/`, Dropbox sincronizado ou caminho de arquivo da máquina do João. Se a tarefa precisar disso, ela fica no repo `content` do João.
4. **Cite a ferramenta pelo conector e pelo nome curto**, por exemplo "AdManage: `launch_draft`". O prefixo da ferramenta muda conforme a instalação.
5. **Ações irreversíveis ou que gastam dinheiro só com ordem explícita do piloto:** lançar, ativar, mudar orçamento, apagar, arquivar. Antes de um lote, mostre um panorama em tabela.
6. **Nada de segredo.** Nenhum token, senha ou chave de API. IDs de conta e página podem entrar no perfil da marca, não na skill.
7. **Frontmatter com `name` e `description`.** A description diz quando usar e quando não usar, com as frases que o piloto falaria.
8. **Uma skill por tarefa.** Modelos e material longo vão em `references/`.

## Origem das skills

Muitas skills nascem das skills e notas do João (repo `~/Desktop/Workspace/content/.claude/skills/` e o vault). Elas são da Gocase e dependem de navegador e de scripts. Não copie: reescreva a skill seguindo as regras acima.

## Versão

A cada mudança publicada, suba a versão em `.claude-plugin/plugin.json` (semver) e registre no `CHANGELOG.md`. Publicar na biblioteca da organização só com ordem do João.
