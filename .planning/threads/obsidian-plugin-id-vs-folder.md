---
slug: obsidian-plugin-id-vs-folder
title: Validar manifest.id vs nome da pasta antes de habilitar plugin Obsidian
status: resolved
created: 2026-04-25
updated: 2026-04-25
---

# Thread: manifest.id vs nome da pasta — validação antes de community-plugins.json

## Goal

Pattern operacional: ao baixar plugin Obsidian via `releases/latest/download/`, o nome do **repositório** GitHub frequentemente NÃO é igual ao **id** declarado no `manifest.json`. Se a pasta em `.obsidian/plugins/` não bater com `manifest.id`, o plugin não carrega.

## Context

*Descoberto em 2026-04-25 durante Fase 1 do projeto.*

**Sintoma:**
Baixei via curl 10 plugins Obsidian. Para o Calendar (liamcain), criei pasta `.obsidian/plugins/calendar/` baseado no nome esperado. Mas:
- Repo: `liamcain/obsidian-calendar-plugin`
- `manifest.json` declara: `"id": "calendar-beta"` (não `calendar`)

Plugin não carrega — Obsidian busca pasta com nome igual ao id.

**Solução aplicada:**
1. Renomeei `.obsidian/plugins/calendar/` → `.obsidian/plugins/calendar-beta/`
2. Atualizei `community-plugins.json` para usar `calendar-beta` (não `calendar`)
3. Validação posterior:

```bash
for dir in .obsidian/plugins/*/; do
  folder=$(basename "$dir")
  id=$(node -e "console.log(require('./$dir/manifest.json').id)")
  [ "$folder" != "$id" ] && echo "MISMATCH: $folder vs $id"
done
```

**Padrão de instalação correto:**
1. Baixar `manifest.json` primeiro
2. Ler `manifest.id` via parser JSON real (não regex)
3. Criar pasta com nome = `manifest.id`
4. Baixar `main.js` e `styles.css` para essa pasta
5. Adicionar `manifest.id` (não nome do repo) em `community-plugins.json`

## References

- Plugin afetado: Calendar (liamcain) v2.0.0
- Repo: https://github.com/liamcain/obsidian-calendar-plugin
- Sem commit dedicado — fix foi parte do batch da Fase 1
- Conhecimento documentado em `Agente Pessoal - Obsidian/_meta/plugins.md` (tabela de plugins)

## Next Steps

- ✅ Validação automatizada em script de instalação (futuro)
- Considerar: plugins com sufixo `-beta` no id geralmente são forks/betas — pode haver versão estável com id diferente. Revisar quando atualizar.
- Aplicável a qualquer plugin Obsidian instalado por curl direto (BRAT plugin, instalações automatizadas, vault-portátil)
