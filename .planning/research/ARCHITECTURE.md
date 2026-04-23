# Architecture Research

**Domain:** Personal high-performance planner vault in Obsidian (PKM + temporal journal + dashboard)
**Researched:** 2026-04-23
**Confidence:** HIGH (structure/data flow) · MEDIUM (dashboard rendering choice — depends on final plugin versions)

---

## Executive Summary

The proposed vault structure (`00-Dashboard`, `01-Diário/YYYY/MM`, `02-Semanal/YYYY`, `03-Mensal/YYYY`, `04-Benchmarks`, `05-Rotinas`, `06-Metas`, `07-Frases`, `08-Livros`, `09-Pessoas`, `10-Projetos`, `Templates/`) is **architecturally sound** and aligns with real-world Obsidian planner vaults. It uses numbered prefixes (ordering), date-hierarchical folders (scales to decades), and a dedicated `Templates/` folder (Templater convention).

**Three architectural concerns require attention:**

1. **Dashboard performance ceiling** — Stacking many Dataview + DataviewJS + Tracker blocks in a single `Home.md` will load slowly (documented 1-3s+ on vaults >500 notes, degrading further over time). Mitigation: segregate heavy queries into sub-dashboards, use `FROM "01 - Diário"` path scoping, limit queries to rolling windows (last 30d / current week).
2. **Data-flow coupling** — Weekly/monthly notes should be **derived views** over daily frontmatter, not independent data stores. Duplicating scores into weekly frontmatter creates drift. Mitigation: weekly note is a *benchmark/reflection* note where Dataview aggregates daily scores live; only human-written reflection fields live in weekly frontmatter.
3. **Empty-state fragility** — Dataview errors silently when no notes match. Templates must render gracefully at Day 0 (no daily notes yet). Mitigation: wrap every Dataview block with a fallback message using `WHERE length(rows) > 0` or DataviewJS conditional.

**Build order is dependency-driven:** Templates/ must exist before Periodic Notes is configured; Periodic Notes must exist before first daily note; daily notes must accumulate before weekly aggregation has meaning; dashboard is the LAST thing built (after all data sources produce real values).

---

## Standard Architecture

### System Overview — Layered View

```
┌──────────────────────────────────────────────────────────────────────┐
│                      PRESENTATION LAYER (Views)                       │
├──────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ 00-Dashboard │  │ 09-Pessoas/  │  │ 10-Projetos/ │  │ Graph    │ │
│  │   Home.md    │  │ (dashboard)  │  │ (kanban)     │  │ View     │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └──────────┘ │
│         │ Dataview/DataviewJS/Tracker queries                         │
├─────────┴──────────────────┴──────────────────┴──────────────────────┤
│                       AGGREGATION LAYER (Derived)                     │
├──────────────────────────────────────────────────────────────────────┤
│  ┌────────────────────┐    ┌────────────────────┐                    │
│  │ 02-Semanal/YYYY/   │    │ 03-Mensal/YYYY/    │                    │
│  │ YYYY-Www.md        │    │ YYYY-MM.md         │                    │
│  │ (reflects 7 daily) │    │ (reflects 4-5 wk)  │                    │
│  └─────────┬──────────┘    └─────────┬──────────┘                    │
│            │ reads daily frontmatter  │                              │
├────────────┴──────────────────────────┴──────────────────────────────┤
│                         DATA LAYER (Source of Truth)                  │
├──────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────┐    ┌──────────────────────┐             │
│  │ 01-Diário/YYYY/MM-MMM/  │    │ 06-Metas/            │             │
│  │ YYYY-MM-DD.md           │    │ (6m / 1a / 90d)      │             │
│  │ frontmatter: pilares,   │    │ frontmatter: targets │             │
│  │ scores, pomodoros, tags │    │                      │             │
│  └─────────────────────────┘    └──────────────────────┘             │
├──────────────────────────────────────────────────────────────────────┤
│                      REFERENCE LAYER (Static / Input)                 │
├──────────────────────────────────────────────────────────────────────┤
│  ┌───────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────┐ │
│  │ Templates │ │ 05-Rotin.│ │ 07-Frases│ │ 08-Livros│ │ 04-Bench. │ │
│  │ (.md)     │ │ Ativa→RX │ │ (rotação)│ │ (coach)  │ │ (exports) │ │
│  └───────────┘ └──────────┘ └──────────┘ └──────────┘ └───────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| `Templates/` | Template definitions for daily/weekly/monthly + fragmentos reutilizáveis (Manhã Intencional, Benchmark, etc.) | Templater files with dynamic YAML frontmatter and Tp.* calls |
| `01 - Diário/YYYY/MM-Month/` | **Source of truth** for all daily data (scores, pomodoros, tasks, reflection) | Periodic Notes plugin + Templater; frontmatter is queryable |
| `02 - Semanal/YYYY/` | Weekly **benchmark + reflection**. Aggregates daily via Dataview; adds qualitative reflection fields | Periodic Notes (Weekly) with Dataview-driven body |
| `03 - Mensal/YYYY/` | Monthly benchmark; evolution vs. previous; progress toward metas | Periodic Notes (Monthly) with Dataview aggregates over 4-5 weekly notes or 28-31 daily notes |
| `04 - Benchmarks/` | Exported/archived snapshots (PDF, frozen MD) — write-once artifacts | Manual export via print/Obsidian-to-PDF; enables "you vs you last year" comparison |
| `05 - Rotinas/` | Rotinas fixas (Dia-Util, Fim-Semana) + `Ativa.md` ponteiro | Regular MD files; `Ativa.md` contains only `![[Rotina-Dia-Util]]` — changes centralizadas em 1 edit |
| `06 - Metas/` | 6-meses, 1-ano, 90-dias metas com frontmatter de targets | MD files; queried by daily/weekly coach prompts |
| `07 - Frases/` | Banco rotativo; single file with frase[] array OR one-frase-per-file | DataviewJS picks index = `dayOfYear % n` for deterministic rotation |
| `08 - Livros/` | Notas de livros como **referência anti-alucinação** do coach | MD files with `#livro` tag + structured notes |
| `09 - Pessoas/` | Relationship dashboard; one file per pessoa + index view | Dataview index of `#pessoa` notes with last-contact metadata |
| `10 - Projetos/` | Projetos pessoais (não o vault project em `.planning/`) | Optionally PARA-like sub-structure |
| `00 - Dashboard/Home.md` | Ponto de entrada; Homepage plugin aponta aqui | MD + embedded Dataview blocks + `![[Ativa]]` transclusion |

---

## Recommended Project Structure

### Validated Folder Layout (with corrections to the proposal)

```
Vault/
├── 00 - Dashboard/
│   ├── Home.md                      # Homepage plugin target
│   ├── Semana-Atual.md              # Secondary dashboard (heavy queries isolated)
│   └── Mes-Atual.md                 # Secondary dashboard
├── 01 - Diário/
│   └── YYYY/                        # e.g. 2026/
│       └── MM-Month/                # e.g. 04-Abril/
│           └── YYYY-MM-DD.md        # e.g. 2026-04-23.md
├── 02 - Semanal/
│   └── YYYY/
│       └── YYYY-Www.md              # e.g. 2026-W17.md  (ISO week)
├── 03 - Mensal/
│   └── YYYY/
│       └── YYYY-MM.md               # e.g. 2026-04.md
├── 04 - Benchmarks/
│   ├── PDF/                         # exported snapshots
│   └── Arquivo/                     # frozen MD copies (anti-drift)
├── 05 - Rotinas/
│   ├── Ativa.md                     # ponteiro: contém ![[Rotina-Dia-Util]]
│   ├── Rotina-Dia-Util.md
│   ├── Rotina-Fim-Semana.md
│   └── Rotina-Viagem.md             # (futuro)
├── 06 - Metas/
│   ├── Metas-2026-S1.md             # 6 meses
│   ├── Metas-2026.md                # 1 ano
│   ├── Temporada-Q2-2026.md         # 90d (extra C)
│   └── INDEX.md                     # Dataview list + progress
├── 07 - Frases/
│   ├── Banco-Frases.md              # Um arquivo único com lista (mais simples p/ rotação)
│   └── Pergunta-Semana.md           # Extra do brainstorm
├── 08 - Livros/
│   ├── INDEX.md
│   └── [Autor] - [Título].md        # convenção de nome
├── 09 - Pessoas/
│   ├── INDEX.md                     # dashboard de relacionamentos
│   └── [Nome].md
├── 10 - Projetos/
│   ├── INDEX.md
│   └── [Projeto]/
└── Templates/
    ├── Daily.md
    ├── Weekly.md
    ├── Monthly.md
    ├── Pessoa.md
    ├── Livro.md
    ├── Projeto.md
    ├── Meta.md
    ├── _fragments/                  # Reutilizáveis (sublinhado = ignorado por graph)
    │   ├── frag-manha-intencional.md
    │   ├── frag-benchmark-diario.md
    │   └── frag-5-pilares.md
    └── _config/
        └── paths.md                 # constantes (paths, IDs) referenciadas por Templater
```

### Structure Rationale & Deviations from Proposal

**What the proposal got right:**

- **Numbered prefixes** (`00 - ... 10 - ...`) — Valid Obsidian pattern for controlling file-explorer order. Kepano himself uses zero-padding. Does NOT affect Dataview/search; is purely presentation.
- **Date-hierarchical folders** (`YYYY/MM-Month/`) — Scales indefinitely. Periodic Notes plugin supports folder-format in Date Format field (e.g., `YYYY/MM-MMMM/YYYY-MM-DD`). A decade of daily notes = ~3650 files spread across 10 yearly folders × 12 monthly folders = no single folder has >31 notes. Obsidian file explorer performance is fine at this scale.
- **Separate `Templates/` folder** — Convention. Templater setting points to this folder.
- **Ponteiro `Ativa.md`** — Valid. `![[Rotina-Dia-Util]]` embed renders live. Change by editing `Ativa.md` to `![[Rotina-Fim-Semana]]`. See Pattern 3 below for caveats.

**Deviations recommended:**

1. **Add `00 - Dashboard/Semana-Atual.md` and `Mes-Atual.md`** as sub-dashboards. Rationale: a single `Home.md` with 6+ heavy Dataview/Tracker blocks becomes slow. Split: `Home.md` keeps lightweight blocks (saudação, frase, hoje, calendar embed, link to sub-dashboards). Heavy charts live in `Semana-Atual.md` and `Mes-Atual.md`, opened on-demand.

2. **Use `MM-Month/` naming (e.g., `04-Abril/`)** instead of pure `MM/`. Rationale: when browsing the file tree manually, month names are scannable. Dataview queries still match on filename regex or use `dateformat(file.day, "yyyy-MM")`. Cost: one extra moment.js token in date format.

3. **ISO week numbering (`YYYY-Www`) over `YYYY-WW`** — moment.js uses `GGGG-[W]WW` for ISO; prevents edge-case where Dec 31 belongs to Week 01 of next year. The `GGGG` (not `YYYY`) correctly yields ISO-week-year.

4. **Add `INDEX.md` in `06-Metas/`, `08-Livros/`, `09-Pessoas/`, `10-Projetos/`** — Dataview table view listing contents. Prevents "I have 40 livros, which one was that quote from?" friction.

5. **Sub-folder `Templates/_fragments/`** with underscore prefix — Obsidian ignores `_`-prefixed items from graph view by default (and most users filter them from search). Use for reusable partials (Manhã Intencional, 5 Pilares block) that are Templater-included (`<%* tp.file.include("[[frag-manha-intencional]]") %>`) by main templates. DRY.

6. **Banco-Frases.md as single file with list** — easier rotation than 365 files (`DayOfYear mod length`). Use:
   ```markdown
   ---
   tipo: banco-frases
   ---
   - "Frase 1" — Autor
   - "Frase 2" — Autor
   ```
   DataviewJS in Home.md picks index = `moment().dayOfYear() % frases.length`.

---

## Architectural Patterns

### Pattern 1: Source-of-Truth Frontmatter (Daily as Canonical)

**What:** All quantitative data lives in daily-note frontmatter. Weekly/monthly notes **aggregate via queries**, never duplicate.

**When to use:** Any metric that is measured daily (pilar scores, pomodoros, humor, energia).

**Trade-offs:**
- Pro: Single source of truth. No drift. Editing a Monday score retroactively updates weekly and monthly views automatically.
- Pro: Weekly/monthly can be regenerated if corrupted.
- Con: Weekly view depends on 7 file reads — but Dataview caches indexed metadata, so this is cheap (<10ms).

**Example — Daily frontmatter (Templater-generated):**

```yaml
---
data: 2026-04-23
semana: 2026-W17
mes: 2026-04
tipo: diario
pilares:
  competitividade: 7
  talento: 6
  saude: 8
  presenca: 7
  performance_emocional: 6
pilar_media: 6.8      # optional: pre-computed to avoid Dataview math every read
auto_performance:
  saude: 7
  familia: 8
  trabalho: 6
  dinheiro: 5
  conhecimento: 7
  mente: 6
  relacionamentos: 8
  amor: 7
  tempo: 5
  lazer: 6
humor: 7
energia: 8
sono_h: 7.5
pomodoros: 0          # atualizado durante o dia
prioridades_feitas: 0 # de 3
tags: [diario]
---
```

**Example — Weekly aggregation (Dataview body):**

```dataview
TABLE
  round(avg(pilares.competitividade), 1) as "Comp",
  round(avg(pilares.talento), 1) as "Tal",
  round(avg(pilares.saude), 1) as "Sau",
  round(avg(pilares.presenca), 1) as "Pre",
  round(avg(pilares.performance_emocional), 1) as "PE"
FROM "01 - Diário"
WHERE semana = this.file.name
```

### Pattern 2: Scoped Dataview Queries (FROM "path")

**What:** Every Dataview block specifies a folder path, never queries the whole vault.

**When to use:** ALWAYS, in every query. Non-negotiable for performance.

**Trade-offs:**
- Pro: Limits index scan. A 3000-note vault with a `FROM "01 - Diário/2026"` query only touches ~365 notes.
- Con: Slightly more typing.

**Example:**

```dataview
TABLE data, pilar_media
FROM "01 - Diário/2026"      ← folder-scoped
WHERE mes = "2026-04"        ← month-scoped
SORT data DESC
LIMIT 31
```

**Anti-example (SLOW — do not write):**

```dataview
TABLE pilar_media FROM "" WHERE tipo = "diario"   ← scans ENTIRE vault
```

### Pattern 3: Pointer File via Transclusion (Ativa.md)

**What:** `05 - Rotinas/Ativa.md` contains a single embed line. Other views (Home.md, daily template) reference `![[Ativa]]`, which resolves transitively.

**When to use:** Swappable "active" configuration — rotinas, modo temporada, frase do dia.

**Trade-offs:**
- Pro: Single edit point changes behavior globally.
- Con: **Transitive embeds (embed inside embed) work in Obsidian but have rendering edge cases inside callouts and can be slower in Live Preview.** Two forum-documented bugs: transclusion inside a `> [!note]` callout sometimes fails to update until refresh.
- Con: Not Templater-time substitution — Ativa.md's content is resolved at render time. If a daily template needs the rotina baked-in (frozen copy that day), use Templater `tp.file.include("[[Ativa]]")` instead of `![[Ativa]]`.

**Example — Ativa.md:**

```markdown
---
tipo: ponteiro-rotina
alvo: Rotina-Dia-Util
atualizado: 2026-04-23
---

![[Rotina-Dia-Util]]
```

**Example — Home.md consuming it:**

```markdown
## Rotina Ativa
![[Ativa]]
```

**Guidance:** Use `![[Ativa]]` in Home.md (live). Use `tp.file.include("[[Ativa]]")` in the daily template if you want the rotina copied into each day's note at creation (historical preservation).

### Pattern 4: Sub-Dashboards (Dashboard Decomposition)

**What:** Instead of one `Home.md` with all widgets, decompose into `Home.md` (lightweight entrypoint) + `Semana-Atual.md`, `Mes-Atual.md`, `Metas-Dash.md` (heavy views).

**When to use:** When dashboard load time exceeds 1.5s OR when total Dataview/Tracker blocks on one file > 4.

**Trade-offs:**
- Pro: Home opens instantly. Heavy dashboards open only when needed.
- Pro: Isolates performance impact — a bad query in Semana-Atual doesn't freeze Home.
- Con: One extra click to see charts.

**Example — Home.md layout:**

```markdown
# Bom dia, Weslley

> [!quote] Frase do dia
> `$= const f=dv.page("Banco-Frases").file.lists[0].values; const i=moment().dayOfYear()%f.length; f[i].text`

## HOJE — 3 Prioridades
```dataview
TASK
FROM "01 - Diário/2026"
WHERE file.day = date(today) AND contains(tags, "prioridade")
```

## Rotina Ativa
![[Ativa]]

## Acessos rápidos
- [[Semana-Atual]] — pilares da semana, gráficos
- [[Mes-Atual]] — evolução mensal, progresso metas
- [[06 - Metas/INDEX]] — todas as metas
- [[09 - Pessoas/INDEX]] — pessoas que preciso contatar
```

### Pattern 5: Empty-State-Safe Queries

**What:** Every Dataview block wrapped to render gracefully when no data matches (especially Day 0).

**When to use:** All queries on dashboards. Daily template and freshly-created weekly/monthly notes.

**Example — DataviewJS with fallback:**

```dataviewjs
const pages = dv.pages('"01 - Diário/2026"')
  .where(p => p.semana === dv.current().file.name);
if (pages.length === 0) {
  dv.paragraph("_Sem notas diárias nesta semana ainda. Comece criando a primeira._");
} else {
  dv.table(["Data","Média"], pages.map(p => [p.file.link, p.pilar_media]));
}
```

---

## Data Flow

### Capture → Aggregate → Reflect

```
DAILY CAPTURE (user writes)
─────────────────────────────────────
Templater creates 2026-04-23.md with:
  frontmatter template (pilares: null, humor: null, ...)
  body sections (Manhã Intencional, Horários, Benchmark)
       │
       │ User fills during day + fim do dia
       ▼
Frontmatter populated: scores, pomodoros, pilares
       │
       │ Obsidian metadata cache updates
       │ Dataview index refreshes (~100-500ms)
       ▼
WEEKLY AGGREGATION (read on demand)
─────────────────────────────────────
02-Semanal/2026/2026-W17.md opened:
       │
       ▼
Dataview queries FROM "01 - Diário/2026" WHERE semana = "2026-W17"
       │
       ▼
Returns 7 rows → AVG/SUM computed at query time
       │
       ▼
Rendered as tables + user writes reflection fields
       │
       ▼
MONTHLY AGGREGATION (read on demand)
─────────────────────────────────────
03-Mensal/2026/2026-04.md opened:
       │
       ▼
Dataview queries FROM "01 - Diário/2026" WHERE mes = "2026-04"
       │ (OR: FROM "02 - Semanal/2026" WHERE month = 4)
       ▼
Returns 30 daily rows OR 4-5 weekly rows → aggregated
       │
       ▼
Progress vs Metas: JOIN with 06-Metas/* frontmatter targets
       │
       ▼
DASHBOARD VIEW (continuous)
─────────────────────────────────────
00-Dashboard/Home.md auto-opens on vault launch
       │
       ▼
Tracker plugin reads frontmatter across "01 - Diário"
       → renders line chart of pilar_media last 30d
       │
Frase rotation: DataviewJS reads Banco-Frases, picks by dayOfYear
       │
Ativa.md embedded → renders active rotina
```

### Key Data Flows

1. **Score flow (daily → weekly → monthly):** Frontmatter YAML in `YYYY-MM-DD.md` → Dataview AVG in `YYYY-Www.md` → Dataview AVG-of-AVG or AVG-of-daily in `YYYY-MM.md`. **Single source of truth = daily.**

2. **Rotina flow (Ativa → consumers):** `Ativa.md` (pointer) → embeds `Rotina-Dia-Util.md` (content). Consumers reference `![[Ativa]]`. To switch rotinas, edit Ativa.md's single embed line.

3. **Frase flow (banco → rotation):** `07 - Frases/Banco-Frases.md` list → DataviewJS in Home.md picks index deterministically from `dayOfYear()`. Same frase for whole day, different each day.

4. **Meta flow (target → progress):** `06 - Metas/Metas-2026.md` frontmatter has `target: 150000` (ex: revenue). Monthly note pulls `sum(receita)` from daily notes and compares. Shows delta.

5. **Coach flow (prompt → references):** When Claude runs "prepara meu dia", it reads:
   - Last 7 daily notes frontmatter (histórico)
   - `06 - Metas/*` (targets ativos)
   - `08 - Livros/*` (anti-hallucination reference pool)
   - Current `02 - Semanal/YYYY-Www.md` (contexto da semana)

### State Management (Obsidian-Specific)

Obsidian maintains an internal metadata cache (JSON index over frontmatter + headings + links) that Dataview subscribes to. When a daily note's frontmatter changes, the following chain fires:

```
Edit daily note frontmatter
    ↓
Obsidian saves file (debounced ~2s)
    ↓
MetadataCache emits "changed" event
    ↓
Dataview's DataviewIndex updates (subscribes to the event)
    ↓
All open Dataview views re-execute queries
    ↓
Dashboard/weekly views repaint
```

**Implication:** You do not manually "refresh" the weekly summary — it is derived. However, rapid edits during typing can cause flicker in open Dataview views (documented). Close dashboards when doing bulk edits.

---

## Scaling Considerations

This is a **personal vault** — "scale" is measured in years of data, not concurrent users.

| Scale | Vault Size | Architecture Adjustments |
|-------|-----------|--------------------------|
| **Year 1** | ~365 daily + 52 weekly + 12 monthly + ~200 other = ~630 notes | No adjustments. Proposed structure handles this trivially. |
| **Year 3** | ~1100 daily + 156 weekly + 36 monthly + 500 other = ~1800 notes | **First perceptible Dataview slowdown** if queries are unscoped. Path-scoped queries (`FROM "01 - Diário/YYYY"`) remain fast. |
| **Year 5+** | ~1800 daily + 260 weekly + 60 monthly + 1000 other = ~3100 notes | **Dashboard decomposition becomes mandatory.** Archive (or exclude from index) old years via `.obsidianignore` or moving to `04 - Benchmarks/Arquivo/`. Use `dv.pages()` with year scope aggressively. |
| **Year 10** | ~5000+ notes | Consider migration to **Datacore** (Dataview's declared successor) OR **Bases** (Obsidian-native property-based views — faster than Dataview in benchmarks). |

### Scaling Priorities (What Breaks First)

1. **First bottleneck — dashboard load time.** Home.md with 6+ Dataview blocks becomes noticeably slow (>1.5s) around vault size 1500+. **Fix:** decompose into sub-dashboards (Pattern 4); scope queries by year/month path; cache pre-computed aggregates into weekly/monthly frontmatter (`pilar_media_semana`).

2. **Second bottleneck — Tracker chart render.** Tracker plugin scans all files in folder at render time. A 5-year chart touches ~1800 daily notes. **Fix:** limit charts to rolling windows (last 90d / 365d). For "you vs you last year" extras, use `04 - Benchmarks/` frozen snapshots as alternate data source, not live re-query.

3. **Third bottleneck — vault indexing on startup.** Obsidian re-indexes all files on launch. Reported slowdown at 5000+ notes on low-end hardware. **Fix:** `04 - Benchmarks/Arquivo/` can be excluded from indexing via Obsidian settings → Files & Links → Excluded Files.

4. **Fourth bottleneck — Sync conflicts.** If using Obsidian Sync mobile, the `.obsidian/workspace.json` can conflict. **Fix:** add to sync-ignore list.

---

## Anti-Patterns

### Anti-Pattern 1: Duplicating Scores into Weekly Frontmatter

**What people do:** At end of week, manually (or via Templater) copy the 7 daily pilar scores into the weekly note's frontmatter as `pilares_semana:` — creating a second copy.

**Why it's wrong:** Two sources of truth. Edit a Monday score retroactively → weekly duplicate is now stale. Silent drift. Violates anti-alucinação principle (coach reads stale data).

**Do this instead:** Weekly frontmatter contains ONLY human-written reflection fields (`foco_proxima_semana:`, `aprendizado_chave:`). Scores are ALWAYS computed via Dataview from daily notes. If you want a frozen snapshot (e.g., for the "you vs you last year" extra), export to `04 - Benchmarks/Arquivo/` as a write-once MD file.

### Anti-Pattern 2: Unscoped Dataview Queries

**What people do:** Write `FROM "" WHERE tipo = "diario"` or `FROM #daily`.

**Why it's wrong:** Queries scan every note in the vault. Fine at 100 notes, painful at 1500, unusable at 5000. Confirmed in forum threads — this is the #1 cause of reported Dataview slowness.

**Do this instead:** Always use path-scoped `FROM "01 - Diário/2026"` or year-range. Tag-based queries are acceptable only if combined with path: `FROM "01 - Diário" AND #daily`.

### Anti-Pattern 3: Monolithic Home.md with Every Widget

**What people do:** Stack 8+ Dataview blocks, 3 Tracker charts, a DataviewJS pomodoro widget, and a calendar embed in a single Home.md.

**Why it's wrong:** Every Dataview block is its own query execution on every file-open. Cumulative load > 3s on medium vaults. Makes the "lightweight daily entry point" slow.

**Do this instead:** Pattern 4 (Sub-Dashboards). Home = lightweight (frase, HOJE, Ativa, links). Heavy views = on-demand sub-dashboards.

### Anti-Pattern 4: Inline Fields Instead of Frontmatter for Queryable Data

**What people do:** Write `pilar_saude:: 7` inline in the body instead of in frontmatter.

**Why it's wrong:** Per Dataview's own maintainer, performance is identical at index time — BUT frontmatter is the **Obsidian-native** metadata system (supported by Properties, Bases, future Datacore). Inline is Dataview-specific legacy. Committing to inline now = migration pain later when Dataview → Datacore/Bases.

**Do this instead:** Frontmatter YAML for all queryable fields. Inline (`::`) only for prose-embedded annotations that truly belong in paragraph context.

### Anti-Pattern 5: Deeply Nested Folders (4+ Levels)

**What people do:** `01 - Diário/2026/Q2/Abril/Semana-17/2026-04-23.md`.

**Why it's wrong:** Kepano (Obsidian CEO) explicitly advocates against deep nesting. Redundant with file metadata (date is in filename). Harder to browse. Periodic Notes plugin doesn't need it.

**Do this instead:** Max 3 levels (`01 - Diário / YYYY / MM-Month / YYYY-MM-DD.md`). Semester/quarter grouping is a Dataview view concern, not a folder concern.

### Anti-Pattern 6: Templates with Hard-Coded Dates

**What people do:** `YYYY-04-23.md` with template containing `"Hoje é 23 de abril"`.

**Why it's wrong:** Template applied tomorrow still says April 23.

**Do this instead:** Templater dynamic: `<% tp.date.now("DD [de] MMMM") %>` evaluated at creation. Always.

### Anti-Pattern 7: Not Handling Empty States

**What people do:** Dashboard query assumes daily notes exist. Day 0 of vault use → Dataview renders empty table, user thinks it's broken.

**Why it's wrong:** Confusing UX. Worse: some queries (`MIN(scores)`) error on empty set and halt the whole block.

**Do this instead:** Pattern 5. Wrap with `if (pages.length === 0)` or `WHERE length(rows) > 0` fallbacks.

---

## Integration Points

### External Services / Plugins

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| **Templater** | Folder templates map folder → template file. Set in Templater settings. | MUST set `Trigger Templater on new file creation` = ON. Otherwise daily notes use plain Periodic Notes templates (no dynamic logic). |
| **Periodic Notes** | Configure Daily/Weekly/Monthly with date format including folder path (`YYYY/MM-MMMM/YYYY-MM-DD`). Point template to Templater file. | ISO week: use `GGGG-[W]WW` for weekly format, not `YYYY-WW` (fixes Dec/Jan edge case). |
| **Dataview** | Code-blocks `\`\`\`dataview` and `\`\`\`dataviewjs` in any MD file. Settings → enable JavaScript queries for DataviewJS. | Enable "Automatic View Refreshing" = ON. Consider "Maximum Recursive Render Depth" = 4 (avoid infinite embed loops). |
| **Calendar** | Sidebar widget; syncs with Periodic Notes (same date format). | Click-to-create daily note. |
| **Tasks** | Parses `- [ ] task` syntax; supports due dates, priorities, recurrence. Queries like `not done`, `due today`. | Combine with Dataview for flexible views. Weekly benchmark can query `done this week`. |
| **Tracker** | Code-block configuration; reads frontmatter/inline/tag data; renders line/bar/calendar heatmap/pie. | Heavier than Dataview for large ranges. Limit to folder-scoped + time-windowed queries. |
| **Homepage** | Points to `00 - Dashboard/Home.md`. Opens on vault launch. | Pure convenience — no perf benefit over manual navigation. |
| **Natural Language Dates** | Parses "@today", "@next friday" into ISO dates. Useful inside daily notes. | Works with Tasks and Dataview. |
| **Pomodoro plugin** | Multiple exist; validate at install time (proposed plugin list in PROJECT.md says "a validar"). | Preferred: one that logs sessions to daily frontmatter (`pomodoros: 4`) so Tracker can chart. |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| `Templates/` ↔ `01 - Diário/` | Templater reads template, writes daily note. One-way. | Templates contain YAML schema; if schema changes, existing daily notes are NOT migrated (breaks queries). Version templates and plan migration. |
| `01 - Diário/` ↔ `02 - Semanal/` | Weekly reads daily via Dataview. One-way. Weekly does not write back. | Derived view only. If you need to override a week's score, add an explicit `override_pilar_media` in weekly frontmatter, and let Dataview prefer override when present. |
| `01 - Diário/` ↔ `00 - Dashboard/` | Dashboard reads daily via Dataview/Tracker. One-way. | Dashboard never mutates daily notes. |
| `05 - Rotinas/Ativa` ↔ consumers | Transclusion (`![[Ativa]]`). Render-time resolution. | One-to-many. All consumers update simultaneously when Ativa changes. |
| `06 - Metas/` ↔ `03 - Mensal/` | Mensal reads metas targets, computes progress. One-way. | If targets change mid-period, old monthly notes show stale comparisons (historically accurate). |
| `.planning/` ↔ `Vault/` | **No communication.** Separate concerns. | `.planning/` = GSD project control. `Vault/` = the product. Never cross-reference. |

---

## Recommended Build Order

Based on dependency graph — each step unblocks the next.

```
Step 1: Install & configure plugins
   └─> Templater, Periodic Notes, Dataview, Calendar, Tasks, Homepage,
       Tracker, Natural Language Dates, Pomodoro (validate choice)

Step 2: Create folder skeleton
   └─> All 11 top-level folders + Templates/_fragments + Templates/_config
   └─> Empty INDEX.md stubs in 06/08/09/10

Step 3: Write templates (MUST come before Periodic Notes configured)
   ├─> Templates/Daily.md with full frontmatter schema + Templater logic
   ├─> Templates/Weekly.md (mostly Dataview body)
   ├─> Templates/Monthly.md (mostly Dataview body)
   └─> Templates/_fragments/* (Manhã Intencional, Benchmark, 5 Pilares)

Step 4: Configure Periodic Notes
   ├─> Daily: format `YYYY/MM-MMMM/YYYY-MM-DD`, template → Templates/Daily.md
   ├─> Weekly: format `02 - Semanal/GGGG/GGGG-[W]WW`, template → Weekly.md
   └─> Monthly: format `03 - Mensal/YYYY/YYYY-MM`, template → Monthly.md

Step 5: Create rotinas
   ├─> Rotina-Dia-Util.md (conteúdo)
   ├─> Rotina-Fim-Semana.md (conteúdo)
   └─> Ativa.md (aponta para Dia-Util)

Step 6: Create reference data (optional at start, but required for coach)
   ├─> 07 - Frases/Banco-Frases.md (seed com ~30 frases)
   ├─> 08 - Livros/ → at least 1 livro file (so coach has reference)
   └─> 06 - Metas/Metas-2026.md (current targets)

Step 7: Test end-to-end without user data
   ├─> Create 1 daily note manually → verify template fills frontmatter
   ├─> Fill frontmatter with dummy scores
   ├─> Create current weekly note → verify Dataview returns the 1 daily
   ├─> Create current monthly note → verify aggregate
   └─> All queries must render without error on near-empty vault (Pattern 5)

Step 8: Build Home.md (LAST — after all data sources produce real values)
   ├─> Lightweight: saudação, frase, HOJE, Ativa embed
   ├─> Links to Semana-Atual, Mes-Atual, Metas, Pessoas
   └─> Configure Homepage plugin → points here

Step 9: Build sub-dashboards
   ├─> 00 - Dashboard/Semana-Atual.md (Tracker charts de pilares, 7d view)
   └─> 00 - Dashboard/Mes-Atual.md (evolução, progress de metas)

Step 10: Seed & go live
   ├─> Fill the first real daily note
   ├─> Run /prepara meu dia
   └─> Iterate based on friction observed in week 1
```

**Critical dependencies:**

- `Templates/Daily.md` must exist BEFORE Periodic Notes plugin is configured (else Periodic Notes creates empty daily notes).
- `Ativa.md` must exist BEFORE Home.md references it (else `![[Ativa]]` shows broken embed).
- At least 1 daily note must exist BEFORE testing weekly aggregation (else Dataview shows "no results" — valid but can't validate query).
- At least 7 daily notes must exist BEFORE weekly averages become meaningful.
- At least 30 daily notes must exist BEFORE monthly trend has signal.
- At least 15 daily notes with time-log must exist BEFORE coach's time-estimate calibration is valid (matches PROJECT.md rule).

**Testing order on empty vault (before user fills data):**

1. Create 3 synthetic daily notes with made-up scores spanning 1 week.
2. Verify: weekly note Dataview aggregates these 3 correctly (avg over 3, not assumed 7).
3. Delete synthetic notes.
4. Verify: dashboard renders "empty state" gracefully (Pattern 5 — no red errors).
5. Create first REAL daily note.

---

## Obsidian-Specific Hard Warnings

| Warning | Source | Mitigation |
|---------|--------|-----------|
| Dataview slows down substantially at 1500+ notes with unscoped queries (forum reports: 3k notes = ~30s initial load) | Obsidian forum, Dataview GitHub discussions | Always path-scoped queries (`FROM "folder"`). Windowed time ranges. |
| DataviewJS re-executes on every keystroke in any vault file (documented CPU issue — 20-30s recomputes on large computations) | Dataview issue #1280 | Use DQL (Dataview Query Language) over DataviewJS when possible. If DVJS needed, keep lightweight. |
| Transclusion inside callouts doesn't always update on source edit | Obsidian forum (bug graveyard) | Avoid putting `![[Ativa]]` inside `> [!note]` callouts. Use plain transclusion. |
| Canvas has severe lag with many embedded dataview nodes on medium vaults | Obsidian forum (canvas performance) | Do NOT use Canvas as primary dashboard substrate. Canvas OK for brainstorm/mapping, not for daily-use dashboard. |
| Large Dataview tables slow Live Preview mode | Obsidian forum | Use Reading Mode for dashboards with heavy tables. |
| Obsidian metadata cache re-indexes on every file change → flicker in open Dataview views | Observed behavior | Close dashboards during bulk data entry. |
| Dataview may be deprecated in favor of Datacore/Bases (community rumor, not official roadmap) | Dataview repo discussions | Use frontmatter (not inline `::`) for all queryable data — migrates cleanly to any future engine. |
| ISO week `YYYY-WW` fails at year boundary (Jan 1, 2026 is W01 but belongs to 2025 in ISO) | moment.js docs | Use `GGGG-[W]WW` format always. |
| Templater does NOT trigger on Periodic Notes creation unless "Trigger on new file" is ON | Templater docs | Verify this setting explicitly. Without it, daily notes don't get dynamic frontmatter. |

---

## Sources

- [Steph Ango (Obsidian CEO) — How I use Obsidian](https://stephango.com/vault) — folder philosophy, "avoid folders, minimize nesting"
- [PARAZETTEL — Breaking Down Kepano's Personal Vault Template](https://parazettel.com/articles/what-i-learned-after-kepano-vault/) — practical vault patterns from Obsidian CEO
- [Periodic Notes plugin — GitHub](https://github.com/liamcain/obsidian-periodic-notes) — daily/weekly/monthly folder format config
- [Daily notes: Allow formatting file location (subfolders based on year/month) — Obsidian Forum](https://forum.obsidian.md/t/daily-notes-allow-formatting-file-location-subfolders-based-on-year-month/19711) — exact YYYY/MM-MMMM/YYYY-MM-DD format documentation
- [Structured daily/weekly notes in Obsidian — Michal Bryxi](https://dev.to/michalbryxi/structured-dailyweekly-notes-in-obsidian-2n5h) — periodic notes + templater integration patterns
- [Dataview performance: Query too slow with many notes — GitHub Discussion #2151](https://github.com/blacksmithgu/obsidian-dataview/discussions/2151) — performance ceiling data
- [Slow file loads — need help with more efficient Dataview queries (Obsidian Forum)](https://forum.obsidian.md/t/slow-file-loads-need-help-with-more-efficient-dataview-queries/87264) — path-scoping recommendation
- [Dataview: Very Slow Performance — Obsidian Forum](https://forum.obsidian.md/t/dataview-very-slow-performance/52592) — 1500 notes threshold reports
- [Serious Issue: High CPU Consumption — Dataview Issue #1280](https://github.com/blacksmithgu/obsidian-dataview/issues/1280) — DataviewJS CPU issue
- [Performance difference yaml vs inline — Dataview Discussion #1029](https://github.com/blacksmithgu/obsidian-dataview/discussions/1029) — frontmatter vs inline perf parity
- [Aggregating Weekly Notes With Obsidian and Dataview — Brian Meeker](https://brianmeeker.me/2022/11/25/aggregating-weekly-notes-with-obsidian-and-dataview/) — DataviewJS pattern for week-number matching
- [Example GROUP BY Queries — Dataview Example Vault](https://s-blu.github.io/obsidian_dataview_example_vault/20%20Dataview%20Queries/Example%20GROUP%20BY%20Queries/) — GROUP BY patterns reference
- [Obsidian Tracker plugin — GitHub](https://github.com/pyrochlore/obsidian-tracker) — chart rendering from frontmatter
- [Obsidian Tracker: Transform Notes Into Powerful Visualizations (BrightCoding 2026)](https://blog.brightcoding.dev/2026/03/18/obsidian-tracker-transform-notes-into-powerful-visualizations) — current (2026) Tracker use patterns
- [Homepage plugin — GitHub](https://github.com/Rainbell129/Obsidian-Homepage) — homepage navigation (convenience, not perf)
- [My Obsidian Canvas Homepage Dashboard — Jake Mahr](https://medium.com/obsidian-observer/my-obsidian-canvas-homepage-dashboard-67c6ce1613c5) — canvas as dashboard (with stability caveats)
- [Canvas Performance — Obsidian Forum](https://forum.obsidian.md/t/canvas-performance/108795) — canvas lag documentation
- [Transclusion in Obsidian — Obsidian Forum](https://forum.obsidian.md/t/transclusion-in-obsidian/69551) — embed patterns
- [Transclusion within a callout doesn't update — Obsidian Forum bug](https://forum.obsidian.md/t/transclusion-within-a-callout-doesnt-update/37941) — callout+embed bug
- [Daily note template with Templater + Dataview — bennewton999 GitHub Gist](https://gist.github.com/bennewton999/62b4a034445a24532591bc4c55a52cf5) — frontmatter + Templater reference

---
*Architecture research for: Obsidian personal high-performance planner vault*
*Researched: 2026-04-23*
