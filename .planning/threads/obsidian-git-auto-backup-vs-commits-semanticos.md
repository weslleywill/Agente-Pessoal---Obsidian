---
slug: obsidian-git-auto-backup-vs-commits-semanticos
title: obsidian-git auto-backup compete com commits semânticos — `git status` pode mentir
status: resolved
created: 2026-04-26
updated: 2026-04-26
---

# Thread: obsidian-git auto-backup compete com commits semânticos

## Goal

Documentar que o plugin `obsidian-git` está rodando auto-backup a cada 30min no vault, e que isso afeta o fluxo de commits semânticos do Claude Code (e do Weslley).

## Context

*Descoberto em 2026-04-26 durante UX Iteration #2.*

### Sintoma

Após fazer `Write` em 3 arquivos (`07 - Frases/Banco-Frases.md`, `.obsidian/snippets/sliders.css`, `.obsidian/snippets/polimento-geral.css`) e tentar `git status` minutos depois, o status retornou **"working tree clean"** — sem mencionar os arquivos como modified/untracked.

Causa-raiz: o plugin `obsidian-git` (configurado em Phase 1) faz commits automáticos a cada 30 minutos com mensagem padrão:

```
vault backup: YYYY-MM-DD HH:MM:SS
```

O backup capturou os arquivos ANTES do meu commit semântico. Resultado: `git log --oneline` ficou:

```
43d3ed7 vault backup: 2026-04-25 10:30:10
5761157 vault backup: 2026-04-25 10:00:09
c5ddb6f feat(03): execute Phase 3 — dashboard decomposto + tracking infra
```

### Implicações práticas

1. **`git status` pode mentir** sobre o que mudou na sessão atual
   - Solução: confiar em `git log --oneline -10` em vez de só `status` quando rastreando o que aconteceu na sessão
   - Ou: `git log --oneline --all --since="2 hours ago"` pra cobrir backups recentes

2. **Histórico fica intercalado** — `vault backup → vault backup → feat(...)` no mesmo dia
   - Aceitar isso é parte do trade-off: auto-backup = defesa contra perda de dados; commits semânticos = legibilidade
   - Em PRs/code review: filtrar mensagens "vault backup" do diff visual

3. **Ainda devemos fazer commits semânticos** depois do trabalho
   - Git aceita commits adicionais sem problema
   - O commit semântico final consolida + documenta o "porquê" da mudança
   - Mensagem do auto-backup é só timestamp; semântico explica intenção

4. **Não desligar o auto-backup** — é a estratégia de defesa contra perda de dados (FOUND-07 e pitfall #2 do projeto)

### Workflow correto pós-descoberta

```
1. Editar arquivos do vault
2. (auto-backup pode acontecer em ~30min — sem problema)
3. Fazer git status / git log antes de commit semântico — conferir se backup já capturou
4. Fazer commit semântico mesmo assim:
   git add <arquivos específicos>
   git commit -m "feat(...): mensagem clara"
5. git push (se quiser sync remoto imediato — auto-backup ALSO faz push)
```

### Como verificar se obsidian-git está ativo

```bash
ls "Agente Pessoal - Obsidian/.obsidian/plugins/obsidian-git/" 2>&1
cat "Agente Pessoal - Obsidian/.obsidian/plugins/obsidian-git/data.json" | grep -i "interval\|autoCommit\|autoPush"
```

Se `commitInterval` está setado (default em vaults configurados pelo Weslley = 30 min), auto-backup roda.

## References

- Plugin: https://github.com/Vinzent03/obsidian-git
- Mitigação documentada em: `MEMORY.md` seção "🔁 Auto-backup obsidian-git a cada 30min"
- Commits exemplo (UX Iteration #2):
  - `5761157 vault backup: 2026-04-25 10:00:09` (auto)
  - `43d3ed7 vault backup: 2026-04-25 10:30:10` (auto)
  - `63d6cff feat(ux-iteration-2): rotina interativa + sliders/UI polidos + frases pt-BR` (semântico — eu)
  - `4212094 chore(ux-iteration-2): pre-instalar temas Minimal/AnuPpuccin + ativar snippets` (semântico — eu)

## Next Steps

- ✅ Resolvido (entendido + documentado)
- Considerar: configurar `obsidian-git` pra usar mensagem de backup mais informativa (ex: incluir lista de arquivos modificados) — opcional, pesquisar plugin settings
- Considerar: na Phase 4 (coach) — coach pode ignorar commits "vault backup" ao analisar histórico
- Em PRs entre repos GSD e vault: documentar pra reviewers que filtros "exclude: vault backup" são úteis
