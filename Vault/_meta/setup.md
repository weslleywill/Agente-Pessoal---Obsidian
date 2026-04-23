# Setup Guide — Fase 1 Foundation

Passos manuais que Weslley precisa executar para completar Fase 1. Claude fez a parte autônoma (skeleton de folders + documentação); este guia cobre o resto.

**Endereça:** FOUND-02 (Windows LongPaths), FOUND-03 (10 plugins), FOUND-04 (Templater config), FOUND-06 (GitHub privado)

---

## 1. Windows LongPaths (FOUND-02) — resolve pitfall #11 (MAX_PATH 260)

**Por quê:** Windows 11 ainda usa MAX_PATH=260 por padrão. Vault em `E:\Claude Code\Agente Pessoal - Obsidian\Vault\...` + nota com nome longo + subfolder + anexo = path >260 → git falha silenciosamente, File Explorer não abre o arquivo, backup tools pulam.

**Como fazer:**

Opção A — Registry (mais rápido, requer admin):
1. Abra PowerShell como Administrador
2. Execute:
   ```powershell
   reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f
   ```
3. Reinicie o computador (mudança só aplica após reboot)

Opção B — Group Policy (interface gráfica):
1. `Win+R` → `gpedit.msc`
2. Navegue: Local Computer Policy → Computer Configuration → Administrative Templates → System → Filesystem
3. Abra **"Enable Win32 long paths"** → Enabled → OK
4. Reinicie

**Verificação:** Após reboot, rode em PowerShell:
```powershell
Get-ItemPropertyValue -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name LongPathsEnabled
```
Deve retornar `1`.

---

## 2. GitHub Private Repo + obsidian-git (FOUND-06) — resolve pitfall #14 (privacy leak)

**Por quê:** Sem backup local only = 1 disco morre = tudo perdido. MAS GitHub repo público = 6+ meses de reflexões pessoais world-readable. Regra: repo é **PRIVADO**, sem exceção.

**Como fazer:**

1. Entre em https://github.com/new
2. Preencha:
   - **Repository name:** `agente-pessoal-obsidian-vault` (ou nome de sua escolha)
   - **Description:** "Vault pessoal Obsidian (Agente Pessoal) — privado"
   - **Visibility:** ✅ **Private** (DOUBLE-CHECK! botão tem que estar marcado em Private, NUNCA Public)
   - **Initialize repository:** NÃO marcar nenhum (README/license/gitignore) — vai conflitar com init local
3. Clique "Create repository"
4. Copie a URL `git@github.com:SEUUSUARIO/agente-pessoal-obsidian-vault.git` (SSH) OU `https://github.com/SEUUSUARIO/agente-pessoal-obsidian-vault.git` (HTTPS)

**Verificação dupla de privacidade:**
- Abra a URL do repo no browser
- Cabeçalho deve mostrar 🔒 + "Private" ao lado do nome
- Se vir "Public" → deletar e recriar (não dá pra arriscar)

Ver Plan 02 para `git init` + primeiro push.

---

## 3. Instalação dos 10 Plugins (FOUND-03) + Configuração Templater (FOUND-04)

Ver `_meta/plugins.md` para lista completa. Ordem de instalação recomendada lá.

**Procedimento para cada plugin:**

1. Abrir Obsidian apontando para `E:\Claude Code\Agente Pessoal - Obsidian\Vault\`
2. `Settings` (⚙) → `Community plugins` → `Turn on community plugins` (se primeira vez)
3. `Browse` → buscar pelo Community ID (ex: `templater-obsidian`)
4. `Install` → `Enable`
5. Ir em `Settings` → `Community plugins` e confirmar que aparece "Enabled"

**Configuração específica Templater (FOUND-04):**

1. `Settings` → `Templater`
2. **Template folder location:** `Templates/` (exatamente com a barra)
3. **Trigger Templater on new file creation:** ✅ **ON** (crítico — sem isso Templater não dispara automaticamente)
4. **Folder Template Settings:** (configurar em Phase 2 quando os templates existirem — por enquanto deixar vazio)

**Configuração Homepage:**

1. `Settings` → `Homepage`
2. **Homepage:** `00 - Dashboard/Home.md` (vai estar vazio até Phase 3, mas aponta)
3. **Open on startup:** ✅ ON
4. **Open in Reading view:** ✅ ON (importante — Dataview só renderiza em Reading)

Ver Plan 03 para lista exaustiva de configurações de cada plugin.

---

## 4. Checklist final Phase 1

Marcar cada item quando completado:

- [ ] Windows `LongPathsEnabled=1` (verificado via PowerShell)
- [ ] GitHub repo criado e confirmado **Private** 🔒
- [ ] `git init` + remote + primeiro push feito (Plan 02)
- [ ] 10 plugins instalados e habilitados (Plan 03)
- [ ] Templater "Trigger on new file = ON" + folder `Templates/`
- [ ] Homepage apontando pra `00 - Dashboard/Home.md` em Reading mode
- [ ] obsidian-git configurado com auto-commit 30min
- [ ] Teste: criar nota com path >250 chars em `01 - Diário/` → abre sem erro
- [ ] Teste: `Ctrl+N` cria nota em folder mapeado e Templater dispara (após Phase 2 que cria templates)
