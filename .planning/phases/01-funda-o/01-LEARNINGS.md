---
phase: 1
phase_name: "Fundação — Vault skeleton + Windows LongPaths + GitHub privado + 10 plugins"
project: "Agente Pessoal Obsidian"
generated: "2026-04-26"
counts:
  decisions: 12
  lessons: 4
  patterns: 6
  surprises: 4
missing_artifacts:
  - "01-02-SUMMARY.md (Plan 02 — Windows + GitHub setup, executado manualmente em 2026-04-23)"
  - "01-03-SUMMARY.md (Plan 03 — Plugins install + config, executado manualmente em 2026-04-23)"
  - "01-VERIFICATION.md (verificação cross-plan da fase)"
  - "01-UAT.md (UAT formal não executado — usuário validou via uso real em Phase 2/3)"
---

# Phase 1 Learnings: Fundação

> **Cobertura:** este LEARNINGS.md foi gerado com 1 dos 3 SUMMARYs disponíveis (apenas `01-01-SUMMARY.md`). Plans 02 e 03 não têm SUMMARY próprio porque foram executados manualmente pelo Weslley com confirmação via resume-signals — outputs ficaram em STATE.md, MEMORY.md, e nos próprios commits. Learnings dos plans 02/03 foram extraídos do PLAN.md + STATE.md + MEMORY.md.

## Decisions

### `.gitkeep` em todos os 13 folders vazios desde o início
Decidido em 01-01-PLAN.md Task 1: cada um dos 13 top-level folders (`00 - Dashboard` ... `10 - Projetos` + `Templates` + `_private`) recebe um arquivo `.gitkeep` placeholder de 0 bytes.

**Rationale:** sem placeholder, `git init` (Plan 02) ignora folders vazios silenciosamente. Resultado seria perda de structure ao restaurar de clone. `.gitkeep` é convenção universal, zero bytes, zero ambiguidade.
**Source:** 01-01-PLAN.md Task 1 + 01-01-SUMMARY.md "Padrão aprendido"

---

### Naming convention: PT-BR + snake_case + espaço " - " separador
Decidido em 01-01-PLAN.md interfaces: folders de topo com espaço (`00 - Dashboard`, `01 - Diário`), schema YAML com `snake_case` (`pilar_media`, `tarefas_log`), datas ISO 8601 (`2026-04-23`).

**Rationale:** consistência com planner físico do Joel Jota (PT-BR + numeração) + previsibilidade pra parsers (snake_case sempre, sem espaços em chaves YAML). Espaço no nome de folder é OK porque Obsidian + git suportam; chaves YAML não suportariam.
**Source:** 01-01-PLAN.md interfaces + _meta/schema.md "Convenção" (criado nesta fase)

---

### `_meta/` separado de conteúdo do vault — documentação operacional NÃO é nota
Decidido em 01-01-PLAN.md Task 2: criar folder `_meta/` paralelo aos 13 numerados, contendo apenas metadata operacional (`README.md`, `plugins.md`, `setup.md`, `backup.md`).

**Rationale:** `00 - Dashboard/` ... `10 - Projetos/` são CONTEÚDO do planner (consumido pelo usuário diariamente). `_meta/` documenta COMO o vault é construído (consumido por Claude/Weslley em sessões de manutenção, NÃO no uso diário). Misturar os dois polui a sidebar e quebra a intuição "tudo aqui é pra revisar/refletir".
**Source:** 01-01-PLAN.md interfaces (linha "_meta/ # Project metadata (NÃO vault conteúdo)")

---

### `_private/` no .gitignore desde o dia 1 — privacy é hard rule
Decidido em 01-01-PLAN.md Task 3: `.gitignore` exclui `_private/` e `_private/**` desde o primeiro commit. Pitfall #14 (privacy leak) referenciado no comentário do `.gitignore`.

**Rationale:** uma vez que reflexões sensíveis são push'd pra GitHub (mesmo privado), assumir vazadas — search engines indexam, wayback archiva, leaks acontecem. Trampolim: configurar exclusão ANTES de qualquer commit potencialmente problemático. "Sagrado" no doc.
**Source:** 01-01-PLAN.md Task 3 (acceptance criteria + justificativa por bloco)

---

### Vault em path local (E:\) — JAMAIS em cloud sync (OneDrive/iCloud/Dropbox)
Decidido em 01-01 e codificado em `_meta/backup.md` regra #1: vault path = `E:\Claude Code\Agente Pessoal - Obsidian\Vault\` (drive local não-sincronizado).

**Rationale:** pitfall #5 (sync corruption). iCloud Windows duplica arquivos até 56x, OneDrive renomeia em conflitos (`(1).md`, `(2).md`), Dropbox cria `hostname-suffix`. Único backup permitido = `obsidian-git → GitHub PRIVADO`. Trade-off: single-device até instalar Obsidian em outra máquina (aceito conscientemente — Weslley single-device por enquanto).
**Source:** 01-01-PLAN.md interfaces + Vault/_meta/backup.md (criado nesta fase)

---

### Windows LongPaths via Registry (Opção A) — não Group Policy
Decidido em 01-02-PLAN.md Task 1: instrução padrão usa `reg add HKLM\SYSTEM\CurrentControlSet\Control\FileSystem LongPathsEnabled` em PowerShell admin + reboot.

**Rationale:** Registry é mais rápido (30s + reboot vs Group Policy GUI navigation + reboot). Group Policy é Opção B alternativa pra quem não gosta de CLI. Ambos resolvem MAX_PATH=260 (pitfall #11). Sem isso, paths >260 chars quebram silenciosamente em git/Explorer/backup tools.
**Source:** 01-02-PLAN.md Task 1 + Vault/_meta/setup.md §1

---

### GitHub repo PRIVATE com double-check obrigatório
Decidido em 01-02-PLAN.md Task 2: criar repo no GitHub com `Visibility: Private` + verificação dupla (badge 🔒 visível + tentativa em aba anônima retorna 404).

**Rationale:** Public visibility = 6+ meses de reflexões pessoais world-readable + indexáveis. Custo de erro = irreversível na prática (search engines, wayback). Por isso double-check obrigatório, não-negociável.
**Source:** 01-02-PLAN.md Task 2 (resume-signal exige confirmação textual "Repo PRIVATE criado")

---

### `git init -b main` (não master) — convenção GitHub atual
Decidido em 01-02-PLAN.md Task 3: comando inicial usa flag `-b main` pra branch padrão.

**Rationale:** GitHub mudou default pra `main` em 2020. `master` ainda funciona mas gera warning + inconsistência com novos repos. `-b main` é zero-friction.
**Source:** 01-02-PLAN.md Task 3 (comando exato)

---

### 10 plugins canônicos verificados em STACK.md (não chute)
Decidido em 01-03-PLAN.md interfaces: lista exata de 10 Community IDs, autores, versões verificadas em GitHub releases na research phase (`research/STACK.md` 2026-04-23).

**Rationale:** plugins têm stale state diferente — 4 estão "ABANDONADO (funcional)", 1 em "LIMBO", 5 ATIVOS. Cada um tem fallback documentado em `_meta/plugins.md` antes de ser instalado. Pitfall #3 (plugin abandonment) mitigado estruturalmente.
**Source:** 01-03-PLAN.md interfaces + Vault/_meta/plugins.md (tabela completa)

---

### Desabilitar core "Templates" pra evitar conflito com Templater
Decidido em 01-03-PLAN.md Task 1: ao ativar Community plugins, também desabilitar core plugin "Templates" (não confundir com `Templates/` folder do vault).

**Rationale:** core Templates é primitivo (sem folder mapping, sem condicional, sem user functions) e potencialmente intercepta atalhos que Templater quer usar. Templater cobre tudo que core Templates faz, com superset de features.
**Source:** 01-03-PLAN.md Task 1 (acceptance criteria opcional mas recomendado)

---

### Templater "Trigger on new file creation" = ON é CRÍTICO pra Phase 2
Decidido em 01-03-PLAN.md Task 3 Parte A: configurar `trigger_on_file_creation: true` antes de fechar Phase 1.

**Rationale:** sem isso, criar nova daily não dispara Templater, e folder template mapping (configurado em Phase 2) não funciona. Resulta em daily vazio em vez de daily com frontmatter pré-populado. Endereça FOUND-04 explicitamente.
**Source:** 01-03-PLAN.md Task 3 Parte A (marcado "**CRÍTICO**" no plan)

---

### Homepage em Reading view (não Edit) — Dataview só renderiza em Reading
Decidido em 01-03-PLAN.md Task 3 Parte B: configurar Homepage com `Open in: Reading view`.

**Rationale:** DataviewJS blocks só renderizam em Reading mode. Em Edit/Source mode, o usuário vê o código `dv.paragraph(...)` literal em vez do output. Phase 3 dashboard inteiro depende de Dataview funcionando — fazer essa config errada na Phase 1 = quebrar Phase 3 silenciosamente.
**Source:** 01-03-PLAN.md Task 3 Parte B (marcado "**CRÍTICO**" no plan)

---

## Lessons

### `.gitkeep` em folder vazio é a única forma de o git rastrear desde o `git init`
Aprendido durante 01-01 quando o plano explicitou que sem placeholder, folders vazios desaparecem do backup. Tentar listar como diretório no `.gitignore` ou comentário não funciona — git só conhece arquivos.

**Context:** projetos novos com structure pre-definida (skeleton-first approach) precisam dessa técnica pra evitar perda silenciosa em clone fresh.
**Source:** 01-01-PLAN.md Task 1 justificativa "Por que `.gitkeep` e não `README.md`"

---

### Plugins Obsidian community: status manutenção é mais importante que versão
Aprendido na research que precedeu 01-03: `Calendar` (Liam Cain) está há 5 anos sem update mas funcional, enquanto plugins novos com versão alta podem não ter base instalada. Decisão: usar plugin abandonado MAS com fallback documentado é mais seguro que plugin novo sem track record.

**Context:** ecossistema Obsidian tem ~1.5k plugins; ~30% recebem update no último ano; alternativas existem mas exigem migração de config. Trade-off explícito feito em `_meta/plugins.md`.
**Source:** _meta/plugins.md tabela "Status manutenção" + STATE.md "Pitfalls top 5 críticos #5 Plugin abandonment"

---

### Cabeça de Campeão é de Ducasse, NÃO Joel Jota — anti-alucinação estrutural
Aprendido durante research da Phase 0 (registrado em MEMORY.md): `Cabeça de Campeão` é livro do François Ducasse; Joel Jota fez prefácio da edição brasileira. Brief inicial atribuía erroneamente.

**Context:** mesmo com prompt bem definido, autoria de livro só é confiável via filesystem check (`08 - Livros/<livro>.md` com frontmatter `autor:`). Phase 4 coach precisa fazer Glob+Read antes de citar — regra de prompt sozinha não basta.
**Source:** MEMORY.md "🔒 Anti-alucinação estrutural — case 'Cabeça de Campeão'"

---

### `_private/` ser `.gitignore`d desde o commit zero — não pode ser retrofit
Aprendido em 01-01: gitignore tem que estar no PRIMEIRO commit. Adicionar depois não remove o que já foi pushed; precisa `git filter-branch` ou `bfg-repo-cleaner` + force-push + rotate de qualquer credencial mencionada.

**Context:** privacy é gate hard, não soft. "Eu coloco depois" = garantia de leak.
**Source:** _meta/backup.md "Recovery" + 01-01-PLAN.md Task 3

---

## Patterns

### Skeleton-first: criar structure inteira antes de qualquer conteúdo
Padrão usado em 01-01: criar 13 folders + `_meta/` + `.gitkeep` + `.gitignore` ANTES de qualquer arquivo de conteúdo. Phase 2 só preenche o que skeleton já reservou.

**When to use:** projetos com structure conhecida e regular (planner físico → vault digital). NÃO usar quando structure ainda está sendo descoberta — over-commits prematuros.
**Source:** 01-01-PLAN.md `<objective>` "Output: Vault/ completo com 13 folders ... antes de qualquer outro plano funcionar"

---

### `_meta/<nome>.md` pra documentação operacional do projeto
Padrão estabelecido em 01-01 e expandido nas fases seguintes: cada decisão arquitetural ganha doc em `_meta/`. Phase 1 criou `README.md`, `plugins.md`, `setup.md`, `backup.md`. Phase 2 expandiu com `schema.md`, `rubric.md`, `fields.md`. Phase 3 com `calibration.md`, `time-logging.md`, `tracking-integration.md`. UX-Iteration-2 com `release-notes/`.

**When to use:** sempre que uma decisão tem WHY que precisa sobreviver à pessoa que decidiu. Curto-circuita "por que isso está assim?" em sessões futuras.
**Source:** 01-01-PLAN.md Task 2 + _meta/README.md "Arquivos futuros (Phase 2+)"

---

### Manual checkpoints com resume-signal textual em plans não-autônomos
Padrão usado em 01-02 e 01-03: cada Task tem `resume-signal` que define a string EXATA que Weslley digita pra confirmar conclusão (ex: "LongPaths OK", "Repo PRIVATE criado: {URL}", "Push OK — confirmado privado").

**When to use:** sempre que Claude depende de ação manual e precisa saber se passou. Evita ambiguidade ("já fiz" vs "fiz parcial"). String específica força usuário a verificar antes de afirmar.
**Source:** 01-02-PLAN.md + 01-03-PLAN.md (todos os Tasks com `<resume-signal>`)

---

### Double-check de privacy via aba anônima
Padrão estabelecido em 01-02 Task 4: pra confirmar que GitHub repo é Private, abrir URL em aba anônima — deve retornar 404 ou "sign in". Se renderizar = é Public e está vazado.

**When to use:** qualquer hard rule de privacidade. Visibility setting na UI pode mentir (cache, edge case). Single source of truth = "eu sou um anônimo, eu vejo isso?".
**Source:** 01-02-PLAN.md Task 4 verification 3

---

### `.gitignore` com comentários explicando WHY (não só WHAT)
Padrão usado em 01-01-PLAN.md Task 3 conteúdo do `.gitignore`: cada bloco tem comentário explicando WHY é ignorado, com referência ao pitfall.

**When to use:** sempre que `.gitignore` recebe pattern não-óbvio. Sem comentário, em 6 meses ninguém lembra por que excluiu, e remove "limpando" — quebra invariante.
**Source:** Vault/.gitignore (criado em 01-01-PLAN.md Task 3)

---

### Tabela com Status manutenção + Fallback pra cada dependência crítica
Padrão estabelecido em `_meta/plugins.md`: cada plugin tem coluna "Status manutenção" (ATIVO/ABANDONADO/LIMBO) + "Fallback" se quebrar.

**When to use:** dependências externas com risco de abandonment (Obsidian plugins, NPM packages obsoletos, libs maintained-by-one-person). Antecipa o "e se quebrar?" sem virar over-engineering — é só uma coluna.
**Source:** _meta/plugins.md tabela "Status manutenção" + research/PITFALLS.md pitfall #3

---

## Surprises

### Periodic Notes v1.0.0-beta1 — campo `folder` não interpola variáveis Moment
Surpresa descoberta durante teste real de Phase 1 (registrado em MEMORY.md e thread `periodic-notes-folder-no-interpola.md`): ao configurar Periodic Notes com `folder: "01 - Diário/YYYY/MM-MMMM"`, plugin cria pasta literal `YYYY/MM-MMMM` em vez de `2026/04-Abril`.

**Impact:** quebrou organização hierárquica das dailies por ano/mês até descobrir. Fix: mover variáveis Moment pro `format` (que SIM interpola) e deixar `folder` literal: `format: "YYYY/MM-MMMM/YYYY-MM-DD"`, `folder: "01 - Diário"`. Sem essa descoberta, dailies ficavam todas em uma pasta literal `YYYY/MM-MMMM/` no root.
**Source:** MEMORY.md "Periodic Notes v1.0.0-beta1 — folder não interpola" + .planning/threads/periodic-notes-folder-no-interpola.md

---

### Plugin manifest.id pode divergir do nome da pasta
Surpresa descoberta durante 01-03 instalação: ao baixar `obsidian-calendar-plugin` via GitHub releases/latest, o `manifest.json` declara `"id": "calendar-beta"` (não `calendar`). Se a pasta em `.obsidian/plugins/` tiver nome diferente do `manifest.id`, o plugin não carrega — silencioso.

**Impact:** plugin Calendar listado em `community-plugins.json` mas não funcionando. Fix: validar `manifest.id` antes de listar e renomear pasta se necessário (`mv calendar/ calendar-beta/`). Sem essa descoberta, sidebar widget Calendar simplesmente não aparecia, sem mensagem de erro.
**Source:** MEMORY.md "Plugin manifest.id pode divergir do nome da pasta" + .planning/threads/obsidian-plugin-id-vs-folder.md

---

### Obsidian abre vault sem `.obsidian/` existente — cria automaticamente
Surpresa esperada-mas-confirmada em 01-03 Task 1: ao apontar Obsidian pra `Vault/` (que tinha 13 folders + `.gitignore` + `_meta/` mas SEM `.obsidian/`), Obsidian abriu sem erro e criou `.obsidian/` populado com `app.json`, `community-plugins.json` (vazio), `core-plugins.json`, `appearance.json`.

**Impact:** confirmou que skeleton skeleton-first do 01-01 é suficiente — não precisa pre-criar `.obsidian/`. Phase 1 não precisou commitar arquivos de plugin antes do plugin existir. Lesson positiva: deixar Obsidian criar o que ele controla.
**Source:** 01-03-PLAN.md Task 1 acceptance criteria "Vault/.obsidian/ existe e contém community-plugins.json (ainda que vazio)"

---

### `obsidian-git` plugin auto-detecta `.git/` existente — não exige re-init
Surpresa de design positiva em 01-03 Task 3: instalar e configurar `obsidian-git` plugin DEPOIS de Plan 02 já ter feito `git init` + `git remote add origin` no `Vault/`. Plugin detectou `.git/` automaticamente e usou o remote existente sem reconfigurar.

**Impact:** validou a separação intencional de Plan 02 (infraestrutura git) e Plan 03 (plugin obsidian-git que CONSOME a infra). Padrão "infra primeiro, app depois" funcionou — eles podem rodar em paralelo (wave 2) ou em qualquer ordem dentro da wave.
**Source:** 01-03-PLAN.md `<output>` "Pattern learned: 'obsidian-git auto-detecta .git existing repo'"

---

*Generated 2026-04-26 by /gsd-extract_learnings (cobertura parcial — 1 de 3 SUMMARYs disponíveis; learnings dos plans 02/03 extraídos de PLAN.md + STATE.md + MEMORY.md + threads).*
