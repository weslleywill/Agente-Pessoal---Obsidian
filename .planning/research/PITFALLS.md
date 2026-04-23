# Pitfalls Research

**Domain:** Personal planner + AI coach system built in Obsidian (Windows 11, long-lived personal vault)
**Researched:** 2026-04-23
**Confidence:** HIGH (most findings verified against Obsidian forum, GitHub issues, Microsoft docs, peer-reviewed research. Some synthesis claims flagged as MEDIUM.)

---

## Critical Pitfalls

### Pitfall 1: Dataview query bloat slows the vault to a crawl after ~1,000 notes

**What goes wrong:**
Dashboard queries that scan the entire vault (no `FROM "01 - Diário/2026"` scoping, no `LIMIT`) become unusable as daily notes accumulate. Users with ~1,500 notes report Obsidian freezing for several seconds when opening their dashboard; users with 3,000+ notes report queries taking ~30 seconds to load. On Windows 11 specifically, Dataview combined with certain other plugins (e.g., Custom Attachment Location) can freeze Obsidian entirely on folder rename.

**Why it happens:**
Dataview re-evaluates queries on every note change and every view reopen. A dashboard with 6–10 unscoped queries (Home.md usually has: today's tasks, weekly pillar scores, monthly goals progress, calendar, recent reflections, 3 priorities) multiplies the cost. Running them against the whole vault when only current week/month is relevant = wasted work. Rendering cost scales with number of rows returned, not just number of files scanned.

**How to avoid:**
- Every Dataview query MUST be scoped to the narrowest folder possible: `FROM "01 - Diário/2026/04"` not `FROM "01 - Diário"`.
- Every query MUST have a `LIMIT` (e.g., `LIMIT 10` for recent reflections, `LIMIT 100` for Tasks-style rollups).
- Prefer `WHERE file.cday >= date(today) - dur(30 days)` over iterating all files.
- Use **DataviewJS only when DQL can't express it** — DataviewJS is harder to optimize and easier to break.
- Cache computed scores (weekly pillar averages) in frontmatter of the weekly note, not recomputed from all daily notes every render.
- Benchmark: if a dashboard takes >1s to render on a cold vault open, the query is already wrong.

**Warning signs:**
- Obsidian hangs 2–5s when opening Dashboard/Home.md.
- Typing in any daily note has perceptible input lag.
- CPU spikes visibly (Task Manager) when switching notes.
- Graph view takes >3s to render.

**Phase to address:**
**Phase 3 (Dashboard + Queries)** — every query added must pass the scope/limit check before the phase closes. Re-audit in **Phase 6 (Hardening/Scaling)** at 365+ notes.

---

### Pitfall 2: Frontmatter schema drift — renaming a field breaks every old note retroactively

**What goes wrong:**
You start with `pilar_saude: 4` in daily notes. Three months in, you decide `saude_score` is clearer and rename it in the template. Now 90 old notes have `pilar_saude` and new ones have `saude_score`. Every Dataview query returns partial data or zeros. Weekly/monthly aggregations become silently wrong. The issue often surfaces months later when trend charts look "broken" — but the real bug is schema inconsistency, not the chart.

**Why it happens:**
- Markdown has no schema enforcement. Obsidian happily accepts any property name.
- Templater prompts let you type any string — no validation.
- Frontmatter is authored by hand in edge cases (user edits a note directly), bypassing the template.
- Property name changes feel cosmetic ("I'll just rename this") but have global impact.

**How to avoid:**
- **Freeze the frontmatter schema in Phase 2** (before any real data is captured). Write it down in `Vault/_meta/schema.md` with the exact property names, types, and valid value ranges.
- All property names in `snake_case` and PT-BR consistent (decide once: `saude` or `pilar_saude`, never mix).
- If a rename is unavoidable, write a migration Templater script that walks every file and updates `processFrontMatter()` — don't do it manually.
- Use Obsidian's **Properties sidebar** (built-in since 1.4) for authoring — catches typos that handwritten YAML doesn't.
- Validate the schema at the Dataview layer: queries should have a "missing data" branch that flags notes with the wrong property name, not silently return 0.

**Warning signs:**
- Weekly pillar averages show "0" or "NaN" for notes you know have scores.
- Dataview `TABLE saude_score` returns blank cells for notes older than a certain date.
- You find yourself writing `WHERE saude_score != null OR pilar_saude != null` — that's schema drift in the wild.

**Phase to address:**
**Phase 2 (Templates + Schema)** — lock schema before any real capture starts. Revisit in **Phase 6** only if migration is needed.

---

### Pitfall 3: Plugin abandonment kills a core dependency mid-project

**What goes wrong:**
The planner depends on Periodic Notes, Calendar, Templater, Dataview, Tasks. Periodic Notes and Calendar (Liam Cain's plugins) are **confirmed abandoned** by the original developer despite being used by 1M+ vaults. If Obsidian releases a breaking API change (it has in 2026 — `BaseOption#shouldHide` signature change), an abandoned plugin stops working and there is no one to fix it. The entire daily-note creation workflow breaks.

**Why it happens:**
- Community plugins are built by volunteers; maintenance dies when the volunteer loses interest.
- Obsidian's plugin API isn't frozen — occasional breaking changes happen in major versions.
- Users concentrate risk by adopting the most popular plugins (which also means most-used = most exposure when they break).

**How to avoid:**
- **Know your dependency graph.** Document in `Vault/_meta/plugins.md`: for each plugin, (a) what breaks if it dies, (b) fallback plan.
- Prefer plugins with **multiple active maintainers** and recent commits. Check GitHub: last commit date, open issue count, responsiveness. Plugins to watch: Templater (SilentVoid13, active), Dataview (blacksmithgu, slower but alive), Tasks (obsidian-tasks-group, healthy — collective maintenance model).
- For Periodic Notes specifically: track the **Journals plugin** (modern alternative) and **Periodic Notes Calendar** (luiisca's fork) as contingencies. Don't migrate preemptively, but know where to jump.
- Core workflow should **degrade gracefully**: if Templater dies, daily note should still open with a reasonable default (Obsidian's built-in Daily Notes core plugin covers the bare minimum).
- Keep plugin config files in git so rollback is possible.

**Warning signs:**
- Plugin GitHub repo hasn't had a commit in 6+ months.
- Open issues piling up with no responses.
- Plugin stops working after an Obsidian update.
- Maintainer announces they're stepping away.

**Phase to address:**
**Phase 1 (Foundation/Plugin selection)** — explicit plugin-health audit as an acceptance criterion. Re-audit at each milestone boundary.

---

### Pitfall 4: Coach hallucinates book citations despite anti-achismo rule

**What goes wrong:**
User asks "me recomenda uma forma de priorizar amanhã". Coach responds "Como Joel Jota diz no capítulo 7 do Alta Performance, priorize os 3 MITs…". Problem: that's not a real chapter. Or the book exists but the quote is invented. User trusts the citation, internalizes a "principle" that doesn't exist, then cites it to a friend. GPT-4 fabricates citations ~20% of the time; GPT-3.5 does so 98% of the time. Even Opus-class models hallucinate when asked for specific chapters/page numbers without grounding.

**Why it happens:**
- LLMs are prediction machines: "plausible-sounding citation" is exactly the thing they're best at generating.
- System prompt rules ("don't invent") help but don't eliminate the failure mode — the model doesn't *know* what it doesn't know.
- Training data contains thousands of self-help book quotes, all jumbled; the model can remix them into confident-sounding but wrong attributions.
- The `Vault/08 - Livros/` rule (must cite an existing file) is only enforced if the coach's prompt *actually checks the filesystem* — a plain-text rule in the prompt doesn't enforce anything.

**How to avoid:**
- **RAG-style grounding**: coach's first action when considering citing a book is to Glob `Vault/08 - Livros/*.md`, pick a relevant file, Read it, then quote ONLY from what's actually in that file. If no file matches, coach must say "não tenho nota de livro sobre isso" and stop.
- Enforce the structural rule in the coach's protocol, not just in the prose: "Step X: if you want to cite a book, you MUST have just Read a file from `08 - Livros/`. Otherwise, don't cite."
- Coach output format should include a **sources section** with filesystem paths: "Fonte: `08 - Livros/Alta-Performance-Joel-Jota.md#priorização`". If the path doesn't resolve, the user knows it's fake.
- When coach gives general advice (not from a book), it must be explicitly marked: "(opinião genérica, sem fonte validada)".
- Add a verification step the user can trigger: `verifica fontes` → coach re-reads every cited file and confirms the quote is present.

**Warning signs:**
- Coach cites a book/chapter/page without a filesystem path.
- User follows up "onde diz isso?" and coach can't point to a specific file + section.
- Two sessions cite the "same" quote with slightly different wording — classic hallucination fingerprint.

**Phase to address:**
**Phase 4 (Coach de Priorização)** — the 6-step socratic protocol must include the grounding check as a hard gate. Verify in **Phase 5 (Análise semanal/mensal)** when coach synthesizes longer reflections (higher hallucination risk).

---

### Pitfall 5: Sync corruption from OneDrive/iCloud/Dropbox running concurrently with Obsidian

**What goes wrong:**
User puts the vault in `OneDrive/Obsidian/` or `iCloud Drive/Obsidian/` for "free sync". Obsidian writes a note, OneDrive is mid-upload, Obsidian writes again → file conflict. Result: `Diário-2026-04-23 (1).md` and `Diário-2026-04-23-DESKTOP-ABC123.md` appearing randomly. iCloud on Windows is particularly bad — users report .md files duplicated up to 56 times (`NOTE (56).md`). Files can get stuck mid-upload. Closing Obsidian before sync completes causes data loss.

**Why it happens:**
- Obsidian writes files aggressively (every keystroke in some configurations, every file-change in others).
- Cloud sync clients don't understand that the "same" file is being rewritten — they see a conflict and resolve by duplicating.
- iCloud's Windows client is notoriously buggy with rapid writes.
- Running Obsidian Sync + OneDrive/iCloud simultaneously = guaranteed corruption.

**How to avoid:**
- **Never put the vault in a cloud-sync folder that's actively syncing.** Period.
- For Windows 11 + personal single-device use (this project's case): vault lives at `E:\Claude Code\Agente Pessoal - Obsidian\Vault\` — NOT in `OneDrive`, `iCloudDrive`, or `Dropbox`.
- Backup strategy uses **Obsidian Git plugin** → GitHub private repo. Git commits are atomic, explicit, and won't fight with file writes.
- If mobile access is needed later, **Obsidian Sync** (paid, official) is the only safe option — it's the only sync that understands Obsidian's write patterns. Free alternatives: Syncthing (P2P, no cloud middleman) is acceptable; cloud folders are not.
- Backup interval in obsidian-git: **every 30 minutes active + on close**. Atomic commits mean rollback is always possible.
- Wait for the git commit checkmark before closing Obsidian (same discipline as Obsidian Sync's green checkmark).

**Warning signs:**
- Files with `(1)`, `(2)` suffixes appearing in the vault.
- Files with hostname suffixes like `Diário-DESKTOP-ABC.md`.
- Obsidian reports "file changed outside the editor" constantly.
- Two versions of the same daily note with different content.

**Phase to address:**
**Phase 1 (Foundation/Storage strategy)** — decide and lock the backup/sync approach before any capture begins. Document in `Vault/_meta/backup.md`.

---

### Pitfall 6: Template bloat — too many fields → abandonment within 30 days

**What goes wrong:**
The Joel Jota planner has a LOT of fields: 5 pillar scores, 10 auto-performance categories, morning intention, 3 priorities, tasks, hours, end-of-day benchmark, weekly planning, weekly benchmark, auto-performance review, monthly goals, etc. If the digital version asks *every* field *every* day, the user will face a 20-minute daily data-entry task. Research shows 80% of users abandon habit apps within 3 days; perfectionism paralysis and mental exhaustion are top abandonment drivers. Even the physical planner is designed to be *skippable* — users don't fill every field every day.

**Why it happens:**
- Faithful digital translation feels like the "right" thing (respects the method).
- Templates default to showing all fields — user thinks "I have to fill them".
- Friction accumulates: 20 min × 365 days = 121 hours/year just capturing data.
- Bullet journal literature: "complexity = abandonment" is well documented.

**How to avoid:**
- **Daily template is minimal**: 3 priorities + morning intention + end-of-day check-in (5 pillar scores only, as quick 1–5 numbers). Target: <3 minutes to fill.
- **Weekly template is richer** (planning + benchmark) but happens once per week — 15 min is acceptable.
- **Monthly template is deepest** (benchmark + 6m/1y goals) — once per month, 30 min is acceptable.
- Every field must answer: "what decision does this inform?" If the answer is "none", cut it.
- 10 auto-performance categories → weekly review only, NOT daily. The planner physical book uses them monthly.
- **Optional fields** must look optional. Use comments in template: `<!-- opcional: preencher só se relevante hoje -->`.
- Pomodoro/time-log capture is automatic (not a field) — user just runs the timer, the data is logged for them.
- Measure: if user skips the daily note 3 days in a row, the coach should notice and ask "quer simplificar o template?" — the system adapts, not the user.

**Warning signs:**
- User starts skipping days after week 2.
- User fills fields with placeholder data ("ok", "normal", "5") just to close them out.
- User opens the dashboard but doesn't open daily notes — capture death.

**Phase to address:**
**Phase 2 (Templates)** — design minimal daily template first, richer weekly/monthly. Principle: "cheap daily, medium weekly, deep monthly". Re-evaluate in **Phase 6** based on actual usage data.

---

### Pitfall 7: Score inflation — everything becomes 5/5 after a few months

**What goes wrong:**
User rates the 5 pillars daily. In week 1, scores range 2–5. By month 3, every score is 4 or 5 — partly because user has improved, partly because of self-rating bias. Harvard research on performance reviews shows self-evaluations drift toward anchoring: users lean toward their own prior scores. Score inflation makes the pillar chart useless (flat line near 5), which makes weekly/monthly analysis useless ("everything is great!"), which breaks the coach's ability to flag problems.

**Why it happens:**
- Self-rating has a positivity bias (nobody wants to write "my health was 2/5 today").
- Anchoring: if yesterday was 4, today feels like "at least a 4".
- No objective calibration — "what does 3/5 even mean?" drifts over time.
- The method lacks peer review or external validation.

**How to avoid:**
- **Define the rubric once** in `Vault/_meta/rubric.md`: what does 1, 2, 3, 4, 5 actually mean for each pillar? Concrete anchors: "Saúde 5 = treinei + comi limpo + 7h sono + energia alta o dia todo. Saúde 3 = fiz 2 dos 4. Saúde 1 = nenhum dos 4."
- **Calibration check weekly**: coach's Sunday analysis must flag "all scores 4+ for 14 days — rubric drift or genuine growth? Confirma."
- **Variance requirement**: if all 5 pillars are the same score 3 days in a row, coach asks "diferencia — qual foi o mais alto, qual o mais baixo?".
- **Comparison with data**: pillar Saúde can be cross-checked against wearable data or pomodoro count. If user rates Saúde 5 but did 0 pomodoros of deep work and slept 5h, coach flags the mismatch.
- **Annual recalibration**: once a year, coach surfaces "your average score this year was 4.3. A year ago it was 4.1. Either you genuinely improved or the rubric drifted. Want to recalibrate?"

**Warning signs:**
- Standard deviation of pillar scores drops below 0.5 over a month.
- All 5 pillars identical for 7+ consecutive days.
- User answers "fine" to every retrospective prompt.

**Phase to address:**
**Phase 2 (Schema/Rubric)** — rubric authored alongside frontmatter schema. **Phase 5 (Análise semanal/mensal)** — coach's analysis includes drift detection.

---

### Pitfall 8: Performative completion — filling the planner without reflecting

**What goes wrong:**
User fills every field but in a hollow way: "today's 3 priorities" = yesterday's 3 priorities copy-pasted. "End of day reflection" = "productive day". "Pillar scores" = yesterday's numbers ±1. The planner becomes a ritual of compliance, not a tool for decision-making. Worse, it looks like engagement to the coach (user is active!), so coach doesn't flag anything. Users gaming productivity metrics — creating fake tasks, splitting work artificially — is well documented.

**Why it happens:**
- Streaks and completion percentages create incentive to fill even when there's nothing meaningful to capture.
- Effort-aversion: writing "I don't know what my priorities are today" is harder than copy-pasting.
- No external accountability — the user is both rater and rated.
- Coach optimizing for "user filled the template" instead of "user made a real decision".

**How to avoid:**
- **Coach detects copy-paste patterns**: if today's 3 priorities match yesterday's 3 priorities exactly (string comparison), coach asks "são realmente os mesmos ou é copy-paste? sem julgamento, só quero entender".
- **Reflection prompts vary**: don't ask "how was your day?" every night — rotate (weekly pergunta rotativa from extras list is already planned) so the user can't auto-respond.
- **Empty is valid**: template explicitly allows "hoje não decidi 3 prioridades, foi dia reativo" as a legitimate answer. Make "não sei" a first-class option.
- **Coach weights reflection depth, not completion**: a 3-line honest reflection > a 20-line performative one. Dashboard metrics should reward qualitative signal, not word count.
- **Weekly "honestidade check"**: on Sunday, coach picks one day from the week and asks "relendo 2026-04-17, você descreveu como produtivo. O que especificamente mudou por causa daquele dia?". If user can't answer, that's the signal.

**Warning signs:**
- Same 3 priorities for 5+ consecutive days.
- Reflection field always 1 sentence.
- User starts filling the dashboard days in advance (planning the whole week Sunday night with fake "morning intentions" for Thursday).

**Phase to address:**
**Phase 5 (Análise semanal/mensal)** — coach's pattern detection includes copy-paste + shallow-reflection flags. **Phase 4 (Coach de Priorização)** — socratic protocol naturally surfaces performativity by demanding "why?" before accepting any priority.

---

### Pitfall 9: Over-questioning — coach asks 10 questions, user wants to log fast

**What goes wrong:**
The 6-step socratic protocol is great for deep priority decisions. But when user types `log tarefa: revisar relatório 2h`, coach responds with "por quê?", "qual meta isso serve?", "qual o prazo?", "e se você não fizer?". User wanted a 5-second log, got a 5-minute interview. After 3 days of this, user stops logging.

**Why it happens:**
- The coach can't distinguish "deep priority decision" from "quick capture".
- Protocol is applied uniformly — same ceremony for everything.
- User never explicitly signals intent.

**How to avoid:**
- **Two modes, explicit**: `/prepara meu dia` (full socratic, deep) vs `log` (capture, zero questions). Coach respects the mode.
- **Intent detection**: if user message is <10 words and starts with a verb + object ("fazer relatório 2h"), treat as capture. Only activate socratic if user says "me ajuda a priorizar" or `/prepara`.
- **Default to silence**: when in doubt, capture first and ask questions later (weekly review). Don't interrupt flow.
- **Coach's socratic mode has an exit**: after 2 questions, if user says "só registra", coach logs and stops.
- **Friction budget**: total coach questions/day capped (e.g., 5). Coach spends the budget on the highest-stakes decisions, not on every tiny task.

**Warning signs:**
- User stops invoking the coach.
- User logs tasks outside the system (back to physical planner, back to Todoist).
- Session length increasing — user spending 20min/day just talking to the coach.

**Phase to address:**
**Phase 4 (Coach de Priorização)** — mode separation (deep vs capture) is a Phase 4 acceptance criterion, not an optimization.

---

### Pitfall 10: No feedback loop — coach never updates when user corrects it

**What goes wrong:**
Coach estimates "revisar relatório = 2h based on history". User actually finishes in 45min. User says "na verdade levou 45min". Coach says "ok" — but next time estimates 2h again. User corrects it 10 times across 2 weeks. Coach never learns. Trust erodes. User stops correcting and just works around the bad estimates.

**Why it happens:**
- The coach's "memory" is limited to what's explicitly written in notes — but if the correction isn't logged with structure, it's invisible next time.
- No persistent learning layer between sessions.
- Estimates based on "history" often mean "the first 3 examples the coach remembers", not weighted-recent data.

**How to avoid:**
- **Corrections write to the data layer**: when user corrects an estimate, coach writes `tempo_real: 45min` to the task's frontmatter AND the correction is surfaced in the weekly calibration review.
- **Time estimates come from actual data, not LLM guessing**: pomodoro + manual time-log (already in requirements) feed into a calibration table. Coach queries the table, doesn't generate estimates from thin air.
- **Calibration minimum threshold**: coach says "sem histórico suficiente" when <3 samples exist (already in anti-achismo rules). Enforce this structurally.
- **Weekly calibration reflection**: coach reports "estimativas vs realidade esta semana: 5 acertos, 2 subestimados (~30%), 1 superestimado. Padrão: subestimo tarefas de escrita."
- **User-editable calibration file**: `Vault/_meta/calibration.md` — user can manually correct biases coach picked up wrong.

**Warning signs:**
- Same estimate error repeated across sessions.
- User stops correcting (passive acceptance = learned helplessness).
- Coach's confidence in estimates doesn't match hit rate.

**Phase to address:**
**Phase 4 (Coach)** — feedback-to-frontmatter loop as acceptance criterion. **Phase 5 (Análise semanal)** — calibration review as part of weekly analysis.

---

### Pitfall 11: Windows MAX_PATH (260 chars) breaks deeply nested vault paths

**What goes wrong:**
Vault is at `E:\Claude Code\Agente Pessoal - Obsidian\Vault\` (already 49 chars). Add `01 - Diário\2026\04\2026-04-23 - quinta-feira - notas sobre a reunião com o cliente X.md` — easily 120+ chars. Total path: ~170 chars — still safe. But nest one more level (`anexos/`, screenshots subfolder) and you hit 260. Windows File Explorer, git, and some plugins fail silently on paths >260. Users report Obsidian can write a file but File Explorer can't open it, causing confusion about which file is "real".

**Why it happens:**
- Windows API's `MAX_PATH = 260` is still the default even on Windows 11.
- Vault path is absolute in many operations (plugins, git, sync).
- Users don't notice the limit until a specific deep file fails.

**How to avoid:**
- **Enable long path support on Windows 11**: `HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled = 1` via registry, OR Group Policy "Enable Win32 long paths". Document this in `Vault/_meta/setup.md`.
- **Filename convention**: daily notes `YYYY-MM-DD.md` (10 chars), NOT `YYYY-MM-DD-dia-da-semana-observações-do-dia.md`. Put the long content *inside* the note, not in the filename.
- **Cap folder nesting at 4 levels**: `Vault/01-Diário/2026/04/` is 4 — stop there, don't add `/manhã/`.
- **Vault location**: keep vault at a short root (`E:\Vault\`) if feasible, not `E:\Projects\Personal\Obsidian\Vaults\MainVault\`. (This project's current path is fine but worth knowing as a fallback option if problems arise.)
- Audit test: write a script that finds any file with absolute path >250 chars and flag it.

**Warning signs:**
- Git fails to stage specific files with "filename too long".
- File Explorer shows a file but double-click opens blank/error.
- Plugin operations succeed on some files, fail on others with no obvious pattern.
- Backup tools skip specific files.

**Phase to address:**
**Phase 1 (Foundation/Windows setup)** — long-path registry key as acceptance criterion. File/folder naming convention locked here too.

---

### Pitfall 12: Graph view becomes a hairball — every note linked to every other

**What goes wrong:**
User enthusiastically `[[links]]` everything. Daily note links to yesterday, to the week, to all 5 pillars, to 3 book notes, to the current project. After 3 months, graph view is an unreadable tangle — "hairball effect". Users claim this is a feature ("look at all my connections!") but it provides zero insight. Worse, every link is weighted equally in the graph: a passing mention counts the same as a structural parent-child.

**Why it happens:**
- Wiki-linking is frictionless; typing `[[` is easier than *not* typing it.
- Community culture celebrates dense graphs ("look at my beautiful brain!") even when they're not useful.
- Dailies + MOCs + tags + frontmatter links all appear in the graph equally.

**How to avoid:**
- **Link budget per daily note**: max 3–5 intentional links, not 15.
- **Distinguish structural vs reference links**: frontmatter (`pilar: [[Saúde]]`) is structural; body mentions (`lembrei do [[Joel Jota]]`) are reference. In graph view, filter by one or the other.
- **Graph view filters are mandatory**: default graph view for this vault should filter to `path:"01 - Diário"` OR tag-based grouping — not "show everything".
- **Local graph at depth 2** for most navigation — global graph is a wallpaper, not a tool.
- **Use Dataview for real navigation**, not the graph. Graph is for serendipity, Dataview is for finding things.
- Don't force links just to make the graph prettier.

**Warning signs:**
- Graph view takes >3s to render.
- User opens graph, looks at it for 5s, closes it — repeatedly. It's decorative, not useful.
- Every daily note has 10+ links in the sidebar.

**Phase to address:**
**Phase 3 (Dashboard)** — graph view filter presets configured. **Phase 6 (Hardening)** — audit link density, prune if needed.

---

### Pitfall 13: Tasks plugin freezes Obsidian when querying thousands of tasks

**What goes wrong:**
Pomodoro logging creates tasks. Daily notes have 5–10 tasks each. After a year: ~2000 tasks. A Tasks query without `limit` (e.g., the dashboard's "upcoming tasks" block) scans all 2000, renders them, and Obsidian freezes. GitHub issue #697 on obsidian-tasks-group confirms: "many thousands of tasks → Obsidian freezes and becomes unresponsive, requires force-quit." A breaking change was shipped specifically to add safeguards against empty queries freezing the app.

**Why it happens:**
- Tasks plugin scans all markdown files for `- [ ]` / `- [x]` patterns.
- Default queries have no limit.
- Rendering cost scales with number of tasks rendered, not just scanned.
- Completed tasks aren't pruned by default — they accumulate forever.

**How to avoid:**
- **Every Tasks query has a `limit 100` (or less)**.
- **Scope Tasks queries with `path includes`**: e.g., `path includes 01 - Diário/2026` — narrows scan dramatically.
- **Archive completed tasks periodically**: quarterly, move completed tasks from daily notes to `09-Arquivo/tasks-2026-Q1.md`. Or use Tasks plugin's "done date" filtering to exclude tasks done >90 days ago.
- **Don't use Tasks plugin for ephemeral TODOs**: pomodoro "tasks" (started/stopped sessions) shouldn't be Tasks-plugin tasks — log them as timestamps in frontmatter instead.
- Benchmark: run `not done` query on the whole vault, measure render time. If >500ms at year 1, rework query scoping.

**Warning signs:**
- Typing in any note has 100ms+ input lag.
- Dashboard freezes for 2+ seconds when it opens.
- Obsidian requires force-quit after switching to a tab with Tasks queries.

**Phase to address:**
**Phase 3 (Dashboard)** — Tasks query scoping + limits as acceptance criterion. **Phase 6 (Hardening)** — archive flow if task count >1000.

---

### Pitfall 14: Privacy leak — coach cites personal reflections publicly

**What goes wrong:**
User shares a coach session publicly (for feedback, tutorial, etc.) and it contains excerpts from their journal reflections. Or: user sets up coach to summarize the week on social media, coach includes raw quotes from "dark" antifragility prompts. Or: backup repo on GitHub is accidentally set to public and 6 months of daily reflections are world-readable.

**Why it happens:**
- Obsidian vault is local, so users treat it as implicitly private — but any sharing breaks that assumption.
- Coach has full vault read access; doesn't distinguish "shareable" from "private".
- Git backup defaults to whatever repo visibility user selects; typo = public.
- User forgets a specific reflection is sensitive when generating a monthly review PDF.

**How to avoid:**
- **Privacy zones in the vault**: `Vault/_private/` folder for sensitive reflections. Coach has read access for analysis but NEVER quotes from it in any output that could be external (exports, shared sessions). Enforce in coach's system prompt and verify.
- **GitHub repo is PRIVATE** — verify at creation, verify after. `Vault/_meta/backup.md` documents "this repo must be private".
- **Export flow has a privacy review step**: monthly benchmark PDF export (extra from brainstorm) runs through a "redact" step that removes raw quotes, keeps only scores and themes.
- **Sharing = redact**: if user asks "me dá um print do meu último mês" for social media, coach returns scores + themes, not raw reflection text.
- **`.gitignore` for sensitive folders**: `_private/` is either encrypted-at-rest or excluded from git backup entirely (accept backup gap as tradeoff).

**Warning signs:**
- Coach's weekly summary quotes a specific journal line verbatim.
- Monthly PDF export contains raw reflections, not aggregated insights.
- GitHub repo settings show "Public" (should never be true).

**Phase to address:**
**Phase 1 (Foundation)** — privacy zones + GitHub private check as acceptance criterion. **Phase 4 (Coach)** — output redaction rules baked into coach's prompts. **Phase 5 (Analysis)** — export flows have redact step.

---

### Pitfall 15: Method dilution — 50 metrics added, original framework lost

**What goes wrong:**
Over time, user adds "just one more" metric: GTD next-actions field, habit tracker, mood tracker, gratitude list, "one thing today", bullet journal migration, PARA tags, Zettelkasten IDs… Each addition seemed small. 18 months later, the daily note is a 30-field monster and the original Joel Jota 5-pillar method is buried. None of the layered systems work because they're all competing for attention. User can't remember which method is the "real" one.

**Why it happens:**
- YouTube and blogs constantly surface new "systems" — temptation to adopt is constant.
- Adding is easier than removing (loss aversion about any data).
- Users try to merge frameworks instead of picking one.
- The project's "10 extras (A-J)" approach — while deliberate — has this risk if not bounded.

**How to avoid:**
- **Annual method review**: once per year (project anniversary), audit every field. For each: "does this directly serve the Joel Jota method? If no, remove or move to an optional secondary template."
- **Out of Scope is sacred** (already in PROJECT.md): "rastreamento de hábitos além dos pilares do método" is explicitly excluded. Defend it against feature creep.
- **New metrics have to earn their slot**: require 30-day trial in a side template, then evaluate. If it didn't change a decision, remove it.
- **Coach surfaces unused fields**: "o campo `X` foi preenchido 3 vezes nos últimos 90 dias. Remove?"
- Document the "why" of each field in `Vault/_meta/fields.md` — forcing articulation prevents accidental addition.

**Warning signs:**
- Daily template has >10 fields and user fills <6 of them regularly.
- User can't articulate why a specific field exists.
- Two fields measure essentially the same thing (e.g., `energia` + `disposição` + `mood`).

**Phase to address:**
**Phase 2 (Templates)** — field justification documented. **Phase 6 (Hardening)** — annual method review as a scheduled coach-driven process.

---

## Technical Debt Patterns

Shortcuts that seem reasonable but create long-term problems.

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Unscoped Dataview queries (`FROM ""`) | Easy to write, shows "everything" | Freezes vault at 500+ notes | **Never** — always scope from day 1 |
| Storing computed scores in queries (not frontmatter) | No cache invalidation needed | Recomputed on every render; slow | MVP only, migrate by Phase 3 |
| Hand-editing frontmatter instead of using Properties sidebar | Fast for one-offs | Introduces typos → schema drift | One-off debug only; never in template |
| Skipping `Vault/08-Livros/` structure — coach "knows" the books | Less setup work | Hallucinated citations guaranteed | **Never** — violates anti-achismo rule |
| Vault in OneDrive for "free sync" | No plugin setup | File duplication, data loss | **Never** on Windows |
| No limit on Tasks queries | Shows all tasks at once | Freezes at ~1000 tasks | Prototype phase only; add limit before real use |
| Copy-pasting yesterday's priorities to "save time" | Fast daily capture | Performative completion, coach blind | **Never** — defeats the method |
| Adding plugins without reading the changelog | New features fast | Breaking changes on Obsidian update | Never for core-workflow plugins (Templater/Dataview/Tasks); OK for experimentation |
| Long filenames with dates, day-of-week, topic | Readable in file explorer | Windows MAX_PATH breaks | Never — use short filenames, rich content inside |
| Manual backup ("I'll git commit sometimes") | No setup | Forgotten → 3 weeks of data at risk | Never — automate via obsidian-git |

---

## Integration Gotchas

Common mistakes when connecting to external services.

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| **Git / GitHub** | Pushing a PUBLIC repo by accident | Verify private at creation; audit settings quarterly; `_private/` in `.gitignore` |
| **GitHub** | Committing `.obsidian/workspace.json` every minute (noisy) | Exclude workspace files in `.gitignore`, commit only config + content |
| **OneDrive/iCloud/Dropbox** | Running concurrently with vault | Never. Pick Obsidian Sync (paid), Syncthing (free P2P), or git only |
| **Obsidian Sync (official)** | Closing Obsidian before sync completes (no green checkmark) | Wait for checkmark; configure "prompt before close if syncing" |
| **Templater** | `tp.file.title` before file has been saved | Use `tp.hooks.on_all_templates_executed` or delay title reads |
| **Dataview** | Relying on DataviewJS for rendering HTML dashboards | Use DQL when possible; DataviewJS has more breakage surface |
| **Wearable (future)** | Auto-pulling sleep/HRV data without user approval | Explicitly excluded in Out of Scope until user requests it |
| **Claude Code (coach)** | Coach writing to vault files without version control | Every write happens on top of a git-tracked vault; rollback is always possible |

---

## Performance Traps

Patterns that work at small scale but fail as usage grows.

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Dashboard re-queries whole vault | Home.md takes 2s+ to open | Scope every query to current week/month; cache rollups in frontmatter | ~500 daily notes (year 2) |
| Tasks query without `limit` | Obsidian freezes 5+ seconds | `limit 100` on every Tasks query | ~500 tasks (month 6) |
| Graph view shows everything | Hairball, nothing legible | Filter presets, depth 2 for local graph | ~300 notes with dense linking |
| Daily note frontmatter bloat | File parse is slow, typing lags | Cap frontmatter to ~15 fields; use body for prose | ~30 fields / 500+ daily notes |
| DataviewJS blocks on main thread | Entire Obsidian UI hangs during render | Prefer DQL; if JS needed, chunk work | Any vault if JS is poorly written |
| Pomodoro tasks logged as Tasks plugin items | Thousands of checklist items accumulate | Log pomodoros as frontmatter timestamps, not as checklist tasks | ~1000 pomodoros (year 1 at 3/day) |
| Attachment folder deep-nested with long names | Windows MAX_PATH errors | Short folder names, `Anexos/` at vault root | Any time path >260 chars |
| Weekly analysis reads all daily notes from scratch | Sunday analysis takes 30s+ | Incremental rollups (daily → weekly cached field) | ~3 months of data |
| Full-text search across all notes for coach context | Claude Code context bloat | Retrieve only current week + specific book notes | Year 1+ |

---

## Security Mistakes

Domain-specific security issues beyond general web security.

| Mistake | Risk | Prevention |
|---------|------|------------|
| Public GitHub repo for backup | 6+ months of personal reflections readable by anyone | Private repo; verify at creation; audit every milestone |
| Coach quoting `_private/` content in shared outputs | Sensitive content leaked to social media or chat screenshots | Coach prompt enforces redaction; `_private/` is explicitly opt-in for quoting |
| Sharing coach sessions (for feedback/tutorial) with raw journal text | Emotional/medical/financial details exposed | Redact journal quotes before sharing; share scores + themes only |
| Monthly PDF export contains raw reflections | User prints and loses document; anyone who finds it reads the journal | PDF export aggregates, doesn't transcribe |
| Plugin auto-updates install malicious version | Plugin with vault read access exfiltrates data | Pin plugin versions in `community-plugins.json`; review before updating core plugins |
| Templater arbitrary code execution | A template copied from internet runs system commands | Only use templates you wrote or from trusted sources (plugin marketplace with reviews) |
| `E:\Claude Code\Agente Pessoal - Obsidian\` accessible by OS-level file search | Another user on the same machine reads notes | BitLocker or folder-level encryption; or user-only permissions on the folder |
| Backing up to a cloud service with weak 2FA | Account compromise → vault exposed | GitHub with 2FA + hardware key; or Obsidian Sync with strong password |

---

## UX Pitfalls

Common user experience mistakes in this domain.

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Coach interrogates user during fast capture | User stops logging, reverts to physical planner | Two modes: `/prepara meu dia` (socratic) vs `log` (silent capture) |
| Dashboard crashes on day 1 (no notes yet) | First impression is a broken product | Empty states: "ainda não há dados — comece com o primeiro daily" |
| Template forces 20 fields every morning | 20-min friction → abandonment in week 2 | Minimal daily (3 priorities + evening check); rich weekly; deep monthly |
| No visible progress indicator | User doesn't feel the system "working" | Weekly analysis highlights "3 melhorias vs semana passada" |
| Coach fires generic advice ("priorize o sono") | User tunes out coach; trust erodes | Every suggestion grounded in user's own data ("seus scores de Saúde caíram 3 dias seguidos — correlaciona com pomodoros noturnos?") |
| "Streaks" shown prominently | User gamesstreak — fills empty notes to avoid breaking it | Show streaks as observation, not score; coach doesn't reward completion |
| No way to say "hoje foi dia ruim, vou pular" | User either fakes data or feels guilt | Template allows `skip: true` with optional reason; doesn't break the streak narrative |
| Coach changes personality session-to-session | User can't trust any specific advice | Coach protocol + tone defined once, locked in system prompt |
| No reflection prompts rotate | User autopilots same 3-sentence response | Rotating weekly prompts (already in plan as extra from brainstorm) |
| Calendar shows every daily note same weight | No visual signal which days were meaningful | Color/sticker daily notes by pillar average score |

---

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **Dashboard renders**: Works when vault is empty (before any daily note exists) — verify empty-state messaging for each block.
- [ ] **Template fires**: Verify Templater runs on `Ctrl+N` for daily note AND on "Open today's daily note" from Periodic Notes sidebar.
- [ ] **Frontmatter schema**: Documented in `_meta/schema.md` AND enforced by a test query that flags non-conforming notes.
- [ ] **Coach citations**: Every book reference in coach output resolves to a real file in `08 - Livros/`. Test with a deliberate "cite something unsupported" prompt — should refuse.
- [ ] **Time estimates**: Coach says "sem histórico suficiente" when <3 samples; verify by querying calibration data.
- [ ] **Backup**: `git log --oneline | head` shows commits from the last 24h AND the repo is PRIVATE on GitHub.
- [ ] **Weekly analysis**: Runs correctly when week has only 2 filled days (partial week scenario).
- [ ] **Monthly analysis**: Handles the case where user was on vacation 2 weeks (skipped notes).
- [ ] **Pillar score rubric**: Documented in `_meta/rubric.md` with concrete anchors for each score level.
- [ ] **Privacy redaction**: Export flows (monthly PDF, shared coach session) strip raw journal text; test with a sensitive sample.
- [ ] **Long paths**: Registry key `LongPathsEnabled = 1` verified; test file at 250+ char path opens correctly.
- [ ] **Plugin health**: `_meta/plugins.md` documents every plugin with last-commit date AND fallback plan if it dies.
- [ ] **Tasks query limits**: Every Tasks code block has `limit <= 100` AND path scoping.
- [ ] **Capture mode**: `log` command works without triggering socratic questions; verify with stopwatch (<5s end-to-end).
- [ ] **Coach feedback loop**: When user corrects an estimate, the correction writes to frontmatter AND is reflected in next week's estimates.
- [ ] **Performative completion detection**: Coach flags exact-copy priorities across consecutive days.
- [ ] **Schema migration path**: If a field needs renaming, there's a Templater script in `_meta/migrations/` — not just manual effort.

---

## Recovery Strategies

When pitfalls occur despite prevention, how to recover.

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Dataview queries freeze vault | LOW | Disable Dataview, edit queries to add scope + limit, re-enable. Restore from git if queries were lost. |
| Frontmatter schema drift detected | MEDIUM | Write Templater migration script in `_meta/migrations/YYYY-MM-DD-rename-X-to-Y.md`. Run against all affected notes. Verify with test query. |
| Plugin abandoned / broken by Obsidian update | MEDIUM-HIGH | Pin plugin to last-working version via manual install. If no fix in 30 days, migrate to alternative (e.g., Periodic Notes → Journals plugin). Test workflow end-to-end. |
| Coach hallucinated citations discovered | LOW | Correct the specific output; add anti-rule to coach system prompt with the specific failure example. Re-test with the same prompt. |
| Sync corruption (duplicate files) | MEDIUM | Stop Obsidian. Close sync client. Manually resolve duplicates (prefer the newest by mtime OR largest by size). Move vault out of sync folder. Restore from git. |
| Score inflation (flatline at 5) | LOW | Run calibration reflection with coach ("qual foi o dia mais baixo este mês e por quê?"). Rewrite rubric anchors with user. |
| Performative completion (copy-paste priorities) | LOW | Coach surfaces the pattern; user + coach agree on a real priority for tomorrow. Archive the hollow week with a note "calibration reset 2026-XX-XX". |
| Vault corruption (any source) | HIGH | `git reset --hard HEAD~1` (or earlier tag); verify integrity. If git backup gap, use Obsidian's built-in `.trash/` for recent files. |
| Private content accidentally in public export | HIGH | Delete the export. If shared: notify recipients, request deletion. Audit export flow, add redact-step test. Rotate anything sensitive (passwords mentioned, etc.). |
| Windows MAX_PATH breaks specific files | LOW | Enable long paths registry key (reboot). OR rename the offending file to shorter name; update links. |
| Tasks plugin freeze | LOW | Kill Obsidian process. Disable Tasks plugin via `community-plugins.json` manual edit. Reopen vault. Add `limit` to every Tasks query. Re-enable plugin. |
| Method dilution (too many fields) | MEDIUM | Annual method review: remove fields that haven't driven a decision in 90 days. Archive removed field data in `_meta/deprecated-fields/`. |

---

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| # | Pitfall | Prevention Phase | Verification |
|---|---------|------------------|--------------|
| 1 | Dataview query bloat | Phase 3 (Dashboard) | Dashboard opens in <1s on cold vault; every query has scope+limit |
| 2 | Frontmatter schema drift | Phase 2 (Templates) | `_meta/schema.md` exists; test query flags non-conforming notes |
| 3 | Plugin abandonment | Phase 1 (Foundation) | `_meta/plugins.md` documents dependency + fallback per plugin |
| 4 | Coach hallucinates citations | Phase 4 (Coach) | "Cite unsupported claim" prompt → coach refuses; output has `08-Livros/` paths |
| 5 | Sync corruption | Phase 1 (Foundation) | Vault NOT in OneDrive/iCloud; obsidian-git configured; GitHub repo private |
| 6 | Template bloat / abandonment | Phase 2 (Templates) | Daily template <3 min to fill; 30-day user-test passes |
| 7 | Score inflation | Phase 2 + Phase 5 | `_meta/rubric.md` exists; weekly analysis includes variance check |
| 8 | Performative completion | Phase 4 + Phase 5 | Coach flags copy-paste priorities; honestidade check weekly |
| 9 | Over-questioning | Phase 4 (Coach) | `log` vs `/prepara` modes separate; `log` <5s end-to-end |
| 10 | No feedback loop | Phase 4 + Phase 5 | Corrections write to frontmatter; weekly calibration review runs |
| 11 | Windows MAX_PATH | Phase 1 (Foundation) | Registry key set; test file at 250+ chars works |
| 12 | Graph view hairball | Phase 3 (Dashboard) | Graph filter presets configured; global graph renders <1s |
| 13 | Tasks plugin freezes | Phase 3 + Phase 6 | All Tasks queries have `limit`; archive flow documented |
| 14 | Privacy leak | Phase 1 + Phase 4 + Phase 5 | `_private/` zone; GitHub private; export redacts quotes |
| 15 | Method dilution | Phase 2 + Phase 6 | Fields justified in `_meta/fields.md`; annual review scheduled |

---

## Sources

**High-confidence (verified against official sources):**
- [Obsidian Forum — Dataview Very Slow Performance](https://forum.obsidian.md/t/dataview-very-slow-performance/52592)
- [GitHub — Dataview query too slow discussion #2151](https://github.com/blacksmithgu/obsidian-dataview/discussions/2151)
- [GitHub — Dataview folder rename freeze issue #2647 (2026-01)](https://github.com/blacksmithgu/obsidian-dataview/issues/2647)
- [Obsidian Forum — Slow Performance Large Vaults](https://forum.obsidian.md/t/slow-performance-with-large-vaults/16633)
- [GitHub — Tasks plugin performance investigation #697](https://github.com/obsidian-tasks-group/obsidian-tasks/issues/697)
- [GitHub — Tasks plugin freeze breaking change #3434](https://github.com/obsidian-tasks-group/obsidian-tasks/issues/3434)
- [Obsidian Forum — Ongoing sync corruption Windows/Android](https://forum.obsidian.md/t/ongoing-sync-corruption-issues-on-windows-android/82834)
- [Obsidian Forum — iCloud duplicate files on Windows](https://forum.obsidian.md/t/obsidian-connected-to-icloud-on-pc-creates-duplicate-files-like-a-lot/34322)
- [Obsidian Forum — OneDrive sync issues](https://forum.obsidian.md/t/obsidian-sync-issues-with-onedrive/47825)
- [Obsidian Help — Troubleshoot Sync](https://help.obsidian.md/sync/troubleshoot)
- [Microsoft Learn — Maximum Path Length Limitation (MAX_PATH)](https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation)
- [Obsidian Forum — MAX_PATH rules](https://forum.obsidian.md/t/what-are-the-rules-to-avoid-max-file-path-lengths/7800)
- [Obsidian Forum — Native Calendar/Periodic Notes core request (mentions abandonment)](https://forum.obsidian.md/t/native-calendar-obsidian-team-needs-to-support-periodic-notes-and-calendar-as-core-plugins/101164)
- [GitHub — obsidian-git (Vinzent03)](https://github.com/Vinzent03/obsidian-git)
- [Harvard Kennedy School — Self-ratings and bias in performance reviews](https://www.hks.harvard.edu/faculty-research/policy-topics/gender-race-identity/self-ratings-and-bias-performance-reviews)
- [PMC — Reducing Hallucinations in AI Chatbots (2025)](https://pmc.ncbi.nlm.nih.gov/articles/PMC12425422/)
- [arxiv — Guarding against AI-hallucinated citations (2025)](https://arxiv.org/html/2503.19848)
- [UNMC Library — AI Hallucinations with Citations (2025 study data)](https://blog.unmc.edu/library/2025/02/12/ai-hallucinations-with-citations/)
- [Practical PKM — 2026 Obsidian Report Card](https://practicalpkm.com/2026-obsidian-report-card/)
- [ObsidianStats — Weekly plugin updates 2026](https://www.obsidianstats.com/posts/2026-03-29-weekly-updates)

**Medium-confidence (domain consensus, less authoritative):**
- [LeStallion — Why people quit bullet journaling](https://lestallion.com/blogs/index/why-do-people-quit-bullet-journaling)
- [My Inner Creative — Why you might fail at bullet journal](https://myinnercreative.com/why-your-bullet-journal-resolutions-might-fail/)
- [Do Not Churn — First 30 days consumer app abandonment](https://www.donotchurn.com/p/where-consumer-apps-lose-users-in-the-first-30-days)
- [Affirmations Flow — Why quit journaling after 3 days](https://www.affirmationsflow.com/blog/why-you-quit-journaling-after-3-days-and-the-stack-and-scale-fix)
- [Computer Weekly — Developer burnout from flawed productivity metrics](https://www.computerweekly.com/news/366563212/Developer-burnout-caused-by-flawed-productivity-metrics)
- [Atlassian — The problem with productivity metrics](https://www.atlassian.com/blog/productivity/the-problem-with-productivity-metrics)
- [Trophy — Productivity gamification that doesn't backfire](https://trophy.so/blog/productivity-app-gamification-doesnt-backfire)
- [Obsidian Forum — Graph view hairball discussions (multiple threads)](https://forum.obsidian.md/t/hide-specific-notes-and-links-from-the-graph-view/65839)

**Domain knowledge / general Obsidian community wisdom (unverified specifics):**
- Template bloat → abandonment correlation is community consensus but not rigorously measured for Obsidian specifically.
- Plugin version-pinning as a defensive pattern is advocated in the Obsidian dev community but has no canonical doc.
- The "weigh reflection depth, not completion" principle for coaches is synthesis across productivity/coaching literature, not a single cited source.

---

*Pitfalls research for: Personal planner + AI coach in Obsidian (Windows 11, long-lived)*
*Researched: 2026-04-23*
*Researcher confidence: HIGH on technical Obsidian pitfalls (1, 2, 5, 11, 12, 13); HIGH on AI hallucination (4) and sync corruption (5); MEDIUM on UX/behavioral (6, 7, 8, 9, 15) — principles are well-established but specific thresholds (e.g., "30 days to abandonment") are approximate.*
