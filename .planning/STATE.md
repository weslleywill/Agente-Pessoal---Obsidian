# STATE: Agente Pessoal Obsidian

**Initialized:** 2026-04-23
**Last updated:** 2026-04-26 after UX Iteration #2 + extract-learnings retroativo Phase 1 (01-LEARNINGS.md gerado)

---

## Project Reference

**Core Value:** Receber coaching real e sem achismo — priorização socrática, tracking calibrado por histórico, análise semanal/mensal fundamentada em livros — enquanto o sistema de capture (planner digital) tem fricção mínima.

**Current Focus:** Fases 1, 2, 2-UX, 3 e UX-Iteration-2 concluídas. Próximo passo: `/gsd-plan-phase 4` para decompor Phase 4 (Coach anti-alucinação + protocolo socrático) em plans executáveis. Antes: aguardando teste consolidado do usuário pós UX-Iteration-2 (snippets ativados + rotina interativa funcionando + tema visual escolhido entre Minimal/AnuPpuccin).

**Granularity:** Coarse (6 phases)
**Mode:** YOLO
**Model Profile:** Quality (Opus 4.7)

---

## Current Position

**Phase:** 4 — Coach anti-alucinação + protocolo socrático (próxima a planejar)
**Plan:** N/A (phase 4 ainda não planejada)
**Status:** Awaiting `/gsd-plan-phase 4` (após teste do usuário pós UX-Iteration-2)
**Progress:** [▰▰▰▱▱▱] 3/6 phases complete (1, 2, 3 done; 2-UX e UX-Iteration-2 micro-iterações pós-Phase 3)

### UX Iteration #2 (2026-04-25) — pós-teste consolidado da Phase 3

**Trigger:** feedback do usuário ao testar vault (Home + daily) no Obsidian.

**4 melhorias coordenadas:**
1. **Banco de Frases 100% pt-BR** — 40 frases de autores estrangeiros traduzidas (mantém autor/fonte; títulos em pt-BR quando edição BR existe)
2. **CSS snippet `sliders.css`** — accent-color + gradient track + thumb 18px com sombra + hover/active scale (resolve "slider sem fill à esquerda")
3. **CSS snippet `polimento-geral.css`** — headings, callouts, tabelas, embeds, links, tags, properties, HR, checkboxes, blockquotes (resolve "interface não tão bonita")
4. **Rotina interativa por bloco de horário** — substitui embed read-only por 18 `INPUT[suggester(option(...), allowOther):agenda_real.hHH_MM]` (resolve "ainda não consigo escrever" + sugestões contextuais com texto livre via allowOther)

**Schema novo:** daily ganha `agenda_real.{h06_00...h22_30}` (18 chaves). Migração graciosa via "regra de adicionar campo novo" (campos novos = ok sem migration).

**Commit vault:** `63d6cff feat(ux-iteration-2): rotina interativa + sliders/UI polidos + frases pt-BR` no repo Agente-Pessoal-Obsidian (privado)

**Release notes:** `_meta/release-notes/2026-04-25-ux-iteration-2.md` (dentro do vault)

**Pendência de teste do usuário antes de Phase 4:**
- [ ] Reload do Obsidian
- [ ] Ativar 2 snippets (`sliders` + `polimento-geral`) em Settings → Appearance → CSS snippets
- [ ] Instalar 1 tema (Minimal OU AnuPpuccin) e comparar
- [ ] Abrir daily de hoje e testar dropdown de cada bloco de horário (escolher sugestão + tentar texto livre via `allowOther`)
- [ ] Validar frase do dia em pt-BR

**V2 da rotina (lock pra próxima micro-iteração):** banco compartilhado `05 - Rotinas/Banco-Atividades.md` + auto-promote inteligente via DataviewJS (atividade mais frequente em 30 dailies vira #1) + condicional dia-útil/fim-semana.

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| v1 requirements total | 72 |
| Requirements mapped | 72/72 (100%) |
| Phases defined | 6 |
| Phases completed | 3 (1, 2, 3) + 2 micro-iterações (2-UX, UX-Iteration-2) |
| Plans completed | 8 (01-01, 02-01..04, 03-01..03) |
| Requirements completed | ~50/72 (Fases 1, 2, 3) |
| Current phase plans | TBD (awaiting `/gsd-plan-phase 4`) |
| Plugins instalados | 11 (10 core + Meta Bind UX iteration) |
| Threads abertos | 5 (1 alto valor crítico Fase 4) |

---

## Accumulated Context

### Decisões locked (do PROJECT.md e research)

- **Granularidade Coarse** — 6 fases, dependency-driven (do research SUMMARY.md)
- **Vault separado de `.planning/`** — `.planning` controla projeto GSD, `Agente Pessoal - Obsidian/` é o produto (vault, renomeado de `Vault/`)
- **Vault NÃO em cloud sync** — E:\ local apenas; backup via obsidian-git → GitHub **privado**
- **Daily frontmatter = single source of truth** — weekly/monthly = Dataview live, nunca duplicam
- **Anti-alucinação ESTRUTURAL** — Glob+Read em `08 - Livros/`, não só prompt
- **Dashboard decomposto** — Home leve + Semana-Atual + Mes-Atual pesados
- **Dataview 0.5.70 no MVP, Bases futuro** — ecossistema ainda assume Dataview
- **"Cabeça de Campeão" = Ducasse, NÃO Joel Jota** — lesson learned valida anti-alucinação estrutural
- **Coach protocolo socrático 6 passos** — anti-achismo é requisito explícito
- **Rotinas via `Ativa.md` como ponteiro** — trocar rotina = editar 1 linha
- **Todos 10 extras (A-J) aprovados** — usuário gostou de todos no brainstorm

### Pitfalls conhecidos (do PITFALLS.md, 15 total mapeados)

**Top 5 críticos:**
1. Coach hallucinates book citations — mitigação estrutural (Phase 4)
2. Sync corruption OneDrive/iCloud/Dropbox — vault local + git privado (Phase 1)
3. Dataview query bloat após ~1k notas — scope + LIMIT (Phase 3)
4. Score inflation + performative completion — detecção pattern (Phase 5)
5. Plugin abandonment — `_meta/plugins.md` com fallback (Phase 1)

### Todos / pendências

- [ ] **Aguardando teste do usuário pós UX-Iteration-2** — reload + ativar 2 snippets + instalar tema + testar rotina interativa + validar frases pt-BR
- [ ] Executar `/gsd-plan-phase 4` (Coach) — provavelmente requer research específica (grounding workflow + protocolo socrático)
- [ ] Validar escolha final Pomodoro Timer vs TaskNotes (deferred — observar uso real)
- [ ] Confirmar status Pentagrama/3Ps (research flagged — não foi encontrado no planner físico, tratar como narrativa)
- [ ] Considerar `/gsd-extract_learnings` retroativo para Fases 2 e 3 (não têm SUMMARY próprio)
- [ ] V2 da rotina interativa (próxima micro-iteração) — banco compartilhado `05 - Rotinas/Banco-Atividades.md` + auto-promote DataviewJS + condicional dia-útil/fim-semana

### Blockers

Nenhum no momento. Trabalho 100% sincronizado com GitHub (2 repos privados).

---

## Session Continuity

### Última sessão (2026-04-23 → 2026-04-25, sprint maratona)

**Ações executadas:**
- Fase 0: PROJECT.md, REQUIREMENTS.md, ROADMAP.md, research completa
- Fase 1: vault `Agente Pessoal - Obsidian/` com 14 folders + 11 plugins + 2 repos GitHub privados
- Fase 2: schema + rubric + fields + 5 templates + 5 fragments + rotinas + 52 frases + 3 livros seed
- Fase 2-UX: Meta Bind 1.4.8 + sliders 0-5 + rotina editável inline (`tp.file.include`)
- Fase 3: dashboard decomposto (Home leve + Semana-Atual + Mes-Atual pesados) + tracking infra (calibration + time-logging + tracking-integration)
- 5 threads criados com aprendizados de plugin/arquitetura
- MEMORY.md + session-logs/2026-04-25 documentando tudo

**Próxima ação recomendada:**
```
/gsd-plan-phase 4
```

Decompõe Phase 4 (Coach anti-alucinação + protocolo socrático) em plans executáveis — research necessária pra grounding workflow (Glob + Read em `08 - Livros/`), protocolo de 6 passos, comandos `/prepara meu dia` e `/loga`, friction budget, 2-modes detection.

**ANTES de plan-phase 4:** aguardar usuário testar Fases 2-UX e 3 no Obsidian (1 reload + abrir Home + sliders na daily).

### Research flags ativas (para plan-phase)

Phases que se beneficiam de `/gsd-research-phase` durante planning:
- **Phase 4 (Coach):** implementação detalhada do grounding workflow (Glob+Read pipeline)
- **Phase 5 (Análise):** algoritmos de pattern detection (threshold exato copy-paste, std dev)
- **Phase 6 (Extras):** export PDF, time-blocking algoritmo, wearable CSV parsing

Phases com padrões bem-documentados (skip research):
- **Phase 1:** Windows + git (caminhos trilhados)
- **Phase 2:** schema + templates (padrão Obsidian)
- **Phase 3:** Dashboard + Dataview + Tracker (patterns consolidados)

---

*STATE.md evolves at every phase transition and significant checkpoint.*
