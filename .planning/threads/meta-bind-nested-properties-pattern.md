---
slug: meta-bind-nested-properties-pattern
title: Meta Bind nested properties — sliders + suggester com allowOther + prefixo h em chaves YAML numéricas
status: resolved
created: 2026-04-25
updated: 2026-04-26
---

# Thread: Meta Bind nested properties pattern

## Goal

Documentar o pattern de uso do plugin Meta Bind (mProjectsCode/obsidian-meta-bind-plugin) para inputs visuais 0-5 ligados a frontmatter properties **aninhadas** — usado em todo o projeto pra pilares e auto_performance.

## Context

*Pattern descoberto e validado em 2026-04-25 durante Fase 2-UX.*

### Sintaxe que funciona

```markdown
| **Competitividade** | `INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.competitividade]` |
| **Talento**         | `INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.talento]`         |
| **Saúde**           | `INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.saude]`           |
| **Presença**        | `INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.presenca]`        |
| **Performance Emocional** | `INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.performance_emocional]` |
```

**Property aninhada `pilares.competitividade`** funciona — Meta Bind navega pelo objeto YAML.

### Pré-requisitos validados

1. **Frontmatter inicial precisa ter o campo com valor numérico (0)** — não `null`:

```yaml
pilares:
  competitividade: 0   # ← funciona
  talento: 0           # ← funciona
```

NÃO:
```yaml
pilares:
  competitividade: null  # ← slider não posiciona marker
```

Razão: slider precisa de número pra renderizar a posição inicial. Com `null`, fica em estado indefinido (pode aparecer no min mas não atualiza).

2. **Plugin habilitado em `community-plugins.json`** + plugin instalado em `.obsidian/plugins/obsidian-meta-bind-plugin/`

3. **Reading view OU Live Preview** — Meta Bind renderiza nos dois. Source mode mostra só a string crua.

### Comparação com inputs alternativos

| Tipo de input | Sintaxe | UX |
|---|---|---|
| Slider | `INPUT[slider(minValue(0), maxValue(5), addLabels):X]` | Arrastável, 6 marcadores visíveis |
| Number | `INPUT[number(minValue(0), maxValue(12)):X]` | Campo numérico simples |
| Toggle | `INPUT[toggle:X]` | Booleano on/off |
| InlineSelect | `INPUT[inlineSelect(option(0), option(1), ...):X]` | Dropdown inline |

Slider escolhido pra pilares 0-5 por: visual + tátil (arrastar é mais convidativo que digitar) + addLabels mostra a escala completa.

### Outras descobertas

- **Excluded folders**: configurar `excludedFolders: ["Templates"]` no `data.json` impede Meta Bind de processar templates como notas reais (senão aparece warning)
- **syncInterval: 200**: 200ms é o sweet spot — atualiza frontmatter perceptivelmente em tempo real sem laggar

---

## Extensão UX-Iteration-2 (2026-04-26): suggester com allowOther + chaves YAML numéricas

Aplicado em `agenda_real` (rotina interativa por bloco de horário) — 18 campos `INPUT[suggester]` no `TPL-Diario.md`.

### Sintaxe `INPUT[suggester]` com texto livre

```markdown
`INPUT[suggester(option(Acordar - higiene), option(Acordar + alongar 5min), option(Acordar + meditar 10min), allowOther):agenda_real.h06_00]`
```

**Argumentos chave:**
- `option(<texto>)` inline — quantos quiser, separados por vírgula
- `allowOther` — flag-only (sem valor). **Permite usuário digitar texto livre além das opções**. Sem `allowOther`, suggester é rígido: só escolha da lista
- Bind target nested funciona igual ao slider: `agenda_real.h06_00`

### Prefixo `h` em chaves YAML iniciando com dígito (CRÍTICO)

Chaves YAML que começam com dígito (ex: `06:00`, `06_00`) são **inseguras** entre parsers:
- YAML 1.2 trata `06_00` como número com `_` separador (legível como `600`)
- Alguns parsers viram string, outros viram int, outros falham
- Aspas (`"06:00":`) resolve mas Meta Bind precisa lidar com chave com `:`

**Solução adotada:** prefixo `h` (de "hora") em todas as chaves de horário:

```yaml
agenda_real:
  h06_00: "Acordar, higiene"
  h06_30: "Hidratação + exercício leve"
  h22_30: "Dormir"
```

Convenção snake_case mantida (consistente com resto do schema). Sem ambiguidade entre parsers.

### Comparação suggester vs. alternativas (atualização da tabela)

| Tipo de input | Sintaxe | Permite texto livre? | UX |
|---|---|---|---|
| Slider | `INPUT[slider(minValue(0), maxValue(5), addLabels):X]` | N/A (numérico) | Arrastável |
| Number | `INPUT[number(minValue(0), maxValue(12)):X]` | N/A (numérico) | Campo |
| Toggle | `INPUT[toggle:X]` | N/A (bool) | On/off |
| InlineSelect | `INPUT[inlineSelect(option(A), option(B)):X]` | ❌ Não | Dropdown inline |
| Suggester | `INPUT[suggester(option(A), option(B), allowOther):X]` | ✅ Com `allowOther` | Modal fuzzy search + texto livre |

`suggester` escolhido pra `agenda_real` porque: (1) lista contextual de sugestões por horário, (2) permite override com texto livre via `allowOther`, (3) escolha salva em frontmatter pra análise downstream (Phase 4-5 coach).

### References adicionais (UX-Iteration-2)

- Doc oficial suggester: https://www.moritzjung.dev/obsidian-meta-bind-plugin-docs/reference/inputfields/suggester/
- Exemplo confirmado: `https://github.com/mProjectsCode/obsidian-meta-bind-plugin/blob/master/exampleVault/Input Fields/Suggester.md`
- Aplicado em: `Agente Pessoal - Obsidian/Templates/TPL-Diario.md` (seção "## ⏰ Horários do dia (interativo)")
- Frontmatter consumidor: `agenda_real.{h06_00...h22_30}` (18 chaves)
- Commit: `63d6cff feat(ux-iteration-2): rotina interativa + sliders/UI polidos + frases pt-BR`

## References

- Plugin: https://github.com/mProjectsCode/obsidian-meta-bind-plugin
- Versão usada: 1.4.8
- Docs: https://www.moritzjung.dev/obsidian-meta-bind-plugin-docs/
- Arquivo: `Agente Pessoal - Obsidian/Templates/TPL-Diario.md` (linhas 192-206 — bloco Meta Bind)
- Config: `Agente Pessoal - Obsidian/.obsidian/plugins/obsidian-meta-bind-plugin/data.json`
- Commit: `2b569b8 feat(02-ux): Meta Bind sliders 0-5 + rotina editável inline`

## Next Steps

- ✅ Resolvido (pattern aplicado em pilares + humor + energia + sono)
- Aplicar mesmo pattern em **Fase 6 extra H** (Rodas duplas Alta × Auto Performance) — provavelmente Meta Bind sliders pras 10 categorias auto_performance
- Considerar: Meta Bind também tem `INPUT[buttonGroup]` que pode substituir checkboxes Manhã Intencional (D S T Q Q S S) — avaliar UX em Fase 6
- Cuidado: se adicionar Meta Bind em arquivos com Tracker plugin, validar que frontmatter writes do Meta Bind não conflitam com leitura do Tracker
