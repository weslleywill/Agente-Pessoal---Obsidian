---
slug: periodic-notes-folder-no-interpola
title: Periodic Notes v1.0.0-beta1 — folder não interpola variáveis Moment
status: resolved
created: 2026-04-25
updated: 2026-04-25
---

# Thread: Periodic Notes v1.0.0-beta1 — folder não interpola

## Goal

Documentar bug/quirk do plugin Periodic Notes (liamcain) v1.0.0-beta1: o campo `folder` é tratado como string LITERAL, não interpola variáveis Moment como `YYYY/MM-MMMM`. O `format` SIM interpola.

## Context

*Descoberto em 2026-04-25 durante teste da Fase 2 do projeto Agente Pessoal Obsidian.*

**Sintoma observado:**
- Daily de hoje foi criada em `01 - Diário/YYYY/MM-MMMM/2026-04-24.md`
- A pasta `YYYY/MM-MMMM` foi criada com esse nome **literal** (não interpolado pra `2026/04-Abril`)
- Esperado: `01 - Diário/2026/04-Abril/2026-04-24.md`

**Causa raiz:**
Na v1.0.0-beta1 do plugin (id `periodic-notes`, autor liamcain), o schema da config ficou assim:
```json
{
  "calendarSets": [{
    "day": {
      "format": "YYYY-MM-DD",     // INTERPOLA (Moment)
      "folder": "01 - Diário/YYYY/MM-MMMM"  // LITERAL (string fixa)
    }
  }]
}
```

**Solução correta:**
Path completo (incluindo subfolders dinâmicos) vai no `format`. `folder` só raiz:
```json
{
  "day": {
    "format": "YYYY/MM-MMMM/YYYY-MM-DD",  // path completo, INTERPOLA
    "folder": "01 - Diário"               // raiz fixa
  }
}
```

## References

- Arquivo de config: `Agente Pessoal - Obsidian/.obsidian/plugins/periodic-notes/data.json`
- Plugin: https://github.com/liamcain/obsidian-periodic-notes
- Versão afetada: 1.0.0-beta1
- Commit fix: `ff06a93 fix(02): periodic-notes path interpolation + Templates/_scripts folder`

## Next Steps

- ✅ Resolvido em sessão de origem (config corrigido + nota movida)
- Se atualizar plugin pra versão estável (>= 1.0.0 GA), reverificar comportamento — schema pode ter mudado
- Documentar este quirk em `_meta/plugins.md` se ainda não estiver
- Aplicável também a configs de `weekly` e `monthly` (mesmo schema)
