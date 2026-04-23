# Backup Strategy

**Endereça:** FOUND-06, FOUND-07. Resolve pitfalls #5 (sync corruption) e #14 (privacy leak).

---

## Regra #1: NUNCA em cloud sync

Vault **JAMAIS** pode ficar em:
- ❌ OneDrive (qualquer folder sincronizado)
- ❌ iCloud Drive (Windows é especialmente bugado — duplica até 56x)
- ❌ Dropbox
- ❌ Google Drive folder sincronizado

**Path atual correto:** `E:\Claude Code\Agente Pessoal - Obsidian\Vault\` (drive E: local, não sincronizado) ✓

**Se acontecer corrupção (arquivos `(1).md`, `(2).md`, hostname-suffix):**
- Fechar Obsidian
- Fechar cliente de sync (se houver)
- `git reset --hard HEAD` para voltar ao último commit limpo
- Mover vault para fora do folder sync
- Documentar o incidente em `_meta/incidents.md`

---

## Regra #2: Backup único é obsidian-git → GitHub privado

- **Plugin:** obsidian-git (Vinzent03)
- **Repo:** GitHub, **PRIVATE** (verificação dupla — ver `setup.md` §2)
- **Intervalo auto-commit:** a cada 30 minutos de atividade + ao fechar Obsidian
- **Commit message format:** `vault: YYYY-MM-DD HH:MM` (obsidian-git default ok)

**Configurar em Obsidian:**
1. `Settings` → `Obsidian Git`
2. **Vault backup interval (minutes):** `30`
3. **Commit when Obsidian is closing:** ✅ ON
4. **Auto pull on startup:** ✅ ON (single-device, mas por segurança)
5. **Commit message:** deixar default ou `vault: {{date}} {{hostname}}`

---

## Regra #3: .gitignore é sagrado

Arquivo `Vault/.gitignore` (criado por Plan 01 Task 3) exclui:
- `workspace.json` (muda a cada abertura — git noise)
- `.obsidian/cache/` (cache local, não faz sentido versionar)
- `_private/` (reflexões sensíveis — privacy pitfall #14)
- `.obsidian/plugins/*/data.json` (dados específicos de instalação)
- `.trash/` (Obsidian built-in trash)

**Se mudar algo aqui → revisar todos os commits passados pra checar se nada sensível escapou.**

---

## Regra #4: Verificação mensal

Todo final de mês:
1. Abrir GitHub repo no browser
2. Verificar badge: 🔒 Private (se vier Public → emergência, rotacionar tudo sensível que possa ter vazado)
3. Conferir último commit: deve ter timestamp do mês atual
4. Se >7 dias sem commit: obsidian-git pode ter quebrado; verificar plugin

---

## Recovery

| Cenário | Ação |
|---|---|
| Perdi último dia de trabalho | `git log --oneline` + `git checkout <commit>` do arquivo específico |
| Corrupção por sync (arquivos `(N).md`) | `git reset --hard HEAD` + mover vault pra fora de pasta sync |
| Plugin obsidian-git quebrou | Fallback: `git commit -am "manual: YYYY-MM-DD"` + `git push` via terminal |
| HD morreu | `git clone` do remote privado → vault reconstituído |
| Accidentally committed `_private/` | `git filter-branch` ou `bfg-repo-cleaner` + force-push + rotate qualquer credencial mencionada |
