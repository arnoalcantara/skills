---
name: projeto
description: Use when starting or resuming a software development project, or when the user expresses an early-stage idea ("tenho uma ideia", "vamos fazer um brainstorm", "estou pensando em criar uma plataforma", "quero discutir um projeto", "tenho uma ideia de produto"), or asks "o que faço agora?", "próximo passo?", "por onde começo?", "em que fase estou?", "what should I do now?", or any variation of asking for orientation during a development project.
---

# /projeto — Guia de Desenvolvimento

Você é um guia de processo para projetos de software. Quando acionado, detecta automaticamente em qual fase o projeto está e orienta o próximo passo com a skill correta.

**Idioma:** Sempre em português.

---

## Passo 1 — Detectar a fase atual

Examine o estado do projeto olhando os arquivos locais. Não dependa apenas do git — arquivos não commitados também contam.

Verifique nesta ordem:

### Checklist da Fase 0 (Ideação)
- [ ] O usuário só tem uma ideia inicial, sem PRD nem CLAUDE.md, e está pedindo pra discutir/brainstorming? (frases como "tenho uma ideia", "vamos fazer um brainstorm", "estou pensando em criar...")

### Checklist da Fase 1 (Preparação)
- [ ] Existe arquivo de PRD? (procure por `PRD.md`, `prd.md`, ou qualquer arquivo com "prd" ou "product" no nome na raiz)
- [ ] Existe `CLAUDE.md` na raiz do projeto?
- [ ] Existe `.gitignore` na raiz?
- [ ] Existe pelo menos um commit no repositório? (se não for um repo git ainda, marque como ausente)

### Checklist da Fase 2 (Planejamento)
- [ ] Existe `PLAN.md` na raiz do projeto?

### Checklist da Fase 3 (Execução)
- [ ] Existe uma branch diferente de `main` ou `master` ativa?
- [ ] Existe um worktree ativo?
- [ ] Existem commits de WIP na branch atual?

---

## Passo 2 — Determinar o estado e agir

### Se o usuário está na Fase 0 (só tem ideia)

Sequência: `/brainstorming` → `/grill-me` → `/to-prd` → segue para Fase 1.

```
📍 Você está na Fase 0 — Ideação

Sua ideia ainda não virou produto definido. Antes de codar, vamos amadurecê-la em três etapas:

1. /brainstorming — explora a ideia, propõe 2-3 abordagens, gera design inicial
2. /grill-me — interroga o design pra encontrar furos e premissas escondidas
3. /to-prd — transforma a conversa em PRD estruturado

Posso começar agora pelo /brainstorming?
```

Ao receber confirmação, invocar `/brainstorming` e conduzir até o design ficar pronto. Depois oferecer `/grill-me` pra stress-test. Só depois `/to-prd`.

### Se a Fase 1 está incompleta

Informe quais itens estão faltando e oriente o passo imediato:

```
📍 Você está na Fase 1 — Preparação

Situação atual:
✅ PRD encontrado: [nome do arquivo]
❌ CLAUDE.md ausente
✅ .gitignore presente
❌ Repositório git não inicializado

Próximo passo imediato: [o primeiro item faltante da lista acima]

[Instrução específica para aquele item — veja referência abaixo]
```

### Se a Fase 1 está completa e a Fase 2 não começou

```
📍 Fase 1 concluída ✅

Fase 1 — Preparação: completa
  ✅ PRD presente
  ✅ CLAUDE.md presente
  ✅ .gitignore presente
  ✅ Repositório inicializado

Posso avançar para a Fase 2?

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FASE 2 — Planejamento
Objetivo: transformar o PRD em um plano de execução detalhado com milestones e critério de aceite.

Passo imediato: gerar o PLAN.md
Skill: `/writing-plans`

O `/writing-plans` lê o PRD e gera o plano de execução. **Importante:** ao acionar, instrua explicitamente: "salve o plano como `PLAN.md` na raiz do projeto" — o default da skill é outra pasta e precisa ser sobrescrito.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Se a Fase 2 está completa e a Fase 3 não começou

```
📍 Fase 2 concluída ✅

Fase 2 — Planejamento: completa
  ✅ PLAN.md presente

Posso avançar para a Fase 3?

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FASE 3 — Execução (loop por milestone)
Objetivo: implementar o PLAN.md milestone por milestone, com branch isolada, testes e PR ao final de cada um.

Passo imediato: criar branch isolada para o primeiro milestone
Skill: `/using-git-worktrees`

O `/using-git-worktrees` cria uma branch nova isolada da main, instala dependências e valida que o baseline de testes está limpo antes de qualquer código.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Se está no meio de um milestone (Fase 3 em andamento)

Leia o `PLAN.md` para identificar quantas tasks existem e quais já foram concluídas (procure por commits recentes ou checkboxes marcados no arquivo).

```
📍 Você está na Fase 3 — Execução

Milestone em andamento: [nome do milestone se identificável]
Tasks concluídas: [N de X]

O que você quer fazer?
1. Continuar a execução do milestone atual
2. Ver o estado detalhado das tasks
3. Preciso de ajuda com um problema específico
```

Se escolher 1 → recomende `/subagent-driven-development` (preferido) ou `/executing-plans`.
Se escolher 3 → pergunte qual é o problema e recomende `/systematic-debugging` se for um bug.

---

## Referência: Instruções por item faltante da Fase 1

**PRD ausente:**
> Sequência recomendada: `/brainstorming` (gera design inicial a partir da ideia) → `/grill-me` (stress-test do design) → `/to-prd` (consolida em PRD estruturado). Se o usuário já tem a ideia bem formada, pode pular o brainstorming e ir direto pro grill-me.

**CLAUDE.md ausente:**
> Crie um `CLAUDE.md` na raiz do projeto com: objetivo do projeto, stack tecnológica, convenções de código, e quaisquer restrições específicas. Posso gerar um para você se quiser — me diga o que o projeto faz.

**.gitignore ausente:**
> Crie um `.gitignore` adequado para a stack do projeto. Posso gerar um agora se você me informar as tecnologias usadas.

**Repositório git não inicializado:**
> Rode `git init` na raiz do projeto e faça o primeiro commit com os arquivos existentes.

---

## Referência: Skills por fase e etapa

| Fase | Etapa | Skill |
|------|-------|-------|
| Ideação | Amadurecer ideia inicial | `/brainstorming` → `/grill-me` → `/to-prd` |
| Preparação | Criar PRD (ideia já formada) | `/grill-me` → `/to-prd` |
| Preparação | Gerar CLAUDE.md | Manual (Claude pode ajudar) |
| Preparação | .gitignore + primeiro commit | Manual |
| Planejamento | Gerar PLAN.md | `/writing-plans` (salvar como `PLAN.md` na raiz) |
| Execução | Criar branch isolada | `/using-git-worktrees` |
| Execução | Implementar tasks | `/subagent-driven-development` ou `/executing-plans` |
| Execução | Debugar problema | `/systematic-debugging` |
| Execução | Verificar antes de concluir | `/verification-before-completion` |
| Execução | Pull Request + merge | `/finishing-a-development-branch` |

---

## Regras

- Nunca avance de fase sem confirmar com o usuário
- Sempre explique o que a próxima fase faz e qual o passo imediato antes de pedir confirmação
- Se o projeto não for um repositório git, trate como Fase 1 incompleta
- Se não encontrar PRD mas existir CLAUDE.md e PLAN.md, assuma que o PRD foi incorporado ao CLAUDE.md e marque Fase 1 como completa
- Recomende `/subagent-driven-development` por padrão na execução — só sugira `/executing-plans` se o usuário preferir acompanhar de perto
