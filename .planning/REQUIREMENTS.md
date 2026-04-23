# Requirements: Agente Pessoal Obsidian

**Defined:** 2026-04-23
**Core Value:** Receber coaching real e sem achismo — priorização socrática, tracking calibrado por histórico, análise semanal/mensal fundamentada em livros — enquanto o sistema de capture (planner digital) tem fricção mínima.

## v1 Requirements

Requirements para release inicial. Cada um mapeia para uma fase do roadmap.

### Fundação (FOUND) — Storage, Plugins, Backup, Windows Setup

- [ ] **FOUND-01**: Vault Obsidian instalado em path local (não sincronizado com OneDrive/iCloud/Dropbox) em `E:\Claude Code\Agente Pessoal - Obsidian\Vault\`
- [ ] **FOUND-02**: Windows Registry `LongPathsEnabled=1` configurado para evitar MAX_PATH 260 chars
- [ ] **FOUND-03**: 10 plugins community instalados (Templater, Dataview, Periodic Notes, Calendar, Tasks, Homepage, Tracker, Pomodoro Timer, Natural Language Dates, obsidian-git)
- [ ] **FOUND-04**: Templater configurado com "Trigger on new file = ON" e folder `Templates/`
- [ ] **FOUND-05**: `_meta/plugins.md` documenta cada plugin com: versão atual, status de manutenção, fallback caso quebre
- [ ] **FOUND-06**: obsidian-git apontando para repositório GitHub **privado** (verificação dupla: repo não é público)
- [ ] **FOUND-07**: `.gitignore` exclui `workspace.json`, `_private/`, `.obsidian/cache`
- [ ] **FOUND-08**: Estrutura de folders skeleton criada (00-Dashboard até 10-Projetos + Templates + _meta + _private)

### Templates + Schema (TMPL)

- [ ] **TMPL-01**: `_meta/schema.md` define schema de frontmatter congelado (snake_case PT-BR) — campos obrigatórios do daily: data, pilares (5 sub-campos 0-5), prioridades, pomodoros
- [ ] **TMPL-02**: `_meta/rubric.md` com âncoras concretas 1-5 por pilar (ex: "Saúde 5 = treinei + comi limpo + 7h sono + energia alta")
- [ ] **TMPL-03**: Template diário fiel ao planner físico (Manhã Intencional com 10 perguntas canônicas + 3 Prioridades + Mensagem do Dia + Tarefas + Pessoas + Agenda via Ativa.md + Resumo do Dia com 6 reflexões + Avaliação 5 Pilares)
- [ ] **TMPL-04**: Template semanal planejamento (grid 7 dias × manhã/tarde/noite, projetos, eventos, prazos, metas, evidência de sucesso)
- [ ] **TMPL-05**: Template semanal benchmark (Semanário Alta Performance com 5 pilares × 5 perguntas × 0-5 → ×4 = % de 100; Aprendizado da Semana com 8 perguntas + Revisão Auto Performance 10 áreas)
- [ ] **TMPL-06**: Template mensal planejamento (maiores projetos/eventos, coisas do mês, prazos, frase motivacional, calendário Dataview)
- [ ] **TMPL-07**: Template mensal benchmark (7 perguntas de aprendizado + Metas 6m/1a)
- [ ] **TMPL-08**: Templates usam `_fragments/` para DRY (dia-util vs fim-semana injetado via Templater)
- [ ] **TMPL-09**: Periodic Notes configurado com daily=`01 - Diário/YYYY/MM-MMMM/YYYY-MM-DD`, weekly=`02 - Semanal/GGGG/GGGG-[W]WW`, monthly=`03 - Mensal/YYYY/YYYY-MM`

### Rotinas Fixas (RTN)

- [ ] **RTN-01**: `05 - Rotinas/Rotina-Dia-Util.md` com horários padrão do dia útil (editável)
- [ ] **RTN-02**: `05 - Rotinas/Rotina-Fim-Semana.md` com rotina do fim de semana
- [ ] **RTN-03**: `05 - Rotinas/Ativa.md` como ponteiro (`![[Rotina-Dia-Util]]`) — trocar rotina = editar 1 linha
- [ ] **RTN-04**: Template diário embeda `![[05 - Rotinas/Ativa]]` no bloco Agenda (fora de callouts)
- [ ] **RTN-05**: Templater auto-detecta dia da semana e escolhe rotina correta

### Banco de Frases (FRASE)

- [ ] **FRASE-01**: `07 - Frases/Banco-Frases.md` com ~50 frases seed (Joel Jota + autores afins da lista do planner físico)
- [ ] **FRASE-02**: Rotação automática por `dayOfYear % length` — cada dia tem 1 frase
- [ ] **FRASE-03**: Cada frase tem metadados: autor, livro/fonte (para rastreabilidade)
- [ ] **FRASE-04**: Template diário tem campo `> [!quote] Frase do dia` editável (permite sobrescrever a rotativa)

### Dashboard (DASH)

- [ ] **DASH-01**: `00 - Dashboard/Home.md` **leve** — saudação com data/nome + frase rotativa + bloco HOJE (3 prioridades + tarefas) + embed Ativa.md + links para sub-dashboards. **Zero Tracker charts aqui** (performance)
- [ ] **DASH-02**: `00 - Dashboard/Semana-Atual.md` **pesado** com score médio 5 pilares da semana, agregação, gráfico Tracker
- [ ] **DASH-03**: `00 - Dashboard/Mes-Atual.md` **pesado** com evolução mês atual, progresso metas, gráficos
- [ ] **DASH-04**: Homepage plugin configurado para abrir `Home.md` em Reading mode ao abrir Obsidian
- [ ] **DASH-05**: Bloco HOJE usa Dataview query `FROM "01 - Diário/YYYY"` (scoped) com LIMIT para evitar freeze
- [ ] **DASH-06**: Tarefas pendentes: Tasks plugin query `not done AND path includes "01 - Diário"` com limit 100
- [ ] **DASH-07**: Pomodoro Timer integrado — iniciar timer em tarefa da nota do dia
- [ ] **DASH-08**: Widget de calendário (Calendar plugin) clicável → abre nota do dia
- [ ] **DASH-09**: Todas as queries Dataview são empty-state-safe (Day 0 sem daily notes não renderiza erro)

### Time Tracking (TRACK)

- [ ] **TRACK-01**: Pomodoro Timer escreve `pomodoros: N` no frontmatter da nota do dia ao finalizar sessão
- [ ] **TRACK-02**: Seção "Log de Tempo" no template diário permite registro manual (formato `HH:MM–HH:MM → tarefa | estimei Nmin | real Nmin`)
- [ ] **TRACK-03**: Frontmatter `tarefas_log` com estrutura: nome, tipo, tempo_estimado, tempo_real, energia, dificuldade, concluida
- [ ] **TRACK-04**: Comando `/loga HH:MM-HH:MM <descrição>` preenche log via Claude
- [ ] **TRACK-05**: `_meta/calibration.md` mantém histórico de mediana de tempo_real por tipo de tarefa

### Coach de Priorização (COACH)

- [ ] **COACH-01**: Prompt documentado em `Templates/TPL-Coach-Prompt.md` implementando protocolo socrático de 6 passos (leio contexto → pergunto → estimo → questiono red flags → paro se falta dado → registro)
- [ ] **COACH-02**: Comando `prepara meu dia` — lê ontem + semana + metas, propõe 3 prioridades com justificativa antes de preencher
- [ ] **COACH-03**: Coach **estruturalmente** verifica arquivos em `08 - Livros/` (Glob + Read) antes de citar qualquer livro; output inclui filesystem path da nota citada
- [ ] **COACH-04**: Comando `verifica fontes` que re-confirma citações em uma análise
- [ ] **COACH-05**: Coach opera em **2 modos** detectados: `/prepara` (completo, socrático) vs `log` (<5s, zero perguntas para capture rápido)
- [ ] **COACH-06**: Friction budget: máximo 5 perguntas por dia no modo coach
- [ ] **COACH-07**: Regra "< 3 amostras = sem histórico" — coach admite explicitamente quando não tem dados suficientes para estimar
- [ ] **COACH-08**: Feedback loop — quando usuário corrige estimativa, coach escreve correção em `_meta/calibration.md`
- [ ] **COACH-09**: Privacy redaction — coach NUNCA cita conteúdo de `_private/` em outputs/exports

### Livros / Anti-Alucinação (LIVR)

- [ ] **LIVR-01**: `08 - Livros/` criado com 3 livros seed validados: "100% Presente" (Joel Jota), "Atomic Habits" (James Clear), "Deep Work" (Cal Newport)
- [ ] **LIVR-02**: Cada nota de livro tem frontmatter `autor:`, `titulo:`, `lido: true/false` + conceitos-chave + citações com número de página
- [ ] **LIVR-03**: `08 - Livros/_INDEX.md` lista todos os livros com status de leitura (Dataview)
- [ ] **LIVR-04**: Coach só cita livros com `lido: true` (se falso, sinaliza "livro não lido ainda, não posso fundamentar")

### Análise Semanal/Mensal (ANAL)

- [ ] **ANAL-01**: Comando `coach semanal` — lê 7 daily notes da semana, calcula score médio 5 pilares, detecta padrões, compara tempo estimado vs real
- [ ] **ANAL-02**: Gera benchmark semanal (fiel ao planner) com 8 perguntas + sugestões de 3 focos para próxima semana
- [ ] **ANAL-03**: Detecção de **performative completion**: se 3+ prioridades do dia são string-match idênticas dos dias anteriores → flag
- [ ] **ANAL-04**: Detecção de **score inflation**: se std dev de scores em 7 dias < 0.5 → coach questiona ("rubric drift ou crescimento real?")
- [ ] **ANAL-05**: Honestidade check semanal: coach escolhe aleatoriamente 1 dia e pergunta "o que realmente mudou nesse dia?"
- [ ] **ANAL-06**: Comando `coach mensal` — consolida 4 benchmarks semanais + ~30 daily notes, calcula evolução mês vs mês anterior
- [ ] **ANAL-07**: Análise mensal avalia progresso concreto das Metas 6m/1a (% baseado em entregas, não sensação)
- [ ] **ANAL-08**: Análise mensal questiona explicitamente: "essa meta de 6m ainda faz sentido? Se sim, em que ritmo precisa ir?"
- [ ] **ANAL-09**: Calibration review semanal — coach mostra quantas estimativas acertou vs errou, por tipo de tarefa

### Extras do Brainstorm (EXTRA) — 10 aprovados (A-J)

- [ ] **EXTRA-A**: Ritual de fechamento do dia (`/fecha-dia`) — se pilar ≤ 2, coach faz pergunta socrática ancorada em livro. Prepara frase de encerramento
- [ ] **EXTRA-B**: "Pergunta da semana" rotativa — toda segunda o usuário recebe 1 pergunta diferente para refletir durante a semana (~30 perguntas, rotacionadas)
- [ ] **EXTRA-C**: Modo "Temporada" — declara 90 dias com foco específico; coach prioriza temporada nas análises; retrospectiva especial ao fim
- [ ] **EXTRA-D**: Integração wearable — importa JSON/CSV exportado (Apple Health/Garmin/Oura) para cruzar sono com score Saúde. **Sob demanda apenas**
- [ ] **EXTRA-E**: Perguntas "dark" (antifragilidade) mensais — 1x/mês coach faz perguntas duras: "qual desculpa você contou 3 vezes?" "qual decisão adiou por medo?"
- [ ] **EXTRA-F**: `09 - Pessoas/` dashboard — quando foi última vez que falei com X? Quantas vezes apareceu em "Pessoas que preciso falar hoje" sem ação?
- [ ] **EXTRA-G**: Export PDF benchmark mensal (fim do mês) — PDF formatado do benchmark mensal como histórico físico
- [ ] **EXTRA-H**: Rodas duplas Alta × Auto Performance (radar charts) no dashboard — visualização lado a lado dos 5 pilares e 10 categorias
- [ ] **EXTRA-I**: Comparação "você vs você do ano passado" — fim de mês compara com mesmo mês ano anterior. Usa `04 - Benchmarks/Arquivo/` frozen snapshots. **Só funciona a partir do mês 13+**
- [ ] **EXTRA-J**: Time-blocking automático — Dataview puxa 3 prioridades + tarefas + rotina → propõe blocos de tempo. **Só após 3 meses de tracking calibrado**

## v2 Requirements

Deferred para futuro — fora do MVP v1.

### Extensões futuras

- **v2-01**: Obsidian mobile sync (se usuário passar a usar celular com frequência)
- **v2-02**: Migração Dataview → Bases (quando Bases cobrir DataviewJS/inline)
- **v2-03**: Integração Burchard HP6 como lente secundária ao framework Joel Jota
- **v2-04**: Quick-capture widget no dashboard para inbox rápida de ideias
- **v2-05**: Archive flow automático (daily > 2 anos → `Arquivo/`)

## Out of Scope

Explicitamente excluído. Documentado para prevenir scope creep.

| Feature | Razão |
|---------|-------|
| App mobile nativo próprio | Obsidian Mobile sync resolve acesso; app próprio não agrega |
| Multi-usuário / planner compartilhado | Planner é pessoal por filosofia; compartilhar quebra a confidencialidade da reflexão |
| Gamificação (streaks, pontos, badges) | Incentiva otimizar métrica errada, conflita frontalmente com anti-achismo |
| Social features (compartilhar posts do planner) | Transforma reflexão em performance social, diluindo valor |
| AI motivação genérica ("você consegue!") | Toxic positivity documentado em LLMs; vai contra anti-achismo |
| AI toma decisões pelo usuário | Coach propõe, usuário decide. Nunca inverter |
| Notificações push invasivas | Usuário abre quando quer; push quebra foco intencional |
| Storage em cloud sync (OneDrive/iCloud/Dropbox) | Corrupção documentada em vaults Obsidian (iCloud duplica até 56x) |
| Citar livros não lidos | Gera alucinação inevitável; coach só cita livros com `lido: true` |

## Traceability

Mapeamento requisito → fase. Atualizado durante roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 até FOUND-08 | Phase 1 | Pending |
| TMPL-01 até TMPL-09 | Phase 2 | Pending |
| RTN-01 até RTN-05 | Phase 2 | Pending |
| FRASE-01 até FRASE-04 | Phase 2 | Pending |
| LIVR-01 até LIVR-04 | Phase 2 | Pending |
| DASH-01 até DASH-09 | Phase 3 | Pending |
| TRACK-01 até TRACK-05 | Phase 3 | Pending |
| COACH-01 até COACH-09 | Phase 4 | Pending |
| ANAL-01 até ANAL-09 | Phase 5 | Pending |
| EXTRA-A até EXTRA-J | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 72 total
- Mapeados para phases: 72
- Unmapped: 0 ✓

---
*Requirements defined: 2026-04-23*
*Last updated: 2026-04-23 after initial definition*
