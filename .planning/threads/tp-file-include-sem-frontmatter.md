---
slug: tp-file-include-sem-frontmatter
title: Templater tp.file.include() requer arquivo sem frontmatter
status: resolved
created: 2026-04-25
updated: 2026-04-25
---

# Thread: tp.file.include() requer arquivo SEM frontmatter

## Goal

Documentar pattern: arquivos referenciados via `tp.file.include("[[X]]")` em templates Templater devem ser SEM frontmatter (apenas conteúdo). Senão o include injeta o YAML como texto no corpo do arquivo destino, quebrando a parse do frontmatter.

## Context

*Descoberto em 2026-04-25 durante Fase 2-UX (rotina editável inline).*

**Sintoma:**
Ao usar `<% tp.file.include("[[Rotina-Dia-Util]]") %>` no template diário, com a rotina contendo seu próprio frontmatter (`tipo: rotina`, `nome: "Dia Útil"`, etc.), a daily resultante ficava com 2 frontmatters seguidos:
```yaml
---
data: 2026-04-25
[campos do daily]
---

# título do daily

---
tipo: rotina           # ← frontmatter da rotina, INJETADO COMO TEXTO
nome: "Dia Útil"
---

# 🗓️ Rotina — Dia Útil
[conteúdo]
```

Resultado: parser YAML do Obsidian quebra; Dataview não consegue ler frontmatter; Properties sidebar reclama.

**Solução aplicada:**
Removido frontmatter de `Rotina-Dia-Util.md` e `Rotina-Fim-Semana.md`. Arquivos passam a ser apenas markdown puro (header H1 + tabelas).

**Trade-off aceito:**
Perde-se metadados estruturados das rotinas (`tipo`, `nome`, `aplica_dias`) — mas Dataview no `_INDEX.md` agora usa `file.name` em vez de filtrar por `tipo`. Aceito porque rotinas são "dados visuais", não queryable.

## References

- Arquivos afetados: `05 - Rotinas/Rotina-Dia-Util.md`, `05 - Rotinas/Rotina-Fim-Semana.md`
- Templater docs: https://silentvoid13.github.io/Templater/internal-functions/internal-modules/file-module.html#tp-file-include
- Commit fix: `2b569b8 feat(02-ux): Meta Bind sliders 0-5 + rotina editável inline (tp.file.include)`

## Next Steps

- ✅ Resolvido (rotinas sem frontmatter, include funciona)
- Pattern reutilizável: qualquer arquivo destinado a `tp.file.include` deve ser frontmatter-free
- Alternativa futura considerada: criar pasta `_body/` com versões sem frontmatter dos arquivos que precisam ser includíveis (mantém metadata no original). Não implementado por ser overhead.
