# _meta — Metadata do Projeto (não é conteúdo do vault)

Este folder contém documentação OPERACIONAL do projeto Agente Pessoal Obsidian. Diferente de `00 - Dashboard/` ... `10 - Projetos/` (que são o conteúdo do planner), `_meta/` documenta COMO o vault é construído.

## Arquivos

| Arquivo | O que é |
|---|---|
| `README.md` | Este índice |
| `plugins.md` | Registro dos 10 plugins community + versão + status manutenção + fallback se quebrar |
| `setup.md` | Guia passo-a-passo de setup (Windows Registry, GitHub, instalação de plugins) |
| `backup.md` | Estratégia de backup (obsidian-git → GitHub privado, intervalo 30min, anti-sync-corruption) |

## Arquivos futuros (Phase 2+)

| Arquivo | Phase | O que é |
|---|---|---|
| `schema.md` | Phase 2 | Schema frontmatter congelado (snake_case PT-BR) |
| `rubric.md` | Phase 2 | Âncoras concretas 1-5 por pilar |
| `fields.md` | Phase 2 | Justificativa de cada campo no template |
| `calibration.md` | Phase 4 | Histórico mediana tempo_real por tipo de tarefa (coach feedback loop) |
| `migrations/` | Phase 2+ | Templater scripts de migração se schema mudar |

## Regra

`_meta/` NÃO contém reflexão pessoal. Só documentação operacional. Tudo aqui é versionado no git (diferente de `_private/` que fica fora do backup).
