---
slug: anti-alucinacao-estrutural-filesystem-check
title: Anti-alucinação ESTRUTURAL via filesystem (não só prompt) — case Cabeça de Campeão
status: open
created: 2026-04-25
updated: 2026-04-25
---

# Thread: Anti-alucinação estrutural via filesystem

## Goal

Documentar a regra hard do projeto: **citações de livros, autores e referências DEVEM ser validadas via filesystem antes de uso pelo coach** — não basta regra de prompt. Estabelecer pattern Glob + Read + frontmatter check como base de Phase 4 (Coach).

## Context

*Pattern crystallizado em 2026-04-25 após case real durante research da Fase 0.*

### Case que validou a regra

**O que aconteceu:**
- Brief inicial do projeto (PROJECT.md inicial) atribuiu o livro "Cabeça de Campeão" ao Joel Jota
- Eu (Claude) tinha aceito como verdade — afinal, Joel Jota é treinador de alta performance, faz sentido
- Research da Fase 0 (gsd-project-researcher dimension Features) descobriu via fontes verificáveis (Citadel Editora) que o livro é de **François Ducasse**
- Joel Jota fez apenas o **prefácio da edição brasileira**

**Por que importa:**
- Se eu (coach) tivesse citado "Cabeça de Campeão" como livro do Joel Jota, teria propagado info errada por 6+ meses de uso
- Usuário tomaria decisão tomada em info incorreta
- Erro era invisível no nível de prompt (faz "sentido" semanticamente)

### Pattern da regra estrutural

**Phase 4 deve implementar:**

```
Antes de citar QUALQUER livro/autor:

1. Glob: encontrar arquivo correspondente em `08 - Livros/*.md`
   - Se NÃO existe → responder "não tenho nota sobre esse livro"
   - NUNCA inventar conteúdo

2. Read: abrir o arquivo encontrado
   - Validar frontmatter `autor:` corresponde ao que vou citar
   - Validar `lido: true`
   - Se `lido: false` → responder "livro ainda não lido, não posso fundamentar"

3. Output: SEMPRE incluir filesystem path no resultado
   Ex: "conforme Atomic Habits (`08 - Livros/Atomic-Habits.md`), p. 42..."
   Path serve como pista de verificação para o usuário
```

### Por que regra de prompt sozinha não basta

- LLMs alucinam citações em ~20%+ dos casos mesmo com regra explícita (research/PITFALLS.md)
- "Faça apenas se tiver evidência" é regra interpretável — viés de complacência ainda emerge
- Filesystem check é binário: arquivo existe ou não. Sem ambiguidade.

## References

- `Agente Pessoal - Obsidian/_meta/calibration.md` — regra hard "<3 amostras = sem histórico"
- `Agente Pessoal - Obsidian/08 - Livros/_INDEX.md` — tabela "autoria — lembretes críticos" (Cabeça de Campeão = Ducasse)
- `Agente Pessoal - Obsidian/07 - Frases/Banco-Frases.md` — frase de Cabeça de Campeão atribuída a Ducasse
- `.planning/research/SUMMARY.md` — case documentado em "Critical Findings"
- `.planning/PROJECT.md` Key Decisions — "Anti-alucinação ESTRUTURAL"

## Next Steps

- 🔴 **CRÍTICO para Fase 4** — implementar o pattern Glob + Read + filesystem path como CORE do coach prompt
- Adicionar testes adversariais: pedir ao coach que cite livro inexistente → deve recusar com mensagem padrão
- Adicionar ao prompt do coach a regra "se você for citar [livro], primeiro me mostre o filesystem path do `08 - Livros/<livro>.md` que está consultando"
- Generalizar pattern para outras citações: autores, frases motivacionais (que já têm `autor + fonte` em Banco-Frases.md), conceitos do método Joel Jota
