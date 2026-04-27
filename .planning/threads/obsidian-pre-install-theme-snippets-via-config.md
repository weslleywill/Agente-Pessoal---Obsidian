---
slug: obsidian-pre-install-theme-snippets-via-config
title: Pré-instalar tema Obsidian via curl + ativar snippets via appearance.json (sem Settings UI)
status: resolved
created: 2026-04-26
updated: 2026-04-26
---

# Thread: pré-instalar tema Obsidian via curl + ativar snippets via appearance.json

## Goal

Documentar pattern reutilizável pra reduzir fricção em testes de UI no Obsidian — pré-instalar temas e ativar CSS snippets via configuração direta de arquivos, sem precisar que o usuário toggue manualmente em Settings.

## Context

*Pattern aplicado em 2026-04-26 durante UX Iteration #2.*

### Contexto da descoberta

Usuário pediu pra "testar" 2 temas (Minimal vs AnuPpuccin) + ativar 2 CSS snippets (`sliders` + `polimento-geral`). Caminho tradicional teria fricção:

1. Settings → Appearance → Themes → Manage → buscar "Minimal" → Install
2. Mesma tela: buscar "AnuPpuccin" → Install
3. Settings → Appearance → CSS snippets → 🔄 atualizar → toggle "sliders" ON
4. Mesma tela: toggle "polimento-geral" ON

**Pattern descoberto**: pode ser feito tudo via configuração direta — usuário só precisa Reload do Obsidian (1 ação).

### Pattern: instalar tema via curl

Estrutura esperada por Obsidian: `.obsidian/themes/<Nome>/manifest.json` + `.obsidian/themes/<Nome>/theme.css`.

```bash
mkdir -p ".obsidian/themes/<Nome>/"
curl -sL "https://raw.githubusercontent.com/<owner>/<repo>/<branch>/manifest.json" \
  -o ".obsidian/themes/<Nome>/manifest.json"
curl -sL "https://raw.githubusercontent.com/<owner>/<repo>/<branch>/theme.css" \
  -o ".obsidian/themes/<Nome>/theme.css"
```

Pra setar como ativo: editar `.obsidian/appearance.json`:

```json
{
  "cssTheme": "<Nome>"
}
```

`<Nome>` deve **bater exatamente** com o `name` do `manifest.json` (case-sensitive).

### Pattern: ativar CSS snippets via appearance.json

Snippets ficam em `.obsidian/snippets/<nome>.css`. Pra ativar **sem ir em Settings**, editar `.obsidian/appearance.json`:

```json
{
  "enabledCssSnippets": [
    "sliders",
    "polimento-geral"
  ]
}
```

Nome do snippet = nome do arquivo sem extensão (ex: `sliders.css` → `"sliders"`).

### Validação aplicada (UX Iteration #2)

```bash
mkdir -p .obsidian/themes/Minimal .obsidian/themes/AnuPpuccin

# Minimal v8.1.7 (kepano)
curl -sL https://raw.githubusercontent.com/kepano/obsidian-minimal/master/manifest.json \
  -o .obsidian/themes/Minimal/manifest.json   # 200 OK, ~250 bytes
curl -sL https://raw.githubusercontent.com/kepano/obsidian-minimal/master/theme.css \
  -o .obsidian/themes/Minimal/theme.css       # 200 OK, 248304 bytes

# AnuPpuccin v1.5.0 (Anubis)
curl -sL https://raw.githubusercontent.com/AnubisNekhet/AnuPpuccin/main/manifest.json \
  -o .obsidian/themes/AnuPpuccin/manifest.json   # 200 OK
curl -sL https://raw.githubusercontent.com/AnubisNekhet/AnuPpuccin/main/theme.css \
  -o .obsidian/themes/AnuPpuccin/theme.css       # 200 OK, 370085 bytes
```

Resultado em `appearance.json`:
```json
{
  "accentColor": "#a879ff",
  "baseFontSize": 16,
  "theme": "obsidian",
  "cssTheme": "Minimal",
  "enabledCssSnippets": ["sliders", "polimento-geral"],
  "showViewHeader": true
}
```

Usuário só precisa: `Ctrl+P → Reload app without saving`. Tudo aplicado.

### Atenção: minAppVersion

`manifest.json` tem campo `minAppVersion`. Se Obsidian instalado for mais antigo que isso, tema não carrega silenciosamente.

**Validação prévia:**
- Minimal v8.1.7 → minAppVersion: 1.9.0
- AnuPpuccin v1.5.0 → minAppVersion: 1.6.0

Se usuário tem Obsidian 1.5.x, AnuPpuccin funciona, Minimal não. Em caso de incompatibilidade, **NÃO há erro visível** — usuário pode achar que comando não funcionou. Sempre validar versão antes.

### Quando usar este pattern

✅ **Bom uso:**
- Reduzir fricção de teste (usuário aprova X feature visual em 1 reload em vez de 5 cliques)
- Setup novo de vault (reproduzir config de um vault existente em outro)
- Onboarding (alguém clona vault e já tem tudo configurado)

❌ **Evitar:**
- Quando usuário quer escolher entre opções lado-a-lado (deixar Settings UI ele explora natural)
- Temas com licença/paywall (kepano e AnubisNekhet são MIT/livres; outros podem não ser)
- Forçar tema sem validar `minAppVersion` (frustração silenciosa)

## References

- Repos confirmados:
  - Minimal (kepano): https://github.com/kepano/obsidian-minimal — branch `master`
  - AnuPpuccin (Anubis): https://github.com/AnubisNekhet/AnuPpuccin — branch `main`
- Pattern documentado em: `MEMORY.md` seção "🎨 Pré-instalar tema Obsidian via curl"
- Aplicado em commit: `4212094 chore(ux-iteration-2): pre-instalar temas Minimal/AnuPpuccin + ativar snippets`

## Next Steps

- ✅ Resolvido (validado em produção — vault privado do Weslley)
- Considerar: criar script `_meta/setup-themes.sh` que reproduz o setup pra novos vaults Obsidian em projetos futuros
- Considerar: pesquisar `minAppVersion` automaticamente via Obsidian API antes de instalar (V2 — só relevante se isso virar feature recorrente)
- Cross-projeto: portátil pra qualquer setup Obsidian, não só este vault
