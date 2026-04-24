# Agente Pessoal Obsidian

## What This Is

Sistema de alta performance pessoal que transporta o **planner físico do Joel Jota** (Método da Alta Performance) para um vault Obsidian nativo, com dashboard integrado, estrutura temporal automatizada e Claude atuando como **coach de priorização** fundamentado em dados reais do usuário e livros validados. Substitui o preenchimento manual cansativo por um fluxo onde o usuário foca em **decidir e refletir**, enquanto a análise quantitativa (scores, padrões semanais/mensais, progresso de metas) é automática.

## Core Value

Receber coaching real e sem achismo — priorização socrática, tracking calibrado por histórico, análise semanal/mensal fundamentada em livros — enquanto o sistema de capture (planner digital) tem fricção mínima.

## Requirements

### Validated

(Nenhum ainda — validar ao longo da execução)

### Active

- [ ] Vault Obsidian estruturado com pastas por data (01-Diário/YYYY/MM, etc.), gerenciamento automático via Periodic Notes
- [ ] Templates fiéis ao planner físico para nível Diário (Manhã Intencional + Horários + Benchmark), Semanal (Planejamento + Benchmark com 5 pilares + Revisão Auto Performance) e Mensal (Planejamento + Benchmark + Metas 6m/1a)
- [ ] Sistema de **rotinas fixas centralizadas** (`05 - Rotinas/Ativa.md` como ponteiro) — permite trocar rotina editando 1 arquivo
- [ ] Dashboard central (`00 - Dashboard/Home.md`) com: saudação + frase rotativa, bloco HOJE (3 prioridades + tarefas), pomodoro integrado, visão Semana/Mês, calendário, gráfico dos 5 pilares
- [ ] Banco de frases motivacionais rotativo (Joel Jota + autores afins da lista do planner) com rotação automática por dia
- [ ] **Coach de Priorização** (protocolo socrático de 6 passos): leio contexto → pergunto por quê/meta/prazo → estimo tempo por histórico → questiono red flags → paro se falta dado → registro
- [ ] **Tracking de Tempo** duplo: pomodoro automático + log manual "de que hora a que hora" → calibração de estimativas após 15+ amostras
- [ ] Comando `prepara meu dia` (manhã) com proposta de 3 prioridades antes de preencher
- [ ] Análise **semanal automática** (domingo): scores 5 pilares, padrões, pontos de atenção, 3 focos próxima semana
- [ ] Análise **mensal automática**: evolução vs mês anterior, progresso metas 6m/1a, ajustes
- [ ] **Fundamentação em livros** (anti-alucinação): toda sugestão ancorada em (A) dados do usuário, (B) nota de livro em 08-Livros/, ou (C) opinião genérica marcada explicitamente
- [ ] 10 extras aprovados do brainstorm (A a J): ritual fechamento do dia, pergunta da semana rotativa, modo temporada 90d, integração wearable, perguntas "dark" antifragilidade, dashboard de pessoas, export PDF benchmark mensal, rodas duplas Alta×Auto Performance, comparação "você vs você do ano passado", time-blocking automático

### Out of Scope

- **App mobile nativo** — Obsidian mobile sync já resolve acesso no celular, app próprio não agrega
- **Multi-usuário / compartilhamento de planner** — planner é estritamente pessoal por filosofia do método
- **Sync com wearable sem pedido explícito** — extra 6.D do brainstorm só quando usuário fornecer dados
- **Rastreamento de hábitos além dos pilares do método** — escopo restrito ao método Joel Jota, sem adicionar sistemas concorrentes

## Context

- **Usuário**: Weslley — já utiliza Obsidian com sucesso em outros projetos, então migração é natural
- **Inspiração**: Planner físico de Joel Jota (Método da Alta Performance) — imagens em `Imagens Planner/` mostram estrutura mensal → semanal → diária com benchmarks entre cada ciclo
- **Filosofia do método (verificada no planner físico)**:
  - **5 Pilares da Alta Performance**: Competitividade, Talento, Saúde, Presença, Performance Emocional (cada um com 5 auto-avaliações no Semanário)
  - **10 Categorias da Auto Performance**: Saúde, Família, Trabalho, Dinheiro, Conhecimento, Mente/Emoções, Relacionamentos, Amor/Parceiro, Tempo, Lazer
  - **Manhã Intencional**: 10 perguntas canônicas do planner + 3 prioridades fixas
  - **Resumo do Dia**: 6 reflexões + avaliação textual dos 5 pilares
- **Autoria de livros — verificado via research** (anti-alucinação interna):
  - Joel Jota: "100% Presente", "Padrões de Alta Performance", "Ultracorajoso", "O Sucesso é Treinável"
  - "Cabeça de Campeão" é de **François Ducasse** (Joel fez prefácio da edição brasileira) — NÃO citar como obra do Joel
  - "Pentagrama da Alta Performance" (Talento → Maestria → Competitividade...) **não foi encontrado no planner físico** — tratar como narrativa auxiliar não-canônica até usuário confirmar fonte
- **Plano aprovado**: `C:\Users\wesll\.claude\plans\use-o-bathc-sempre-distributed-graham.md` — fonte de verdade para escopo
- **Dor atual**: usuário preenche o planner físico mas analisar padrões manualmente é fricção; quer que Claude atue como coach real (não só preenchedor)
- **Princípio anti-achismo**: usuário pediu explicitamente que Claude não invente; só sugira baseado em dados ou livros validados

## Constraints

- **Plataforma**: Obsidian (Windows 11, vault em `E:\Claude Code\Agente Pessoal - Obsidian\Vault`)
- **Plugins**: apenas community plugins consolidados — Templater, Dataview, Periodic Notes, Calendar, Tasks, Homepage, Tracker, Pomodoro (a validar no momento da instalação), Natural Language Dates
- **Controle do projeto**: `.planning/` (GSD) ≠ produto (`Agente Pessoal - Obsidian/`) — pastas separadas
- **Anti-alucinação (hard rule)**:
  - Coach nunca cita livro/autor sem arquivo existente em `Agente Pessoal - Obsidian/08 - Livros/`
  - Coach nunca estima tempo com < 3 amostras sem dizer "sem histórico suficiente"
  - Coach para e pergunta se falta dado, nunca chuta
- **Idioma**: português (PT-BR) — templates, prompts e frases motivacionais

## Key Decisions

| Decisão | Rationale | Outcome |
|---|---|---|
| Granularidade Coarse (6 fases) | Bate com as fases já mapeadas no plano; evita fragmentação | — Pending |
| Mode YOLO | Usuário quer velocidade, só pausamos em bloqueios reais | — Pending |
| Modelo Quality (Opus 4.7) | Projeto pessoal, longevo; qualidade > custo | — Pending |
| Research + Plan Check + Verifier ligados | Projeto é central e deve ser sólido desde a fundação | — Pending |
| Todos os 10 extras (A-J) do brainstorm aprovados | Usuário gostou de todos no brainstorm | — Pending |
| Vault separado de `.planning/` | `.planning` controla projeto GSD, `Vault` é o produto Obsidian | — Pending |
| Coach com protocolo socrático de 6 passos | Anti-achismo é requisito explícito do usuário | — Pending |
| Rotinas via `Ativa.md` como ponteiro | Permite trocar rotina editando 1 arquivo | — Pending |
| **Dashboard decomposto** (Home leve + Semana-Atual + Mes-Atual) | Research: Home único com 6+ blocos Dataview engasga após 1k notas | — Pending (Phase 3) |
| **Vault NÃO em cloud sync** (OneDrive/iCloud/Dropbox) | Research: corrupção documentada (iCloud duplica arquivos Windows); vault em E:\ local | ✓ Resolvido (path local) |
| **Backup via obsidian-git → GitHub PRIVADO** | Single-device Windows: git+GitHub privado é o único backup seguro | — Pending (Phase 1) |
| **Daily frontmatter = single source of truth** | Weekly/monthly = Dataview live. NUNCA duplicar scores (silent drift quebra anti-achismo) | — Pending (Phase 2) |
| **Anti-alucinação ESTRUTURAL** (Glob+Read em 08-Livros, não só prompt) | Research: LLMs alucinam citações 20%+ mesmo com regra prompt. Filesystem check é o gate real | — Pending (Phase 4) |
| **Detecção de performative completion** é objetivo do coach | Research: risco comportamental #1 não é abandono, é score inflation e prioridades copy-paste | — Pending (Phase 5) |
| **Queries Dataview sempre scoped + LIMIT** | Unscoped é #1 causa de freeze no Obsidian (research) | — Pending (Phase 3) |
| **Dataview 0.5.70 no MVP, Bases futuro** | Bases (core plugin Obsidian 1.9+) ainda não cobre DataviewJS/inline fields; ecossistema assume Dataview | — Pending |
| **"Cabeça de Campeão" = Ducasse, NÃO Joel Jota** | Research: Joel fez só prefácio. Coach NUNCA citar como obra do Joel. Lesson learned valida anti-alucinação estrutural | ✓ Documentado |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-23 after initialization*
