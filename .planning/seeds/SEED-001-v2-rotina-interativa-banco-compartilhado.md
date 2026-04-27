---
id: SEED-001
status: dormant
planted: 2026-04-26
planted_during: UX-Iteration-2 (pós Phase 3, pré Phase 4)
trigger_when: 4 semanas de uso real do MVP V1 da rotina interativa OU sinal de fricção do usuário ("queria que aprendesse com o que eu uso", "tô digitando o mesmo override toda vez", "podia ser mais inteligente") OU início do planejamento da Phase 5 (Análise / Pattern Detection)
scope: Medium
---

# SEED-001: V2 da rotina interativa — banco compartilhado + auto-promote inteligente

## Why This Matters

O MVP V1 (entregue na UX Iteration #2 em 2026-04-26, commit `63d6cff`) resolveu o problema imediato de "rotina read-only que não dá pra editar nem dropdown de sugestões". Cada bloco virou um `INPUT[suggester(option(A), option(B), allowOther):agenda_real.hHH_MM]` com 3-5 sugestões hardcoded por horário e opção de texto livre.

**Limitação reconhecida do V1:**
- Sugestões são **estáticas** — definidas no template, não mudam com o uso real
- Não tem **memória do que o usuário escolhe** — se Weslley sempre faz "Academia" no 06:30, o sistema não promove pra #1 sugestão automaticamente
- Texto livre via `allowOther` **não vira sugestão futura** — fica preso só na daily daquele dia
- Defaults assumem **dia útil** sempre — fim de semana exige override manual completo
- **18 listas paralelas** no template = manutenção tediosa quando precisar adicionar/remover atividade

**Por que isso importa estrategicamente:**

1. **Coach Phase 4 (`/loga`, `/prepara meu dia`) precisa de dados ricos.** Hoje, `agenda_real` armazena strings livres. Pra coach detectar padrões ("Saúde ≥4 toda vez que você fez Academia às 06:30"), precisa que as atividades sejam **estáveis e categorizáveis** — banco compartilhado dá isso.

2. **Pattern detection Phase 5 cruza atividade × horário × pilar score.** Sem banco unificado, atividades com mesma intenção mas digitadas diferente ("Academia" vs "academia" vs "treino") são tratadas como atividades distintas — quebra a estatística.

3. **Friction debt acumula com tempo.** Em 4 semanas (28 dailies), ler 18 dropdowns hardcoded × 28 dias = experiência cansativa quando 80% do que aparece nunca foi escolhido. Banco dinâmico ranqueado por frequência elimina o ruído.

4. **`allowOther` é um sinal de gap no banco.** Cada vez que Weslley digita texto livre, é uma atividade que poderia estar pré-cadastrada e melhorar a UX da próxima vez. Sem auto-feed do banco, esse sinal se perde.

## When to Surface

**Trigger:** ver `trigger_when` no frontmatter.

Esta seed deve ser apresentada durante `/gsd-new-milestone` (ou `/gsd-add-phase` ad-hoc) quando QUALQUER um dos sinais aparecer:

- ✅ **4 semanas de uso real** — verificável via `Glob "Agente Pessoal - Obsidian/01 - Diário/**/*.md" | wc -l ≥ 28` (assumindo 1 daily/dia). Se chegou nesse threshold E `agenda_real` aparece preenchido (não-default) em ≥50% delas → escalar.
- ✅ **Sinal de fricção do usuário** — palavras-chave: "aprendesse", "padrão automático", "mesma coisa toda vez", "podia ser mais esperto", "lista cheia de coisa que não uso", "fim de semana com lista de dia útil tá ruim"
- ✅ **Início Phase 5 (Análise / Pattern Detection)** — V2 do `agenda_real` é input direto. Sem V2, Phase 5 ainda funciona mas perde uma dimensão de análise (rotina × pilar score)

**Não-trigger (ignorar se acontecer):**
- ❌ Weslley pedindo CSS/visual ajuste no dropdown — V2 não é sobre visual
- ❌ Phase 4 sozinha — Phase 4 (Coach) consome `agenda_real` mas funciona com V1 também

## Scope Estimate

**Medium** — uma fase ou duas. Estimativa breakdown:

- **CRIAR `Banco-Atividades.md`** (~30min): YAML com array de atividades. Cada uma com `nome`, `horario_tipico` (manhã/tarde/noite/qualquer), `tags` (`saude`, `trabalho`, `lazer` etc), `frequencia_tipica` (alta/média/baixa). Seed inicial = ~50 atividades migradas das listas hardcoded do V1.
- **DataviewJS auto-promote** (~1-2h): query analisa últimas 30 dailies, conta frequência por bloco, ranqueia. Output consumido pelo `optionQuery` do Meta Bind suggester.
- **Migrar `TPL-Diario.md`** (~30min): cada `INPUT[suggester(option(A), option(B), ...)]` vira `INPUT[suggester(optionQuery(<dataview>)):agenda_real.hHH_MM]`.
- **Templater condicional dia-útil/fim-semana** (~30min): `<%* if (dia_semana in [sabado, domingo]) include rotina-fim-semana else rotina-dia-util %>`.
- **Auto-feed do banco** (~1h): hook (DataviewJS ou plugin script) que detecta nova string em `agenda_real.h*` que não está no banco → adiciona com flag `auto_added: true`.
- **Schema docs update** (~15min): `_meta/schema.md` e `_meta/fields.md` documentando `Banco-Atividades.md` + mudanças em `agenda_real`.
- **Dashboard "Padrões da rotina"** (~1h, opcional): DataviewJS heatmap atividade × dia da semana × frequência. Pode ser sub-página de `Mes-Atual.md`.

**Total**: ~5-7h de execução (1 fase). Não dá pra ser "small" porque envolve query DataviewJS + Templater condicional + migração de template + criação de banco. Mas também não é "large" — é um aumento focado, não reescrita.

## Breadcrumbs

### Código atual (V1) — base do V2

- `Agente Pessoal - Obsidian/Templates/TPL-Diario.md` — seção "## ⏰ Horários do dia (interativo)" linhas ~85-130. Frontmatter `agenda_real.{h06_00...h22_30}` (18 chaves).
- `Agente Pessoal - Obsidian/01 - Diário/2026/04-Abril/2026-04-24.md` — daily migrada (referência de uso real).
- `Agente Pessoal - Obsidian/05 - Rotinas/Rotina-Dia-Util.md` — rotina padrão dia útil (V1 estática).
- `Agente Pessoal - Obsidian/05 - Rotinas/Rotina-Fim-Semana.md` — rotina fim semana (V1 estática, hoje não-conectada).
- `Agente Pessoal - Obsidian/_meta/schema.md` — seção 1 "Daily Note" → frontmatter `agenda_real`.
- `Agente Pessoal - Obsidian/_meta/fields.md` — entry `agenda_real.*` com motivação completa.

### Documentação relevante

- `Agente Pessoal - Obsidian/_meta/release-notes/2026-04-25-ux-iteration-2.md` — release notes V1 com seção "V2 (depois — lock pra outra micro-iteração)".
- `.planning/threads/meta-bind-nested-properties-pattern.md` — extensão UX-Iteration-2 documenta sintaxe `suggester` + `allowOther` + chaves `h<HH>_<MM>` (V2 vai usar isso + `optionQuery`).
- `.planning/threads/obsidian-git-auto-backup-vs-commits-semanticos.md` — relevante porque V2 implica muitos commits de auto-feed do banco que vão ser intercalados com auto-backups.

### Consumidores downstream

- **Phase 4 (Coach anti-alucinação + protocolo socrático)** — `/loga` e `/prepara meu dia` consomem `agenda_real` planejado vs executado. V2 não bloqueia Phase 4, mas enriquece.
- **Phase 5 (Análise / Pattern Detection)** — pattern detection cruza atividade × horário × pilar score. V2 é input direto.
- **Phase 6 extra C/D** (extras aprovados no brainstorm) — possível "mini-coach do dia seguinte" que sugere rotina baseada em padrão histórico = V2 + Phase 4 combinados.

### Pesquisa pré-V2 (a fazer no começo da implementação)

- Confirmar sintaxe `optionQuery(<dataview>)` do Meta Bind suggester — doc oficial: https://www.moritzjung.dev/obsidian-meta-bind-plugin-docs/reference/inputfields/suggester/ (já vimos `optionQuery(#example-note)` mas não testamos com query custom).
- Avaliar performance: query DataviewJS rodando 18× por daily (1 por bloco) pode laggar — cachear resultado por sessão Obsidian?
- Decisão arquitetural: banco como YAML frontmatter (mais simples, queryable via Dataview) vs JSON em `_meta/` (mais estruturado, não queryable via Dataview).

## Notes

### Por que não fizemos isso já no V1

Decisão consciente em 2026-04-26 (UX Iteration #2): o feedback do usuário foi específico e imediato ("não consigo escrever na rotina, queria sugestões"). MVP rápido com hardcoded options resolveu o problema PRINCIPAL em ~2h de trabalho. Banco compartilhado + auto-promote teria sido 5-7h pra mesma resolução do problema imediato — over-engineering pré-uso real.

**Lição:** seguir o que o usuário PRECISA agora, não o que parece "mais correto". V1 manda dados pra produção, V2 escala QUANDO os dados mostram que precisa.

### Sinais de que JÁ é hora de escalar

- Weslley voltou e disse algo como "quase nunca uso a opção 1, mas preciso scrollar antes de digitar minha"
- `agenda_real` tem 5+ overrides (`allowOther`) na mesma daily → fricção alta
- `agenda_real` mostra mesmo override texto livre repetido em 5+ dailies → atividade que devia estar no banco
- Phase 5 começou e Pattern Detection está retornando ruído por strings inconsistentes ("academia" vs "Academia" vs "Treino na academia")

### Sinais de que NÃO é hora ainda

- Weslley usa V1 sem reclamar
- `agenda_real` raramente é editado (= MVP é "good enough")
- Outras prioridades de roadmap aparecem antes (Phase 4 cobrir gap maior de valor)

### Risco de "esquecer" esta seed

Esta seed é **mid-priority**. Risco real: ficar dormente eternamente porque outras urgências sempre aparecem. Mitigação: `/gsd-new-milestone` deve scan-ar `.planning/seeds/` automaticamente — quando milestone novo for criado e cobrir Análise/Pattern Detection (Phase 5+), surface esta seed.

Se passar 6 meses dormente sem trigger, revisar manualmente: ainda relevante? V1 ainda em uso? Talvez upgrade pra status `expired` ou `obsolete`.

---

*Plantada em 2026-04-26 durante session-wrap pós UX Iteration #2. Plant-seed disparado por GSD smart capture.*
