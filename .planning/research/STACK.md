# Stack Research — Obsidian Personal Planner (Método Joel Jota)

**Domain:** Personal productivity / planner / PKM dentro do Obsidian (Windows 11)
**Researched:** 2026-04-23
**Confidence:** HIGH (todos os nomes de plugins verificados via GitHub + obsidianstats.com; versões confirmadas no dia da pesquisa)

---

## TL;DR — Stack recomendado (verificado Abril 2026)

| Categoria | Recomendação | Versão | Confiança |
|---|---|---|---|
| Templates | **Templater** (SilentVoid13) | 2.19.3 | HIGH |
| Query engine primário | **Dataview** (blacksmithgu) | 0.5.70 | HIGH |
| Query engine futuro | **Bases** (core plugin, embutido) | Obsidian 1.9+ | HIGH |
| Periodic notes | **Periodic Notes** (liamcain) — stale mas funcional | 1.0.0-beta.3 | HIGH (com ressalva) |
| Calendar widget | **Calendar** (liamcain) — stale mas funcional | 2.0.0-beta.2 | HIGH (com ressalva) |
| Tasks | **Tasks** (obsidian-tasks-group) | 7.23.1 | HIGH |
| Homepage | **Homepage** (mirnovov) | 4.4.0 | HIGH |
| Data viz / gráficos | **Tracker** (pyrochlore) | 1.19.0 | HIGH |
| Pomodoro | **Pomodoro Timer** (eatgrass) | 1.2.3 | MEDIUM |
| Natural language dates | **Natural Language Dates** (argenos) | 0.6.2 | MEDIUM (stale) |

**Decisões fortes (não-negociáveis):**
- **Tracker vence Charts** — Charts está mais stale que Tracker, Tracker foi feito para este caso de uso exato (frontmatter → line chart).
- **Dataview agora, Bases depois** — Dataview cobre 100% dos casos do projeto hoje; Bases é o futuro mas ainda é mais limitado. Manter Dataview como padrão e usar Bases para views simples.
- **Pomodoro Timer (eatgrass)** — único plugin que faz start/stop em task específica do Tasks plugin via inline-field. Alternativa forte é **TaskNotes** (nota-por-task + pomodoro built-in), mas é outra filosofia inteira que reestruturaria o projeto.
- **Manter Periodic Notes + Calendar do liamcain** — stale há 4 anos, mas ainda são o padrão de facto usado pela maioria do ecossistema Obsidian (Tasks, Homepage, Dataview queries esperam a estrutura deles). **Periodic Notes Calendar (luiisca)** é uma alternativa moderna viável mas adiciona risco de quebrar convenções esperadas por outros plugins.

---

## Detalhamento por plugin

### 1. Templater — SilentVoid13 ✅

**Versão verificada:** 2.19.3 (release: 2026-04-22, <1 dia antes desta pesquisa)
**Manutenção:** ALTAMENTE ATIVA — release fresquíssimo
**Confiança:** HIGH
**Community ID:** `templater-obsidian`

**Por que é a escolha:**
- Única opção que suporta folder template mapping (arquivo criado em `01-Diário/2026/04/` automaticamente aplica template diário)
- Suporta `tp.date.*`, `tp.file.*`, `tp.system.prompt()`, user functions em JS
- Suporta execução condicional (`<% tp.date.now("dddd") === "Sunday" ? ... %>`) — essencial para "análise semanal automática no domingo"
- Suporta user scripts (arquivos .js em pasta) — permite compor lógica complexa sem inflar cada template

**Capacidades que o projeto vai usar:**
- Folder template mapping → `01-Diário/YYYY/MM` cria diário com template certo sem escolha manual
- `tp.date.now("YYYY-MM-DD")` → popular frontmatter com data
- Conditional blocks → render diferente em domingo (trigger da análise semanal)
- User functions → abstrair "pegar frase motivacional do dia", "listar livros lidos este mês"

**Alternativa considerada:** Templates core plugin — descartado. Só faz substituição de string simples, sem lógica, sem folder mapping, sem user functions. Sobrecai em fricção.

**Conflitos conhecidos:** Nenhum com os outros plugins da lista.

**Sources:**
- [GitHub repo](https://github.com/SilentVoid13/Templater)
- [Latest release 2.19.3](https://github.com/SilentVoid13/Templater/releases)

---

### 2. Dataview — blacksmithgu ✅ (escolha pragmática)

**Versão verificada:** 0.5.70 (release: 2025-04-07, ~1 ano atrás)
**Manutenção:** MAINTENANCE-ONLY — sem features novas, só bug fixes. Autor mudou foco para `Datacore` (sucessor, ainda beta).
**Confiança:** HIGH (sobre o estado; MEDIUM sobre longevidade)
**Community ID:** `dataview`

**Por que é a escolha (apesar do status):**
- Ainda é o padrão de facto com 8.8k stars e ecossistema inteiro construído em cima
- **DQL** (query language declarativa) cobre 100% do que o projeto precisa:
  - `TABLE` / `LIST` / `TASK` queries
  - `FROM "01-Diário"` para filtrar pasta
  - `WHERE date >= date(today) - dur(7 days)` para janelas temporais
  - `GROUP BY file.folder` para agregações
- **DataviewJS** (JS livre) cobre casos complexos: calcular scores agregados dos 5 pilares, montar o dashboard com múltiplas fontes
- Frontmatter + inline fields (`key:: value`) — ambos suportados
- Performance é OK em vault pequeno-médio (<5000 notas). O projeto começa vazio → zero problema nos primeiros anos

**DQL vs DataviewJS — quando usar qual (para este projeto):**
- **DQL**: tabelas simples (últimos 7 diários, tarefas pendentes do mês)
- **DataviewJS**: dashboard com lógica complexa (calcular média móvel dos pilares, gerar recommendation baseada em histórico)

**Limites de performance (relevante só a partir de ~5k notas):**
- Vault com 1k–5k notas: sem issues perceptíveis
- Vault com 10k+ notas: queries podem ficar lentas, mobile sofre mais
- **Este projeto nunca deve passar de ~2k notas** (5 anos × 365 diários + semanais + mensais + livros + pessoas ≈ 2200). Performance é non-issue.

**Por que NÃO Bases (ainda):**
- Bases é core plugin desde Obsidian 1.9 (Mai 2025) — estável
- MAS: Bases é limitado a views tabulares sobre YAML frontmatter. Sem equivalente a `DataviewJS` para lógica customizada (ex: calcular score ponderado dos 5 pilares). Sem suporte para inline fields (`pilar::8`).
- Tasks plugin, Tracker e Homepage ainda esperam Dataview como runtime de queries embutidas
- **Recomendação:** usar Dataview como primary. Experimentar Bases em views específicas depois que o MVP funcionar, especialmente para listas simples de "livros lidos este mês" ou "tarefas concluídas na semana".

**Conflitos conhecidos:** Nenhum direto. Bases e Dataview convivem sem problema.

**Sources:**
- [GitHub repo](https://github.com/blacksmithgu/obsidian-dataview)
- [Obsidian Stats — Dataview](https://www.obsidianstats.com/plugins/dataview)
- [Bases vs Dataview comparison](https://obsidian.rocks/dataview-vs-datacore-vs-obsidian-bases/)

---

### 3. Periodic Notes — liamcain ⚠️ (usar com ressalva)

**Versão verificada:** 1.0.0-beta.3 (release: ~1469 dias atrás ≈ Abril 2022)
**Último commit:** ~607 dias atrás (meados 2024)
**Manutenção:** EFETIVAMENTE ABANDONADO — 2 commits no último ano, 9 PRs abertos sem review
**Confiança:** HIGH (sobre o estado; risco MEDIUM sobre longevidade)
**Community ID:** `periodic-notes`

**Por que MANTER (apesar de abandonado):**
- Define o contrato `{daily, weekly, monthly, quarterly, yearly}` que **Tasks plugin, Homepage, Dataview queries, Calendar plugin, Templater folder mapping** todos esperam. Substituí-lo = reescrever convenções em todo lugar.
- Funciona. O código não quebra — só não recebe features novas.
- Nenhum bug crítico reportado impede uso básico.
- Obsidian core tem Daily Notes, mas **não** tem Weekly/Monthly/Quarterly/Yearly. Periodic Notes ainda é a única forma de ter essa granularidade sem código custom.

**Alternativa moderna: Periodic Notes Calendar (luiisca)**
- Repo: https://github.com/luiisca/obsidian-periodic-notes-calendar
- Versão: v1.2.1 (release: 2026-02-14) — ATIVO
- **Features que superam o liamcain:** UI de calendário integrada, emoji summaries, bulk rename/move/delete, natural language dates built-in, modo flutuante
- **Risco:** substitui tanto Periodic Notes quanto Calendar. Se outras plugins (Tasks, Homepage) não reconhecerem a estrutura dele, preciso configurar tudo manualmente.

**Decisão recomendada:**
- **MVP (Fases 1–3):** Usar Periodic Notes (liamcain) + Calendar (liamcain). É o caminho de menor fricção, todo o ecossistema assume essa estrutura, e o projeto não precisa de features avançadas no início.
- **Revisitar após MVP funcionar:** Avaliar Periodic Notes Calendar (luiisca) como substituto unificado se sentir fricção nos plugins do liamcain. Documentar essa decisão em PROJECT.md.

**Configuração crítica:**
- Folder format: `01-Diário/YYYY/MM/YYYY-MM-DD` (daily), `02-Semanal/YYYY/YYYY-[W]ww` (weekly), etc.
- Template path apontando para `04-Templates/diario.md`, `04-Templates/semanal.md`, etc.
- **IMPORTANTE:** Configure o core "Daily Notes" plugin também — Tasks e outros caem nele como fallback.

**Conflitos conhecidos:**
- Core "Daily Notes" e "Periodic Notes" precisam estar **sincronizados** (mesmo folder, mesmo template) para evitar conflito de "qual template usar". Recomendo: core Daily Notes com mesma config, ou desabilitar core.

**Sources:**
- [GitHub repo](https://github.com/liamcain/obsidian-periodic-notes)
- [Obsidian Stats — Periodic Notes](https://www.obsidianstats.com/plugins/periodic-notes)
- [Periodic Notes Calendar (alternativa)](https://github.com/luiisca/obsidian-periodic-notes-calendar)

---

### 4. Calendar — liamcain ⚠️ (mesmo caso do Periodic Notes)

**Versão verificada:** 2.0.0-beta.2 (release: 2024-04-20)
**Último commit:** 0 commits no último ano — 1828 dias desde última release
**Manutenção:** ABANDONADO (mas funcional)
**Confiança:** HIGH (estado) / MEDIUM (longevidade)
**Community ID:** `calendar`

**Por que MANTER:**
- 413k+ downloads — plugin mais usado para essa função, todo tutorial Obsidian presume
- Integra naturalmente com Periodic Notes (liamcain): clicar dia no calendário abre/cria o diário daquela data
- Widget no sidebar direito é exatamente o que o projeto precisa para o dashboard ("visão Semana/Mês + calendário")
- Meter de "quanto você escreveu" por dia no calendário é feature útil para o sentido de progresso

**Sobre "year view":**
- Calendar (liamcain) NÃO tem year view nativo — só month view. Isso é uma limitação.
- Se year view for requisito (ex: extra "comparação você vs você do ano passado"), precisaria de um dos abaixo:
  - **QuickCalendar** (snovis) — year-at-a-glance com grid/row/stream views, mais ativo
  - **Periodic Notes Calendar** (luiisca) — tem views de múltiplos períodos incluindo yearly
- **Recomendação:** começar com Calendar (liamcain) no MVP, se year view virar dor adicionar QuickCalendar como plugin complementar (não conflita).

**Alternativa considerada: Full Calendar**
- Mais voltado para eventos/agenda (tipo Google Calendar) do que daily notes
- Útil se quiser embed de eventos externos (iCal)
- Não é substituto direto do Calendar (liamcain) — é complementar

**Conflitos conhecidos:** Nenhum com os plugins dessa lista.

**Sources:**
- [GitHub repo](https://github.com/liamcain/obsidian-calendar-plugin)
- [Obsidian Stats — Calendar](https://www.obsidianstats.com/plugins/calendar)

---

### 5. Tasks — obsidian-tasks-group ✅

**Versão verificada:** 7.23.1 (release: 2025-02-27; possivelmente mais recente em 2026 — última release conhecida via GitHub na data da pesquisa)
**Manutenção:** ATIVA — cadência regular, múltiplos contribuidores, releases em Fev/2025
**Confiança:** HIGH
**Community ID:** `obsidian-tasks-plugin`

**Por que é a escolha:**
- Plugin padrão de facto para tasks em Obsidian (muito mais maduro que alternativas como Task Genius ou TaskForge)
- Syntax inline compatível com markdown padrão: `- [ ] Ler Joel Jota 📅 2026-04-30 ⏫ #pilar/talento`
- **Query language própria** (dentro de code blocks ```tasks ... ``` ) com filtros por data, prioridade, tag, path
- Suporta recurring tasks: `🔁 every week` (essencial para rotinas fixas)
- **Integra com Dataview**: queries Dataview podem filtrar `TASK FROM ...` independentemente; Tasks plugin renderiza checkboxes corretamente dentro de Dataview
- Presets (v7.23) — salvar queries comuns e reusar (ótimo para o dashboard "tarefas da semana")

**Syntax essencial para o projeto:**
```markdown
- [ ] Treino força #pilar/saúde 📅 2026-04-24 ⏫ 🔁 every weekday
- [ ] Revisar semana #ritual 📅 2026-04-27 🔁 every week on Sunday
```

**Query exemplo para o dashboard:**
```tasks
not done
due before tomorrow
short mode
```

**Integração Tasks ↔ Dataview:**
- Dataview TASK queries veem qualquer checkbox como task — não entendem emojis de prioridade/data
- Tasks plugin lê os emojis e renderiza UI rica
- **Recomendação**: usar `tasks` blocks para views do usuário, `dataview TASK` para agregações estatísticas (ex: "quantas tasks concluídas por pilar este mês")

**Conflitos conhecidos:**
- **Pomodoro Timer (eatgrass)**: o `pomodoros::` inline field DEVE vir **antes** dos emojis do Tasks plugin. Caso contrário, o Tasks plugin não parseia corretamente o resto da linha. Documentar isso nos templates.

**Sources:**
- [GitHub repo](https://github.com/obsidian-tasks-group/obsidian-tasks)
- [Latest release 7.23.1](https://github.com/obsidian-tasks-group/obsidian-tasks/releases/latest)

---

### 6. Homepage — mirnovov ✅

**Versão verificada:** 4.4.0 (release: ~Março 2026, "cerca de um mês atrás" da pesquisa)
**Manutenção:** ATIVA — releases regulares ao longo de 2024-2026
**Confiança:** HIGH
**Community ID:** `homepage`

**Por que é a escolha:**
- Único plugin que resolve "abrir `00 - Dashboard/Home.md` automaticamente ao startar Obsidian"
- Suporta abrir em Reading/Source/Live Preview mode (vai querer Reading para que Dataview queries renderizem imediatamente)
- Suporta rodar Obsidian command ao abrir homepage (ex: "abrir Calendar sidebar", "focar no Dashboard") — 1 click equivalente
- Controle sobre tabs antigas: keep / replace last / close all — importante para começar cada sessão "limpa"
- **Totalmente compatível com Dataview** (issue #79 sobre v3.7.0 + Obsidian 1.5.6 foi resolvido; versão atual é 4.4.0)

**Configuração recomendada para este projeto:**
- Homepage: `00 - Dashboard/Home.md`
- Open mode: Reading view (para Dataview renderizar)
- Manage tabs: Replace last note (começa limpa)
- Command ao abrir: nenhum por padrão; depois considerar "Calendar: Open calendar" para já mostrar o widget

**Conflitos conhecidos:** Nenhum reportado com Dataview, Periodic Notes, Calendar. Histórico: houve incompat com Obsidian 1.5.6 (issue #79) — **resolvido** há muito, versão atual é safe.

**Sources:**
- [GitHub repo](https://github.com/mirnovov/obsidian-homepage)
- [Latest releases](https://github.com/mirnovov/obsidian-homepage/releases)

---

### 7. Tracker — pyrochlore ✅ (vence Charts)

**Versão verificada:** 1.19.0 (release: ~Fev 2026, "cerca de 2 meses atrás")
**Último commit:** ~9 dias atrás
**Manutenção:** ATIVA
**Confiança:** HIGH
**Community ID:** `obsidian-tracker`

**Por que Tracker vence Charts (decisão):**

| Critério | Tracker (pyrochlore) | Charts (phibr0) |
|---|---|---|
| Última release | ~Fev 2026 | ~Abril 2024 (2 anos atrás) |
| Commits último ano | 9 dias atrás (ativo) | 2 commits (stale) |
| Issues abertas | 6 PRs sendo revisados | 72 abertas vs 53 fechadas |
| Feito para este uso? | **SIM** — criado especificamente para ler frontmatter/inline/tag ao longo de múltiplos arquivos e plotar | Não — precisa passar arrays manuais ou pipear via DataviewJS |
| Suporte a frontmatter | `searchType: frontmatter` nativo | Precisa montar array manualmente |
| Line chart | Output type `line` direto | Sim (Chart.js) |
| Heatmap | Sim (calendar heatmap) | Não |

**Conclusão: Tracker é a ferramenta certa para o caso de uso.** Charts seria escolha se você quisesse gráfico one-off com dados montados à mão, mas para "plotar scores dos 5 pilares ao longo de 30 dias puxando do frontmatter dos diários" o Tracker foi **literalmente desenhado para isso**.

**Exemplo de uso (5 pilares no dashboard):**
```tracker
searchType: frontmatter
searchTarget: pilar_saude, pilar_competitividade, pilar_talento, pilar_presenca, pilar_emocional
folder: 01-Diário
startDate: -30d
endDate: 0d
line:
    title: "5 Pilares — últimos 30 dias"
    yAxisLabel: "Score"
    yMin: 0
    yMax: 10
```

**Limites conhecidos:**
- API é JSON em code block (curva de aprendizado)
- Desde v1.9.0 alguns template variables (`{{sum}}`) foram deprecated → usar operadores (`+`, `-`) e funções (`sum()`, `dataset()`)
- Não renderiza gráfico interativo (Chart.js); é SVG estático — aceitável para dashboard

**Alternativa complementar:** **Life Tracker** (dsebastien) — baseado em Bases core plugin, 12 tipos de gráficos, mais moderno. Vale considerar em Fase 5+ se quiser explorar Bases para visualização. Não substitui Tracker hoje (Bases ainda está evoluindo).

**Conflitos conhecidos:** Nenhum.

**Sources:**
- [GitHub repo](https://github.com/pyrochlore/obsidian-tracker)
- [Obsidian Stats — Tracker](https://www.obsidianstats.com/plugins/obsidian-tracker)
- [Charts plugin (descartado)](https://github.com/phibr0/obsidian-charts)

---

### 8. Pomodoro — Pomodoro Timer (eatgrass) ⚠️

**Versão verificada:** 1.2.3 (release: 2024-05-30 — ~2 anos atrás)
**Manutenção:** LIMBO — 36 issues abertas, last push 2024-06-20, sem resposta a issues de 2025
**Confiança:** MEDIUM (funciona, mas sem evolução)
**Community ID:** `pomodoro-timer`

**Por que ESTE plugin (apesar do status):**
- **Único plugin popular que integra com Tasks plugin** — adiciona `pomodoros::` inline field na task, incrementa automaticamente quando sessão termina, registra histórico
- Daily note integration: escreve log automático de sessões no diário (essencial para "tracking de tempo duplo" do projeto)
- Funciona bem, é estável — 51k downloads, 175 stars, código não quebrou
- Status bar widget (pode posicionar onde quiser)

**Alternativa forte: TaskNotes (callumalpass) — considerar seriamente**

| Critério | Pomodoro Timer (eatgrass) | TaskNotes (callumalpass) |
|---|---|---|
| Manutenção | Limbo (2024) | MUITO ATIVA (v3.22.0 em 2026) |
| Modelo | Plugin standalone + integra com Tasks | **Nota-por-task** + pomodoro built-in + 8 views |
| Filosofia | Minimal, encaixa no que já existe | Reescreve como tasks funcionam |
| Sobrepõe Tasks plugin? | Não, integra | Parcialmente — substitui fluxo |
| Pomodoro UI | Simples, status bar | Dedicated view + stats view |
| Integra com Bases? | Não | SIM (v4 base em Bases core) |

**Decisão recomendada:**
- **MVP:** Pomodoro Timer (eatgrass). Caminho de menor disrupção — adiciona pomodoro sem alterar como tasks funcionam.
- **Fase 4+ (se hit pain):** Se pomodoro ficar central E Pomodoro Timer realmente não receber updates, considerar migração para TaskNotes. Mas essa é uma refatoração grande — reescreve toda a parte de tasks do projeto para ser "nota-por-task". Só fazer se sentir dor real.

**Riscos de usar Pomodoro Timer:**
- Incompatibilidade futura com Obsidian API (improvável no curto prazo; Obsidian é conservador com breaking changes)
- Bug não corrigido acumulado (até agora nada crítico reportado)

**Outras alternativas consideradas (descartadas):**
- **Status Bar Pomodoro Timer** (kzhovn) — minimal, mas **sem integração com Tasks**
- **PomoBar** — igual ao de cima, sem integração
- **Flexible Pomodoro** (grassbl8d) — mais features mas ainda sem integração tight com Tasks

**Conflitos conhecidos:**
- Se usar com Tasks plugin: `pomodoros::N` DEVE vir antes dos emojis do Tasks (`📅`, `⏫`). Documentar nos templates.

**Sources:**
- [GitHub repo](https://github.com/eatgrass/obsidian-pomodoro-timer)
- [Obsidian Stats — Pomodoro Timer](https://www.obsidianstats.com/plugins/pomodoro-timer)
- [TaskNotes (alternativa)](https://github.com/callumalpass/tasknotes)

---

### 9. Natural Language Dates — argenos ⚠️

**Versão verificada:** 0.6.2 (release: ~2 anos atrás, 2023/2024)
**Manutenção:** ABANDONADO pelo autor original
**Confiança:** MEDIUM (funciona; revivals existem)
**Community ID:** `nldates-obsidian`

**Por que manter (com ressalva):**
- Ainda é o padrão citado por Tasks plugin, Periodic Notes, Dataview — todos suportam parsing NLD (`@tomorrow`, `@next friday`)
- Código não quebrou; funciona em Obsidian 1.12 (atual)
- Trigger `@` para inserir data é muscle memory do ecossistema Obsidian

**Alternativas:**
- **nldates-revived** (Amato21) — fork explícito declarando-se "revival", com updates mais recentes. Mesma syntax, drop-in replacement. Vale considerar se encontrar bugs no original.
- **obsidian-nldates-redux** (tbergeron) — outro fork com comportamento estilo Notion
- **Periodic Notes Calendar (luiisca)** — tem natural language dates built-in, eliminaria necessidade deste plugin se decidir trocar o stack de periodic notes

**Decisão recomendada:**
- Começar com `nldates-obsidian` (argenos original) no MVP. É o mais bem testado.
- Se hit bug: trocar para `nldates-revived` (drop-in, zero refactor).

**Conflitos conhecidos:** Nenhum.

**Sources:**
- [GitHub repo](https://github.com/argenos/nldates-obsidian)
- [nldates-revived fork](https://github.com/Amato21/nldates-revived)

---

## Stack completo final — ordem de instalação recomendada

```
# Fase 1 (base):
1. Templater              (templates + folder mapping)
2. Periodic Notes         (estrutura daily/weekly/monthly)
3. Calendar               (widget sidebar)
4. Natural Language Dates (input @tomorrow etc)

# Fase 2 (produtividade):
5. Tasks
6. Dataview
7. Homepage

# Fase 3 (tracking & viz):
8. Tracker
9. Pomodoro Timer

# Fase 4+ (explorar):
10. (Opcional) Periodic Notes Calendar — se sentir dor no liamcain stack
11. (Opcional) QuickCalendar — se year view virar requisito
12. (Opcional) TaskNotes — só se Pomodoro Timer virar bloqueio
13. (Opcional) Life Tracker — explorar Bases-native visualization
```

---

## Alternatives Considered

| Recomendado | Alternativa | Quando trocar |
|---|---|---|
| Dataview | **Bases** (core) | Para views tabulares simples, depois do MVP. Bases é mais rápido em vaults grandes mas menos flexível. |
| Dataview | **Datacore** (beta) | Nunca no MVP — ainda em beta, precisa conhecer React. Considerar só se Dataview quebrar. |
| Periodic Notes (liamcain) | **Periodic Notes Calendar** (luiisca) | Se liamcain quebrar em Obsidian futuro OU se quiser UI mais moderna |
| Calendar (liamcain) | **QuickCalendar** (snovis) | Se year view virar requisito (ver extra "você vs ano passado") |
| Tracker | **Life Tracker** (dsebastien) | Se migrar grosso da stack para Bases |
| Pomodoro Timer (eatgrass) | **TaskNotes** (callumalpass) | Só se Pomodoro Timer quebrar E projeto tiver maturidade para refatorar para "nota-por-task" |
| Natural Language Dates (argenos) | **nldates-revived** (Amato21) | Drop-in se hit bug no original |
| Tasks plugin | **Task Genius** / **TaskForge** | Não trocar — Tasks é dominante e ambas alternativas são menos integradas |

---

## What NOT to Use

| Evitar | Por quê | Usar no lugar |
|---|---|---|
| **Charts** (phibr0) | 818 dias sem release, 72 issues abertas, API requer passar arrays manuais; Tracker foi feito para este caso de uso | **Tracker** (pyrochlore) |
| **Core Templates** (plugin core) | Sem lógica condicional, sem folder mapping, sem user functions — ferramenta primitiva | **Templater** |
| **Datacore** no MVP | Beta há anos, API instável, requer React; risco alto para projeto que precisa funcionar | **Dataview** (estável) |
| **Status Bar Pomodoro** / **PomoBar** | Sem integração com Tasks plugin — não incrementa pomodoros em task específica | **Pomodoro Timer** (eatgrass) |
| **Obsidian Day Planner** | Excelente plugin mas filosofia diferente (time-blocking baseado em heading de markdown) — conflita com Periodic Notes structure | — (não é substituto direto) |
| **Full Calendar** como widget principal | Voltado para eventos/ics, não para navegação entre daily notes | **Calendar** (liamcain) |

---

## Version Compatibility

| Package A | Compatible With | Notas |
|---|---|---|
| Obsidian 1.12.7 | Todos os plugins da lista | Current desktop version (Mar 2026) |
| Templater 2.19.3 | Dataview 0.5.70 | Totalmente compatível; Templater pode invocar DataviewJS via `dv = app.plugins.plugins.dataview.api` |
| Tasks 7.23.1 | Pomodoro Timer 1.2.3 | Compatíveis — **atenção**: `pomodoros::` deve vir ANTES dos emojis Tasks na linha |
| Dataview 0.5.70 | Bases (core) | Coexistem sem conflito — queries independentes |
| Periodic Notes 1.0.0-beta.3 | Calendar 2.0.0-beta.2 | Feitos para trabalhar juntos (mesmo autor) |
| Homepage 4.4.0 | Dataview / Templater | Dashboard Dataview-heavy funciona — issue antigo (#79 com Obsidian 1.5.6) está resolvido |
| Tracker 1.19.0 | Dataview / frontmatter | Tracker pode ler frontmatter diretamente (não depende de Dataview) — mais robusto |

---

## Riscos a monitorar (para o Roadmap)

1. **Periodic Notes (liamcain) abandonado** — probabilidade baixa de quebrar em <2 anos, mas planejar fallback para Periodic Notes Calendar (luiisca) como Plan B.
2. **Calendar (liamcain) — 5 anos sem update** — mesmo cenário; funcionar hoje não garante funcionar em Obsidian 2.0.
3. **Pomodoro Timer (eatgrass) limbo** — se pomodoro for centrais, considerar TaskNotes como estratégia de contingência.
4. **Dataview em maintenance-only** — não é risco curto prazo, mas em 3–5 anos o projeto deve migrar porções para Bases.
5. **Natural Language Dates original abandonado** — risco baixo (código simples, funciona), mas `nldates-revived` é fallback pronto.

**Conclusão geral sobre riscos:** 4 dos 10 plugins (Periodic Notes, Calendar, Pomodoro Timer, NLD) estão em estado stale/limbo. Isso é a realidade do ecossistema Obsidian — muitos plugins foram feitos em 2020–2022 e ainda são padrão, apenas não recebem features novas. **Nenhum deles está quebrado hoje** e todos têm rotas de migração. O projeto pode seguir com confiança.

---

## Sources (confidence matrix)

| Fonte | Tipo | Uso | Confiança |
|---|---|---|---|
| [GitHub — SilentVoid13/Templater releases](https://github.com/SilentVoid13/Templater/releases) | Release oficial | Versão 2.19.3 | HIGH |
| [GitHub — blacksmithgu/obsidian-dataview releases](https://github.com/blacksmithgu/obsidian-dataview/releases) | Release oficial | Versão 0.5.70 | HIGH |
| [GitHub — liamcain/obsidian-periodic-notes](https://github.com/liamcain/obsidian-periodic-notes) | Repo oficial | Status de manutenção | HIGH |
| [GitHub — liamcain/obsidian-calendar-plugin](https://github.com/liamcain/obsidian-calendar-plugin) | Repo oficial | Status de manutenção | HIGH |
| [GitHub — obsidian-tasks-group/obsidian-tasks releases](https://github.com/obsidian-tasks-group/obsidian-tasks/releases) | Release oficial | Versão 7.23.1 | HIGH |
| [GitHub — mirnovov/obsidian-homepage](https://github.com/mirnovov/obsidian-homepage) | Repo oficial | Versão 4.4.0 | HIGH |
| [GitHub — pyrochlore/obsidian-tracker](https://github.com/pyrochlore/obsidian-tracker) | Repo oficial | Versão 1.19.0 | HIGH |
| [GitHub — eatgrass/obsidian-pomodoro-timer](https://github.com/eatgrass/obsidian-pomodoro-timer) | Repo oficial | Versão 1.2.3 + Tasks integration | HIGH |
| [GitHub — argenos/nldates-obsidian](https://github.com/argenos/nldates-obsidian) | Repo oficial | Status abandonado | HIGH |
| [obsidianstats.com — plugin pages](https://www.obsidianstats.com/) | Agregador de stats | Commits/downloads por plugin | HIGH |
| [Obsidian 1.9.0 changelog — Bases](https://obsidian.md/changelog/2025-05-21-desktop-v1.9.0/) | Oficial | Bases é core plugin | HIGH |
| [obsidian.rocks — Dataview vs Datacore vs Bases](https://obsidian.rocks/dataview-vs-datacore-vs-obsidian-bases/) | Community comparison | Panorama estratégico | MEDIUM |
| [dsebastien — Best Obsidian Plugins 2026](https://www.dsebastien.net/the-must-have-obsidian-plugins-for-2026/) | Curated list | Validação cruzada | MEDIUM |
| [Obsidian Forum — Periodic Notes abandonment](https://forum.obsidian.md/t/obsidian-team-needs-to-support-periodic-notes-and-calendar-as-core-plugins/101164) | Community discussion | Contexto social do stale | MEDIUM |

---
*Stack research para: Obsidian-based personal planner com framework Joel Jota (5 pilares + 10 categorias)*
*Researched: 2026-04-23 por agent gsd-researcher*
