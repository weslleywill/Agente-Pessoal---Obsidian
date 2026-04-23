# Project Research Summary

**Project:** Agente Pessoal Obsidian (Planner Joel Jota + Coach IA)
**Domain:** Personal productivity / PKM / planner digital + AI coach, dentro de Obsidian (Windows 11, vault permanente longevo)
**Researched:** 2026-04-23
**Confidence:** HIGH (stack, architecture, pitfalls técnicos) / HIGH (framework Joel Jota, verificado nas imagens do planner físico) / MEDIUM (UX/behavioral — princípios sólidos, thresholds aproximados)

## Executive Summary

**Planner digital fiel ao Joel Jota em Obsidian + coach IA anti-achismo com fundamentação estrutural em livros**, construído em 6 fases dependency-driven sobre stack Templater/Dataview/Periodic Notes/Tasks/Tracker com risk frontload de sync/privacy/plugins em Phase 1.

O sistema combina três camadas: (1) **captura estruturada** (daily/weekly/monthly via Periodic Notes + Templater), (2) **agregação derivada** (Dataview + Tracker fazendo rollup sobre frontmatter dos diários — daily é single source of truth), e (3) **coaching anti-achismo** (protocolo socrático de 6 passos, fundamentado em livros reais em `08 - Livros/` via filesystem check, não só prompt).

## Key Findings

### Stack (HIGH confidence)

- **Templater 2.19.3** (SilentVoid13, ativíssimo)
- **Dataview 0.5.70** (maintenance-only mas necessário — Bases não cobre DataviewJS/inline)
- **Periodic Notes 1.0.0-beta.3 + Calendar 2.0.0-beta.2** (liamcain, abandonados mas padrão de facto). Plan B: Periodic Notes Calendar (luiisca)
- **Tasks 7.23.1** (ativo) — cuidado: `pomodoros::` ANTES dos emojis Tasks
- **Homepage 4.4.0** (ativo)
- **Tracker 1.19.0** (pyrochlore, ativo) — **vence Charts com folga** (Charts 2 anos sem release; Tracker desenhado para frontmatter → chart)
- **Pomodoro Timer 1.2.3** (eatgrass, limbo). Plan B: TaskNotes
- **NLDates 0.6.2** (abandonado funcional). Plan B: nldates-revived

**4 de 10 plugins stale/limbo** — todos com rota de migração; auditoria periódica obrigatória.

### Features (HIGH confidence — framework Joel Jota verificado)

**Framework Joel Jota (verificado nas imagens do planner físico):**
- **5 Pilares da Alta Performance**: Competitividade, Talento, Saúde, Presença, Performance Emocional — cada um com 5 auto-avaliações 0-5 no Semanário
- **10 Categorias da Auto Performance**: Saúde, Família, Trabalho, Dinheiro, Conhecimento, Mente/Emoção, Relacionamentos, Amor/Parceiro, Tempo, Lazer
- **Manhã Intencional**: 10 perguntas canônicas + 3 prioridades fixas
- **Resumo do Dia**: 6 reflexões + avaliação dos 5 pilares

**⚠️ Correção crítica de autoria** (anti-alucinação interna):
- "Cabeça de Campeão" é de **François Ducasse**, NÃO Joel Jota (Joel fez apenas prefácio da edição brasileira, via Citadel Editora)
- "Pentagrama da Alta Performance" não foi encontrado no planner físico — tratar como narrativa auxiliar até confirmação
- Livros seguros do Joel: "100% Presente", "Padrões de Alta Performance", "Ultracorajoso", "O Sucesso é Treinável"

**Table stakes (must have):** Daily/Weekly/Monthly em estrutura hierárquica; templates fiéis; 3 prioridades fixas; revisão diária/semanal; time-blocking manhã/tarde/noite; calendário clicável; dashboard Home; banco de frases.

**Killer differentiators:** Coach socrático anti-achismo (6 passos); `08 - Livros/` como base; tracking duplo com calibração após 15+ amostras; análise semanal/mensal fundamentada; rotinas via Ativa.md; "prepara meu dia"; ritual fechamento; rodas duplas Alta×Auto (radar); perguntas "dark"; dashboard de pessoas.

**Defer (v2+):** "Você vs você do ano passado" (só mês 13+); time-blocking automático (após 3 meses tracking); wearable CSV; Modo Temporada 90d; export PDF.

**Anti-features (não construir):** Gamificação/streaks/pontos (conflita com anti-achismo); social; AI motivação genérica (toxic positivity); livros não-lidos (gera alucinação); notificações push; AI que toma decisões.

### Architecture (HIGH confidence)

**4 camadas:** Referência → Data (daily frontmatter = single source of truth) → Aggregation (weekly/monthly = Dataview views, NUNCA duplicam) → Presentation (Home + sub-dashboards).

**Componentes principais:**
1. `Templates/` com `_fragments/` e `_config/` — DRY via `tp.file.include()`
2. `01 - Diário/YYYY/MM-Month/YYYY-MM-DD.md` — source of truth; frontmatter com pilares, auto_performance, humor, energia, sono_h, pomodoros
3. `02 - Semanal/` + `03 - Mensal/` — apenas reflexão humana em frontmatter; scores via Dataview `FROM "01 - Diário/YYYY"`. **Zero duplicação.**
4. **Dashboard decomposto**: `Home.md` (leve) + `Semana-Atual.md` + `Mes-Atual.md` (pesados sob demanda). Home.md único com 6+ blocos engasga após 1k notas.
5. `05 - Rotinas/Ativa.md` — pointer `![[Rotina-Dia-Util]]`. Trocar = 1 linha. Não dentro de callouts (bug documentado)
6. `08 - Livros/` — 1 nota por livro com frontmatter `autor:`, citações + página. Coach Glob + Read obrigatório
7. `04 - Benchmarks/Arquivo/` — snapshots MD frozen (write-once) para "você vs você do ano passado"

**Regras não-negociáveis:**
- Queries **sempre path-scoped** (`FROM "01 - Diário/2026"`) — unscoped = #1 causa de freeze
- Frontmatter > inline fields — migra limpo para Datacore/Bases
- ISO week `GGGG-[W]WW` (não `YYYY-WW`) — bug Jan 1
- Max 3 níveis de folder
- Empty-state-safe queries — Day 0 não pode renderizar erro

### Pitfalls críticos (top 5)

1. **Coach hallucinates book citations** — LLMs 20%+ taxa mesmo com prompt rule. **Mitigação estrutural:** Glob + Read em `08 - Livros/`, output inclui filesystem path. Regra em prompt sozinha não enforça nada.
2. **Sync corruption OneDrive/iCloud/Dropbox** — iCloud Windows duplica até 56x. Vault NUNCA em pasta sincronizada; backup via obsidian-git → GitHub **privado** apenas.
3. **Dataview query bloat após ~1k notas** — `FROM ""` freezes 2-5s. Scope + LIMIT sempre. Benchmark <1s cold open.
4. **Score inflation + performative completion** — risco comportamental #1. Tudo drift para 4-5/5; prioridades copy-paste. Coach detecta std dev <0.5 em 7+ dias; detecta copy-paste exato; honestidade check semanal.
5. **Plugin abandonment** — 4 de 10 stale. `_meta/plugins.md` com fallback; workflow degrada graciosamente; plugin config em git.

**Outros pitfalls** (mapeados em PITFALLS.md): schema drift, Windows MAX_PATH 260 chars, graph hairball, Tasks freeze, privacy leak em exports, method dilution, over-questioning em captura, no feedback loop em estimativas, template bloat → abandonment 30d.

## Implications for Roadmap

**6 fases** (granularidade Coarse, dependency-driven):

### Phase 1 — Fundação (Storage + Plugins + Windows Setup + Backup)
**Mais pesada do que o plano inicial sugere — 6 pitfalls aqui.**
- Vault em path curto local (NÃO cloud sync); Registry `LongPathsEnabled=1`
- obsidian-git → GitHub **privado** (verificar duas vezes)
- `.gitignore` `workspace.json` + `_private/`
- 10 plugins instalados; Templater "Trigger on new file = ON"
- `_meta/plugins.md` com last-commit + fallback por plugin
- `_meta/setup.md`, skeleton de folders, `_private/` zone
- Naming conventions (filenames curtos)
- **Addresses:** pitfalls #3, #5, #11, #14, parcial #1

### Phase 2 — Templates + Schema + Rubric (Fiel ao Planner Físico)
- `_meta/schema.md` (snake_case PT-BR congelado)
- `_meta/rubric.md` (âncoras concretas 1-5 por pilar)
- `_meta/fields.md` (justificativa por campo)
- Templates verbatim do planner físico
- `_fragments/` reutilizáveis
- `05 - Rotinas/` povoado
- Periodic Notes configurado (`YYYY/MM-MMMM/YYYY-MM-DD`, `GGGG/GGGG-[W]WW`)
- `06 - Metas/`, `07 - Frases/` (~50 seed), `08 - Livros/` (3 seed — com `autor:` correto)
- **Addresses:** #2, #6, #7, #15

### Phase 3 — Dashboard + Queries + Tracking (Sintéticos primeiro)
- `Home.md` **leve** (saudação + frase + HOJE + Ativa + links — zero Tracker aqui)
- `Semana-Atual.md` + `Mes-Atual.md` pesados
- Homepage → `Home.md` Reading mode
- `INDEX.md` em 06/08/09/10
- Query audit (todas scope + LIMIT; benchmark <1s)
- Pomodoro Timer valida integração Tasks
- Teste end-to-end com 3 daily sintéticas
- **Addresses:** #1, #12, #13, parcial #6

### Phase 4 — Coach de Priorização + Anti-Alucinação Estrutural
**Killer feature + maior risco.**
- System prompt com 6 passos socráticos
- **Regra estrutural** (Glob + Read `08 - Livros/` obrigatório; output com filesystem path)
- Comando `verifica fontes`
- **2 modos** (`/prepara` vs `log` <5s zero perguntas)
- Friction budget 5 Q/dia
- Comando "prepara meu dia"
- **Feedback loop** (correções em `_meta/calibration.md`)
- Regra "<3 amostras = sem histórico"
- **Privacy redaction** (nunca cita `_private/` em exports)
- Teste adversarial
- **Addresses:** #4, #9, #10, #14

### Phase 5 — Análise Semanal + Mensal (Detecção de Padrões)
- Análise semanal (domingo)
- Análise mensal
- **Rodas duplas Alta × Auto (radar)**
- **Detecção performative completion** (string match + repetição)
- **Detecção score inflation** (std dev <0.5 em 7d)
- **Honestidade check** (amostra aleatória)
- **Calibration review semanal**
- Pergunta semana rotativa + perguntas "dark"
- **Addresses:** #7, #8, #10, #14

### Phase 6 — Extras + Hardening + Scaling Prep
- Modo Temporada 90d (C), Dashboard pessoas (F), Export PDF + redact (G)
- Wearable CSV manual (D), Ritual fechamento + pergunta rotativa (A)
- "Você vs você ano passado" (I — usa Arquivo, não re-query live)
- Time-blocking automático (J — após 3 meses)
- **Annual method review** (#15)
- Archive flow (daily >2 anos → `Arquivo/`)
- Plugin health re-audit
- Performance audit com 365+ notes
- **17-item "Looks Done But Isn't" checklist**

### Research flags (fases que precisam pesquisa adicional)

**Precisa `/gsd-research-phase` durante planning:**
- **Phase 4 (Coach):** implementação detalhada do grounding workflow
- **Phase 5 (Análise):** algoritmos de pattern detection (thresholds)
- **Phase 6 (Extras):** export PDF, time-blocking algoritmo, wearable CSV

**Padrões bem-documentados (skip):**
- **Phase 1:** Windows + git (caminhos trilhados)
- **Phase 2:** schema + templates (padrão Obsidian)
- **Phase 3:** Dashboard + Dataview + Tracker (patterns consolidados)

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Plugins + versões verificados em GitHub releases 2026-04-23 |
| Features | HIGH | Framework Joel Jota verificado direto nas imagens do planner físico; correção de autoria "Cabeça de Campeão" = Ducasse |
| Architecture | HIGH | Vaults de referência (Kepano/Steph Ango CEO); thresholds específicos (1500, 5000 notas) |
| Pitfalls técnicos | HIGH | Verificados em Forum + GitHub issues com issue numbers |
| Pitfalls comportamentais | MEDIUM | Princípios consensuais mas thresholds aproximados |

**Overall confidence: HIGH.**

## Gaps to Address

- **Pentagrama Joel Jota** — não consta no planner físico. Confirmar nos livros ou descartar como narrativa de palestras
- **3Ps (Propósito, Processo, Desempenho)** — fontes secundárias; confirmar ou descartar
- **Threshold exato copy-paste detection** — decisão em Phase 5
- **Pomodoro Timer vs TaskNotes final** — decisão em Phase 1 plugin audit
- **Bases vs Dataview em 2-3 anos** — monitorar; migration plan parcial Phase 6+
- **Calibração empírica thresholds comportamentais** — só uso real do Weslley valida

**Lesson learned meta:** "Cabeça de Campeão = Joel Jota" passou como assumption; research descobriu Ducasse. **Valida o padrão anti-alucinação estrutural — regras de prompt não bastam, check de filesystem é o gate real.**

## Sources

### Primary (HIGH — verificação direta)
- Imagens do planner físico em `E:\Claude Code\Agente Pessoal - Obsidian\Imagens Planner\*.jpeg`
- Brief do usuário em `.planning/PROJECT.md`
- GitHub releases oficiais dos 10 plugins (2026-04-23)
- Obsidian 1.9.0 changelog (Bases core plugin)
- Steph Ango / PARAZETTEL (folder philosophy)
- Microsoft Learn — MAX_PATH limitation
- Obsidian Forum issues (perf, Tasks freeze, iCloud corruption, MAX_PATH, callout+embed bug)
- arxiv — AI-hallucinated citations (2025); UNMC Library
- Harvard Kennedy School — Self-ratings bias

### Secondary (MEDIUM)
- Joel Jota Store / Poder da Disciplina / Facebook Joel Jota
- Citadel Editora (confirma Ducasse)
- Brendon Burchard HP6
- Frontiers in Psychology (2020)
- Full Focus / Notion 2026 Planner / Silk+Sonder
- obsidianstats.com
- Trophy / Sudo Science / Bullet Journal / Mobile Coach — anti-pattern literature

### Tertiary (LOW — precisam validação em uso)
- Thresholds comportamentais específicos
- Pentagrama Joel Jota (não encontrado)
- 3Ps (fontes secundárias)
