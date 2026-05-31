---
name: varrer-home-candidatos
description: Análise sistêmica de toda a home do usuário — apaga direto pastas vazias, cria pasta de Candidatos com subpastas-categoria-justificativa, e varre cada pasta movendo candidatos com critério explícito. Use quando o usuário pede limpeza geral da home, com regras claras de preservação e reversibilidade.
---

# Varrer home — candidatos por categoria

## Quando usar

Usuário pede análise sistêmica da home toda, não só de uma pasta. Sinais:
- "analise todas as minhas pastas"
- "limpa minha home"
- "espaço acabando, varre tudo"

## Estrutura obrigatória

Criar `~/Candidatos_a_Apagar/` com **poucas** subpastas-categoria, cada uma sendo a **justificativa** do porquê apagar:

```
Candidatos_a_Apagar/
├── Cache_e_Build/           # node_modules, .next, dist, build, venv, .cache
├── Duplicatas/              # arquivos repetidos, formatos redundantes
├── Rascunhos_e_Testes/      # v1, v2, teste_, tmp_, drafts antigos
└── Midia_Antiga/            # thumbs, screenshots, frames, brutas de projeto encerrado
```

**Regra:** poucas categorias (≤5), nomes curtos, sem ambiguidade. Cada arquivo entra na categoria que melhor justifica seu descarte.

## Fase 1 — Apagar pastas vazias diretamente (sem confirmação)

Pastas com 0 arquivos úteis (só `.DS_Store`, `.localized`, ou subpasta vazia). Mandar para Lixeira:
```zsh
mv "Pasta Vazia" ~/.Trash/
```

## Fase 2 — Restrições obrigatórias

Antes de varrer, confirmar e respeitar a lista de exclusão do usuário. Padrões usuais:
- **Pastas-sistema do macOS:** `Library/`, `Applications/`, `Desktop/`, `Documents/`, `Downloads/`, `Movies/`, `Music/`, `Pictures/`, `Public/`
- **Hidden dotfiles:** `.claude/`, `.config/`, `.npm/`, etc.
- **Adobe sync:** `Creative Cloud Files .../`
- **Material de estudo declarado** (ex: "não toque em Olavo")
- **Música** (Music, Guitar, Curso de Piano)
- **Documentos importantes** (jurídicos, financeiros, identidade)

Pedir confirmação se houver dúvida.

## Fase 3 — Manifesto e helper de move

```zsh
MAN=~/Candidatos_a_Apagar/_MANIFESTO_REVERSAO.tsv
printf "arquivo_ou_pasta\torigem_completa\tcategoria\n" > "$MAN"

mv_to() {
  local src="$1"; local cat="$2"
  [ ! -e "$src" ] && return 0
  case "$src" in /*) ;; *) src="$HOME/$src" ;; esac
  [ ! -e "$src" ] && return 0
  local base=$(basename "$src")
  local dest="$HOME/Candidatos_a_Apagar/$cat/$base"
  # collision handling
  if [ -e "$dest" ]; then
    local i=2
    while [ -e "${dest}_${i}" ]; do i=$((i+1)); done
    dest="${dest}_${i}"; base="${base}_${i}"
  fi
  printf "%s\t%s\t%s\n" "$base" "$src" "$cat" >> "$MAN"
  mv "$src" "$dest"
}
```

## Fase 4 — Varredura por padrões universais

**Cache/Build (alvo fácil — alto ganho de espaço):**
```zsh
find . -maxdepth 6 -type d \( -name "node_modules" -o -name ".next" -o -name ".turbo" -o -name "dist" -o -name "build" -o -name "venv" -o -name ".cache" \) \
  | grep -v -E "(Library|\.Trash|Candidatos_a_Apagar|node_modules/.+/node_modules)"
```

**Outros padrões por pasta:** screenshots em pastas de produção, "Brutas" subfolders de projetos antigos (>6 meses), instaladores .dmg, etc.

## Fase 5 — Script de reversão

Sempre criar `~/Candidatos_a_Apagar/_reverter.sh`:

```zsh
#!/bin/zsh
MAN="$HOME/Candidatos_a_Apagar/_MANIFESTO_REVERSAO.tsv"
tail -n +2 "$MAN" | while IFS=$'\t' read -r item origem categoria; do
  case "$origem" in /*) ;; *) origem="$HOME/$origem" ;; esac
  src="$HOME/Candidatos_a_Apagar/$categoria/$item"
  if [ -e "$src" ]; then
    mkdir -p "$(dirname "$origem")"
    mv -n "$src" "$origem"
    echo "✓ $item → $origem"
  fi
done
```

## Fase 6 — Justificativa final obrigatória

Ao final, justificar:
- **2 escolhas que foram movidas** (mostrar critério e custo de manter)
- **2 escolhas que foram mantidas** (defender por que sobrevivem ao corte)

Isso força transparência e ensina o usuário a auditar.

## Anti-padrão

- **Não** apagar definitivamente nesta skill (use `apagar-candidatos`)
- **Não** criar 10 categorias — ≤5
- **Não** mexer em pasta com material declarado intocável
- **Não** seguir cegamente padrão de extensão — usar `mtime`/`atime` para decidir idade
