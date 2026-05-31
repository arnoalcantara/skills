---
name: reverter-organizacao
description: Reverte qualquer etapa da organização da home executada pelas skills do pipeline (organizar-home e suas 7 sub-skills). Detecta quais manifestos/scripts existem, lista opções para o usuário, e executa a reversão (tudo, uma etapa, ou um item específico) na ordem correta. Use quando o usuário disser "reverte", "desfaz", "volta como estava", "/reverter-organizacao".
---

# Reverter Organização — controle de reversão

## Quando usar

Triggers:
- "/reverter-organizacao"
- "reverte tudo"
- "desfaz a organização"
- "volta como estava"
- "reverte só a super-mãe"
- "põe a pasta X de volta no lugar dela"

## Manifestos e scripts conhecidos

Esta skill conhece os 4 pontos de reversão padrão criados pelo pipeline `organizar-home`:

| # | Etapa | Manifesto | Script |
|---|---|---|---|
| 4 | Refinamento interno (Opção A) | `~/_MANIFESTO_OPCAO_A.tsv` | `~/_reverter_opcao_a.sh` |
| 3 | Super-mãe + Cowork edits | `~/_MANIFESTO_SUPERMAE.tsv` | `~/_reverter_supermae.sh` |
| 2 | Reorganização PARA | `~/_MANIFESTO_REORGANIZACAO.tsv` | `~/_reverter_reorganizacao.sh` |
| 1 | Varredura de candidatos | `~/Candidatos_a_Apagar/_MANIFESTO_REVERSAO.tsv` | `~/Candidatos_a_Apagar/_reverter.sh` |

**Numeração propositalmente invertida** — ordem de reversão é do mais recente para o mais antigo (4 → 3 → 2 → 1). Isso replica a ordem cronológica reversa em que as transformações foram feitas.

## Fluxo de operação

### Passo 1 — Detectar o que existe

Antes de qualquer coisa, varrer:
```zsh
for f in ~/_reverter_opcao_a.sh ~/_reverter_supermae.sh ~/_reverter_reorganizacao.sh ~/Candidatos_a_Apagar/_reverter.sh; do
  [ -f "$f" ] && echo "✓ $f" || echo "✗ $f (não existe)"
done
```

Se um script não existe, a etapa correspondente nunca foi feita (ou já foi revertida). Mostrar só os disponíveis.

### Passo 2 — Apresentar opções

Modelo de mensagem:
> "Encontrei N pontos de reversão disponíveis. O que quer fazer?
>
> **A. Tudo** — reverte todas as etapas na ordem correta (4→3→2→1)
> **B. Uma etapa específica** — escolhe qual reverter
> **C. Um item específico** — devolve um arquivo/pasta ao lugar original (sem reverter a etapa toda)
>
> Lista de etapas disponíveis:
> - Etapa 4 — Refinamento interno (N movimentações)
> - Etapa 3 — Super-mãe (N pastas + edições no CLAUDE.md)
> - Etapa 2 — PARA (N pastas)
> - Etapa 1 — Candidatos varridos (N itens)
>
> Qual?"

Mostrar a contagem de linhas de cada manifesto:
```zsh
[ -f "$MAN" ] && echo "$(tail -n +2 "$MAN" | wc -l | tr -d ' ') itens"
```

### Passo 3 — Executar conforme a escolha

**Opção A (tudo):**
```zsh
# Sempre executar do mais recente para o mais antigo
[ -f ~/_reverter_opcao_a.sh ] && zsh ~/_reverter_opcao_a.sh
[ -f ~/_reverter_supermae.sh ] && zsh ~/_reverter_supermae.sh
[ -f ~/_reverter_reorganizacao.sh ] && zsh ~/_reverter_reorganizacao.sh
[ -f ~/Candidatos_a_Apagar/_reverter.sh ] && zsh ~/Candidatos_a_Apagar/_reverter.sh
```

Reportar o output de cada script à medida que rodam.

**Opção B (uma etapa):**
Pedir qual número (1-4) e rodar o script correspondente.

**Opção C (um item):**
Pedir o nome do arquivo/pasta. Todos os 4 scripts aceitam um argumento opcional:
```zsh
zsh ~/_reverter_opcao_a.sh "nome_do_arquivo"
zsh ~/_reverter_supermae.sh "Nome Pasta"
zsh ~/_reverter_reorganizacao.sh "Nome Pasta"
zsh ~/Candidatos_a_Apagar/_reverter.sh "nome_arquivo"
```

Como a skill não sabe em qual manifesto o item está, tentar todos os scripts em sequência. O primeiro que encontrar reverte; os outros reportam "não encontrado" (não é erro).

### Passo 4 — Limpeza após reversão completa

Quando a Opção A é executada com sucesso (todos os 4 scripts rodaram limpo), oferecer:
> "Reversão completa. Quer também apagar os manifestos e scripts de reversão? (Já não servem para mais nada.)"

Se sim:
```zsh
rm ~/_reverter_opcao_a.sh ~/_MANIFESTO_OPCAO_A.tsv
rm ~/_reverter_supermae.sh ~/_MANIFESTO_SUPERMAE.tsv
rm ~/_reverter_reorganizacao.sh ~/_MANIFESTO_REORGANIZACAO.tsv
# Candidatos_a_Apagar inteira (já vazia)
[ -d ~/Candidatos_a_Apagar ] && [ -z "$(ls -A ~/Candidatos_a_Apagar 2>/dev/null | grep -v _MANIFESTO | grep -v _reverter)" ] && rm -rf ~/Candidatos_a_Apagar
```

## Casos especiais

### Reversão parcial não bate com etapa posterior

Se o usuário reverteu só uma etapa específica (não tudo) e isso quebra coerência com etapas posteriores, **avisar antes de executar**:

> "Atenção: você está revertendo a Etapa 2 (PARA), mas a Etapa 3 (super-mãe) ainda está aplicada. As pastas voltariam para a home mas a super-mãe `00_Arno/` ficaria vazia ou com referências quebradas. Recomendo reverter Etapa 3 primeiro. Quer ajustar?"

### Apagar candidatos já foi executado

Se `~/Candidatos_a_Apagar/` não existe mais (foi para a Lixeira via `apagar-candidatos`), **o reverter de candidatos não tem o que fazer**. Avisar:

> "A pasta Candidatos_a_Apagar já foi para a Lixeira em sessão anterior. Para recuperar arquivos individuais, abra o Finder, Lixeira, e arraste de volta manualmente. A skill não consegue reverter itens já na Lixeira."

### Cowork CLAUDE.md/MEMORY.md editados

Os scripts de Etapa 3 e 4 já restauram `CLAUDE.md` e `MEMORY.md` do Cowork via Python inline. Não precisa fazer manualmente. Só confirmar no output que aparecem as linhas "✓ CLAUDE.md restaurado" e "✓ MEMORY.md restaurado".

## Output esperado

Ao final, mostrar:
- Quantas etapas foram revertidas
- Quantos itens voltaram ao lugar
- Estado atual da home (`ls ~/`)
- Se sobrou alguma anomalia (pasta esperada que não voltou, manifesto vazio, etc.)

## Anti-padrão

- **Não** reverter sem saber o que existe (sempre detectar primeiro)
- **Não** reverter na ordem errada (sempre 4→3→2→1 na opção "tudo")
- **Não** apagar manifestos antes de confirmar com o usuário
- **Não** tentar reverter coisas que já foram para a Lixeira (instruir uso manual do Finder)
- **Não** assumir que o usuário quer reverter tudo — sempre apresentar as 3 opções (A/B/C)
