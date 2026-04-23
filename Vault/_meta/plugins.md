# Plugins — Registro e Fallbacks

**Atualizado:** 2026-04-23 (verificado via GitHub releases na research/STACK.md)

10 plugins community obrigatórios para v1. 4 estão em status stale/limbo mas funcionais. TODOS têm rota de fallback documentada (pitfall #3: plugin abandonment).

**Endereça:** FOUND-05

---

## Tabela de plugins

| # | Plugin | Community ID | Versão alvo | Status manutenção | O que quebra se morrer | Fallback |
|---|--------|--------------|-------------|-------------------|------------------------|----------|
| 1 | Templater | `templater-obsidian` | 2.19.3 | ATIVO (release 2026-04-22) | Folder-template mapping, lógica condicional, user functions. Daily note vira só Daily Notes core. | Core "Templates" (primitivo; sem folder mapping; aceitar degradação) |
| 2 | Dataview | `dataview` | 0.5.70 | MAINTENANCE-ONLY (autor foco em Datacore) | Todas queries do dashboard, weekly/monthly agregações, rollups | Bases core plugin (Obsidian 1.9+); migração parcial em Phase 6+ |
| 3 | Periodic Notes | `periodic-notes` | 1.0.0-beta.3 | ABANDONADO (funcional) | Weekly/Monthly/Quarterly/Yearly criação estruturada | Periodic Notes Calendar (luiisca) — drop-in mas precisa re-configurar |
| 4 | Calendar | `calendar` | 2.0.0-beta.2 | ABANDONADO (funcional; 5 anos sem update) | Widget sidebar clicável; integração com Periodic Notes | QuickCalendar (snovis) ou Periodic Notes Calendar (luiisca) |
| 5 | Tasks | `obsidian-tasks-plugin` | 7.23.1 | ATIVO | Queries de tarefas pendentes, recurring (🔁 every week), syntax emojis | Task Genius / TaskForge (menos maduros; refatoração grande) |
| 6 | Homepage | `homepage` | 4.4.0 | ATIVO | Abertura automática de `00 - Dashboard/Home.md` em Reading mode | Nenhum — abrir manualmente (degradação aceitável) |
| 7 | Tracker | `obsidian-tracker` | 1.19.0 | ATIVO (commit <10 dias) | Charts dos 5 pilares ao longo do tempo em Semana-Atual/Mes-Atual | Charts plugin DESCARTADO (2 anos sem release). Life Tracker (Bases-native) como alternativa futura |
| 8 | Pomodoro Timer | `pomodoro-timer` | 1.2.3 | LIMBO (último push 2024-06) | Pomodoros ligados a tasks específicas; `pomodoros::N` inline field | TaskNotes (callumalpass) — reescreve tasks como notas (refatoração grande; só se Pomodoro quebrar) |
| 9 | Natural Language Dates | `nldates-obsidian` | 0.6.2 | ABANDONADO (funcional) | Input `@tomorrow`, `@next friday` em Tasks e daily notes | nldates-revived (Amato21) — drop-in, zero refactor |
| 10 | obsidian-git | `obsidian-git` | Latest (Vinzent03, ativo) | ATIVO | Backup automático 30min → GitHub privado | git CLI manual (`git commit -am` + `git push`) — manual mas possível |

---

## Ordem de instalação recomendada

1. Templater (base para criação de notas)
2. Periodic Notes (estrutura daily/weekly/monthly)
3. Calendar (widget sidebar)
4. Natural Language Dates (input dates)
5. Tasks
6. Dataview
7. Homepage
8. Tracker
9. Pomodoro Timer
10. obsidian-git

---

## Conflitos conhecidos

- **Tasks ↔ Pomodoro Timer**: `pomodoros::N` inline field DEVE vir ANTES dos emojis Tasks (`📅 ⏫`) na linha. Caso contrário, Tasks plugin não parseia o resto.
- **Periodic Notes + core Daily Notes**: precisam estar sincronizados (mesmo folder, mesmo template) — ou desabilitar core Daily Notes.
- **Homepage ↔ Dataview**: issue antigo (#79 com Obsidian 1.5.6) já resolvido em Homepage 4.4.0.

---

## Auditoria periódica

Re-verificar este arquivo em cada milestone boundary:
- Plugin teve commit nos últimos 90 dias? (se não → flag monitoramento)
- Plugin ainda funciona na versão atual do Obsidian? (testar)
- Fallback ainda é válido? (fallback pode ter abandonado também)

Pendência Phase 1: validar decisão final Pomodoro Timer vs TaskNotes durante audit da Phase 1 (ver STATE.md).
