# Roadmap: Agente Pessoal Obsidian

**Created:** 2026-04-23
**Granularity:** Coarse (6 fases)
**Coverage:** 72/72 v1 requirements mapped

**Core Value:** Receber coaching real e sem achismo — priorização socrática, tracking calibrado por histórico, análise semanal/mensal fundamentada em livros — enquanto o sistema de capture (planner digital) tem fricção mínima.

---

## Phases

- [ ] **Phase 1: Fundação** — Vault local + plugins + Windows setup + backup privado (anti-corrupção + anti-abandono de plugin)
- [ ] **Phase 2: Templates + Schema + Rubric** — Schema congelado, templates fiéis ao planner físico, rotinas + frases + livros seed (prevenção de schema drift + template bloat)
- [ ] **Phase 3: Dashboard + Queries + Tracking** — Dashboard decomposto (Home leve + Semana/Mês pesados), queries scoped + LIMIT, pomodoro integrado, validado com daily sintéticas
- [ ] **Phase 4: Coach + Anti-Alucinação** — Protocolo socrático 6 passos + filesystem check estrutural em `08 - Livros/` + modos `/prepara` vs `log` + feedback loop calibration
- [ ] **Phase 5: Análise Semanal/Mensal** — Coach semanal/mensal + detecção de score inflation + performative completion + honestidade check + calibration review
- [ ] **Phase 6: Extras (A-J) + Hardening** — 10 extras aprovados + annual method review + performance audit com 365+ notas

---

## Phase Details

### Phase 1: Fundação

**Goal:** Infraestrutura de armazenamento, plugins e backup estabelecida para que capture possa começar SEM risco de corrupção, perda de dados ou conflito de sync.

**Depends on:** Nothing (primeira fase)

**Requirements:** FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05, FOUND-06, FOUND-07, FOUND-08

**Success Criteria** (what must be TRUE):
  1. Usuário abre Obsidian em `E:\Claude Code\Agente Pessoal - Obsidian\Vault\` e vê os 11 folders de topo (00-Dashboard até 10-Projetos + Templates + _meta + _private) sem erros de carregamento
  2. Usuário cria arquivo de teste com path > 250 chars e consegue abrir/editar/commit sem erro de Windows MAX_PATH
  3. Usuário roda `git push` manual e vê commit aparecer em repositório GitHub confirmado como **privado** (não público)
  4. Usuário instala os 10 plugins community e Templater dispara automaticamente ao criar nota nova em folder mapeado
  5. Usuário abre `_meta/plugins.md` e vê cada um dos 10 plugins documentado com versão atual + status de manutenção + fallback

**Plans:** TBD
**UI hint:** no

---

### Phase 2: Templates + Schema + Rubric

**Goal:** Estrutura de dados (schema frontmatter + rubric 0-5 + templates fiéis ao planner físico) congelada antes de qualquer captura real, para que weekly/monthly sempre consigam agregar daily sem drift silencioso.

**Depends on:** Phase 1 (folders + Templater + Periodic Notes)

**Requirements:** TMPL-01, TMPL-02, TMPL-03, TMPL-04, TMPL-05, TMPL-06, TMPL-07, TMPL-08, TMPL-09, RTN-01, RTN-02, RTN-03, RTN-04, RTN-05, FRASE-01, FRASE-02, FRASE-03, FRASE-04, LIVR-01, LIVR-02, LIVR-03, LIVR-04

**Success Criteria** (what must be TRUE):
  1. Usuário cria nota diária via Periodic Notes e frontmatter vem preenchido com os campos canônicos (data, pilares com 5 sub-campos 0-5, prioridades, pomodoros) conforme `_meta/schema.md`
  2. Usuário consulta `_meta/rubric.md` e encontra âncoras concretas 1-5 por pilar (ex: "Saúde 5 = treinei + comi limpo + 7h sono + energia alta")
  3. Usuário edita `05 - Rotinas/Ativa.md` para apontar de `Rotina-Dia-Util` para `Rotina-Fim-Semana` e muda rotina globalmente em 1 edit
  4. Usuário abre qualquer dia do ano e vê 1 frase motivacional rotativa distinta (`dayOfYear % length`), com autor + fonte rastreáveis
  5. Usuário preenche 1 daily + 1 weekly + 1 monthly completos em menos de 3 minutos (daily), 15 min (weekly), 30 min (monthly) — templates fiéis ao planner físico sem bloat

**Plans:** TBD
**UI hint:** no

---

### Phase 3: Dashboard + Queries + Tracking

**Goal:** Dashboard decomposto renderiza sob stress (365+ notas sintéticas) em <1s cold, com pomodoro e log manual de tempo operacionais, validado end-to-end antes de qualquer dado real entrar.

**Depends on:** Phase 2 (templates + schema + seed data para queries terem o que agregar)

**Requirements:** DASH-01, DASH-02, DASH-03, DASH-04, DASH-05, DASH-06, DASH-07, DASH-08, DASH-09, TRACK-01, TRACK-02, TRACK-03, TRACK-04, TRACK-05

**Success Criteria** (what must be TRUE):
  1. Usuário abre Obsidian e `Home.md` renderiza em <1s cold com saudação + frase rotativa + bloco HOJE + embed `Ativa.md` + links para sub-dashboards (zero Tracker aqui)
  2. Usuário clica em `Semana-Atual.md` e vê score médio 5 pilares + gráfico Tracker dos últimos 7/30 dias renderizado sem freeze
  3. Usuário inicia Pomodoro Timer em tarefa da nota do dia, finaliza sessão, e vê `pomodoros: N` incrementado no frontmatter automaticamente
  4. Usuário digita `/loga 14:00-15:30 revisar relatório` e Claude preenche a seção "Log de Tempo" com formato estruturado (estimado vs real), atualizando `tarefas_log` no frontmatter
  5. Usuário cria vault vazio (Day 0, zero daily notes) e abre dashboard — nenhuma query renderiza erro vermelho; todas mostram empty state com mensagem graciosa

**Plans:** TBD
**UI hint:** yes

---

### Phase 4: Coach + Anti-Alucinação

**Goal:** Coach socrático funcional com grounding estrutural em filesystem (não só prompt) — cita apenas livros que existem em `08 - Livros/`, diferencia modo deep de modo capture, aprende com correções do usuário.

**Depends on:** Phase 2 (livros seed + calibration data structure) + Phase 3 (daily notes para o coach ler histórico)

**Requirements:** COACH-01, COACH-02, COACH-03, COACH-04, COACH-05, COACH-06, COACH-07, COACH-08, COACH-09

**Success Criteria** (what must be TRUE):
  1. Usuário digita `/prepara meu dia` e coach executa protocolo 6 passos: lê ontem + semana + metas, propõe 3 prioridades com justificativa ancorada em dados, antes do usuário preencher
  2. Usuário pede "me sugere algo baseado em livro" e coach apenas cita obra cujo arquivo existe em `08 - Livros/` com `lido: true` — output inclui filesystem path da nota; se não existir, responde "não tenho nota de livro sobre isso"
  3. Usuário digita `/loga revisar relatório 45min` e coach registra em <5s sem fazer pergunta alguma (modo capture detectado)
  4. Usuário corrige estimativa do coach ("na verdade levou 45min, não 2h") e correção é escrita em `_meta/calibration.md` — na próxima estimativa da mesma tarefa, coach usa o novo dado
  5. Usuário roda `verifica fontes` em análise do coach e recebe re-confirmação de cada citação (path + quote presente). Coach com <3 amostras responde explicitamente "sem histórico suficiente", não chuta. Export público nunca contém conteúdo de `_private/`

**Plans:** TBD

---

### Phase 5: Análise Semanal/Mensal

**Goal:** Coach detecta padrões comportamentais reais (score inflation, performative completion) e entrega análises fundamentadas semanais/mensais sem encher de achismo, após ≥2 semanas de dados reais.

**Depends on:** Phase 4 (coach + grounding) + ≥14 dias de daily notes reais via uso diário

**Requirements:** ANAL-01, ANAL-02, ANAL-03, ANAL-04, ANAL-05, ANAL-06, ANAL-07, ANAL-08, ANAL-09

**Success Criteria** (what must be TRUE):
  1. Usuário roda `coach semanal` no domingo e recebe benchmark fiel ao planner (8 perguntas + 3 focos próxima semana) com scores 5 pilares agregados dos 7 daily notes
  2. Coach detecta quando 3+ prioridades do dia são string-match idênticas dos dias anteriores e faz pergunta sem julgamento ("são realmente os mesmos ou é copy-paste?")
  3. Coach detecta quando std dev de scores em 7 dias < 0.5 e questiona explicitamente "rubric drift ou crescimento real?"
  4. Usuário roda `coach mensal` e recebe: evolução mês vs mês anterior, progresso % concreto das Metas 6m/1a (baseado em entregas, não sensação), questão "essa meta ainda faz sentido?"
  5. Coach mostra weekly calibration review: "esta semana você acertou 5 estimativas, subestimou 2 (escrita), superestimou 1" — com diferença quantificada por tipo de tarefa

**Plans:** TBD

---

### Phase 6: Extras (A-J) + Hardening

**Goal:** 10 extras aprovados do brainstorm implementados após MVP validado em uso real, mais hardening (annual method review, performance audit com 365+ notas, archive flow) para que sistema sobreviva anos.

**Depends on:** Phase 5 (dados reais suficientes para time-blocking automático e "você vs você do ano passado")

**Requirements:** EXTRA-A, EXTRA-B, EXTRA-C, EXTRA-D, EXTRA-E, EXTRA-F, EXTRA-G, EXTRA-H, EXTRA-I, EXTRA-J

**Success Criteria** (what must be TRUE):
  1. Usuário roda `/fecha-dia` ao final do dia, coach faz pergunta socrática ancorada em livro se algum pilar ≤ 2, e entrega frase de encerramento — ritual fecha com reflexão, não com métrica
  2. Toda segunda, usuário recebe 1 pergunta da semana distinta (rotativa ~30) para refletir ao longo dos 7 dias; perguntas "dark" de antifragilidade aparecem 1x/mês
  3. Usuário declara Modo Temporada 90d e coach prioriza foco da temporada nas análises semanais; ao fim, entrega retrospectiva especial. Dashboard `09 - Pessoas/` lista pessoas + última vez que falou com cada
  4. Usuário clica "exportar benchmark mensal" e recebe PDF formatado em `04 - Benchmarks/PDF/` com scores + temas agregados, SEM raw quotes de reflexões (privacy redact aplicado)
  5. Usuário abre dashboard com 365+ daily notes acumuladas e Home.md ainda renderiza em <1s cold; annual method review roda e coach pergunta sobre campos não preenchidos nos últimos 90 dias; archive flow move daily > 2 anos para `Arquivo/`

**Plans:** TBD

---

## Progress Table

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Fundação | 0/? | Not started | - |
| 2. Templates + Schema + Rubric | 0/? | Not started | - |
| 3. Dashboard + Queries + Tracking | 0/? | Not started | - |
| 4. Coach + Anti-Alucinação | 0/? | Not started | - |
| 5. Análise Semanal/Mensal | 0/? | Not started | - |
| 6. Extras (A-J) + Hardening | 0/? | Not started | - |

---

## Coverage Summary

- v1 requirements: **72 total**
- Mapped to phases: **72**
- Orphaned: **0** ✓

| Phase | Requirement Count | Requirement IDs |
|-------|-------------------|-----------------|
| Phase 1 | 8 | FOUND-01 → FOUND-08 |
| Phase 2 | 22 | TMPL-01 → TMPL-09, RTN-01 → RTN-05, FRASE-01 → FRASE-04, LIVR-01 → LIVR-04 |
| Phase 3 | 14 | DASH-01 → DASH-09, TRACK-01 → TRACK-05 |
| Phase 4 | 9 | COACH-01 → COACH-09 |
| Phase 5 | 9 | ANAL-01 → ANAL-09 |
| Phase 6 | 10 | EXTRA-A → EXTRA-J |
| **Total** | **72** | |

---

## Key Architectural Constraints (from research)

Estes constraints derivados da pesquisa aplicam-se durante o planning de cada phase:

- **Phase 1:** Vault em path local (NÃO cloud sync — iCloud Windows duplica até 56x); GitHub repo **privado** (verificar duas vezes); Registry `LongPathsEnabled=1` obrigatório; naming convention = filenames curtos.
- **Phase 2:** Schema `snake_case` PT-BR congelado em `_meta/schema.md` ANTES de qualquer captura real; daily frontmatter = single source of truth; weekly/monthly NUNCA duplicam scores; ISO week `GGGG-[W]WW` (não `YYYY-WW`).
- **Phase 3:** Toda query Dataview **sempre** path-scoped + LIMIT (unscoped é #1 causa de freeze); dashboard decomposto (Home leve + Semana/Mês pesados — Home único com 6+ blocos engasga após 1k notas); queries empty-state-safe (Day 0 não renderiza erro).
- **Phase 4:** Anti-alucinação **ESTRUTURAL** (Glob+Read em `08 - Livros/`, não só prompt — LLMs alucinam 20%+ mesmo com regra prompt); detecção de modo deep vs capture via intent (<10 palavras + verbo+objeto = capture); friction budget 5 perguntas/dia; privacy redaction em exports.
- **Phase 5:** Detecção de performative completion = risco comportamental #1 (não abandono, é score inflation + prioridades copy-paste); honestidade check amostra aleatória 1 dia/semana.
- **Phase 6:** Validar MVP sobrevive uso diário ANTES de adicionar extras; EXTRA-I só funciona mês 13+; EXTRA-J só após 3 meses de tracking calibrado.

---

*Last updated: 2026-04-23 after roadmap creation*
