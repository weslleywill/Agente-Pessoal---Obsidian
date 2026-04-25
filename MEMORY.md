# MEMORY — Agente Pessoal Obsidian

Aprendizados específicos deste projeto que esquecer custaria retrabalho.

---

## 🐛 Bugs / quirks de plugins Obsidian descobertos

### Periodic Notes v1.0.0-beta1 — `folder` não interpola

**Sintoma:** ao criar daily, plugin cria pasta literal `YYYY/MM-MMMM` em vez de `2026/04-Abril`.

**Causa:** na v1.0.0-beta1 do `liamcain/periodic-notes`, o campo `folder` é tratado como path **literal** (não interpola variáveis Moment). O `format` SIM interpola.

**Solução correta:**
```json
{
  "day": {
    "format": "YYYY/MM-MMMM/YYYY-MM-DD",
    "folder": "01 - Diário"
  }
}
```
NÃO:
```json
{
  "day": {
    "format": "YYYY-MM-DD",
    "folder": "01 - Diário/YYYY/MM-MMMM"
  }
}
```

### Templater `tp.file.include()` — falha com frontmatter

**Sintoma:** ao usar `<% tp.file.include("[[Rotina-Dia-Util]]") %>` dentro de template diário, o frontmatter da rotina é incluído como texto no corpo, quebrando o YAML do daily.

**Solução:** arquivos referenciados via `tp.file.include()` devem ser **SEM frontmatter** (apenas conteúdo). Aplicado às rotinas em `05 - Rotinas/Rotina-Dia-Util.md` e `Rotina-Fim-Semana.md`.

### Embed `![[]]` dentro de callout não atualiza

**Sintoma:** `> [!note]\n> ![[Rotina-Ativa]]` não atualiza quando arquivo embed muda.

**Solução:** colocar embeds **fora** de callouts. DataviewJS dentro de callout funciona normalmente — só transclusion `![[]]` que tem o bug.

### Plugin manifest.id pode divergir do nome da pasta

**Sintoma:** ao baixar `obsidian-calendar-plugin` via releases/latest, o manifest.json declara `"id": "calendar-beta"` (não `calendar`). Se a pasta em `.obsidian/plugins/` tiver nome diferente do `manifest.id`, o plugin não carrega.

**Solução:** validar `manifest.id` antes de listar em `community-plugins.json` e renomear pasta se necessário. Padrão de verificação:
```bash
for dir in .obsidian/plugins/*/; do
  folder=$(basename "$dir")
  id=$(node -e "console.log(require('./$dir/manifest.json').id)")
  [ "$folder" != "$id" ] && echo "MISMATCH: $folder vs $id"
done
```

---

## 🎨 Patterns Meta Bind para inputs visuais

### Slider com property aninhada

```markdown
`INPUT[slider(minValue(0), maxValue(5), addLabels):pilares.competitividade]`
```

Funciona com properties aninhadas tipo `pilares.competitividade`. Atualiza frontmatter automaticamente.

**Pré-requisito:** valor inicial no frontmatter precisa ser `0` ou número (não `null` — slider não consegue posicionar marker em null).

---

## 🔒 Anti-alucinação estrutural — case "Cabeça de Campeão"

Durante research da Fase 0, descobri que `Cabeça de Campeão` é de **François Ducasse**, NÃO de Joel Jota (Joel só fez prefácio da edição brasileira). O brief inicial atribuía erroneamente.

**Lição:** mesmo com prompts bem definidos, verificação de autoria via filesystem (`08 - Livros/<livro>.md` com frontmatter `autor:`) é o único gate confiável. Regra de prompt sozinha não basta — coach Phase 4 deve fazer Glob + Read antes de citar qualquer livro.

**Aplicado em:**
- `08 - Livros/_INDEX.md` (tabela "autoria — lembretes críticos")
- `07 - Frases/Banco-Frases.md` (frase de Cabeça de Campeão atribuída a Ducasse)
- `_meta/calibration.md` (regra hard: < 3 amostras → recusa estimativa)

---

## 🏗️ Arquitetura: dashboard decomposto

Single Home.md com 6+ Dataview blocks engasga após ~1k notas. Decomposição:
- `Home.md` — LEVE (saudação + frase + bloco HOJE + embed rotina + links). **Zero Tracker.**
- `Semana-Atual.md` — PESADO (5 pilares agregados + Tracker 8 semanas + Tasks)
- `Mes-Atual.md` — PESADO (médias mês + bar chart + heatmap)

Sub-dashboards opens-on-demand. Se usuário não clicar, não paga custo de render.

---

## 🌳 Estratégia de repos

**2 repos privados separados** no GitHub:
- `Agente-Pessoal---Obsidian` — projeto GSD (`.planning/`, `Imagens Planner/`, `CLAUDE.md`)
- `agente-pessoal-obsidian-vault` — vault pessoal (auto-backup obsidian-git 30min)

Vault em `.gitignore` do parent → evita nested git issues. Cada repo tem responsabilidade clara.

---

*Atualizado: 2026-04-25 após Phase 3 concluída.*
