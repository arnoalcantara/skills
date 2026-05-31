---
name: criar-super-mae
description: Cria uma pasta umbrella ("super mãe") na home agrupando as pastas PARA + workspace Claude Cowork em um lugar único, e atualiza o CLAUDE.md/MEMORY.md do Cowork com rotas externas para as pastas-irmãs. Use depois de aplicar PARA, quando o usuário pede para "isolar" essas pastas numa super-mãe.
---

# Criar super-mãe + integrar Cowork

## Quando usar

Depois de `aplicar-sistema-para`, quando o usuário pede:
- "isola essas 4 pastas em uma super mãe"
- "agrupa tudo dentro de uma pasta umbrella"
- "quero tudo meu em um lugar só"

E quando o workspace de IA (Claude Cowork ou similar) deve coexistir como peer dessas pastas.

## Decisão fundamental antes de executar

**Cowork dentro ou fora da super mãe?**

Antes de mover, perguntar:
1. **Dentro (recomendado):** super-mãe vira root único de tudo do usuário. Cowork tem acesso fácil às irmãs via rotas externas.
2. **Fora:** super-mãe contém só as PARA. Cowork fica como peer da super-mãe na home.

Se for dentro, NÃO numerar o Cowork. Razão: números (01-04) sinalizam categoria; Cowork é superfície de controle, natureza diferente. Misturar dilui o significado.

## Nome da super-mãe

Sugerir nomes curtos:
- `Arno/` (ou nome do usuário) — pessoal, direto
- `00_Arno/` — prefixo numérico força ordenar primeiro no Finder
- `Workspace/`, `Hub/` — neutros

Confirmar com o usuário antes.

## Execução

```zsh
cd ~ && \
mkdir -p "00_Arno" && \
MAN=~/_MANIFESTO_SUPERMAE.tsv && \
printf "pasta\torigem\tdestino\n" > "$MAN" && \
mv_sm() {
  local pasta="$1"
  [ ! -e "$HOME/$pasta" ] && return 0
  printf "%s\t%s\t%s\n" "$pasta" "$HOME/$pasta" "$HOME/00_Arno/$pasta" >> "$MAN"
  mv "$HOME/$pasta" "$HOME/00_Arno/"
} && \
mv_sm "01_Projetos" && \
mv_sm "02_Conteúdo" && \
mv_sm "03_Vida" && \
mv_sm "04_Recursos" && \
mv_sm "Claude Cowork"
```

## Atualizar CLAUDE.md do Cowork

Adicionar **seção "Rotas externas"** apontando para as 4 pastas-irmãs. Modelo:

```markdown
## Rotas externas (pastas-irmãs em `~/00_Arno/`)

O Cowork mora em `~/00_Arno/Claude Cowork/`. Ao lado existem 4 pastas-irmãs organizadas pelo sistema PARA.

| Pasta-irmã | O que tem | Carregar quando... |
|---|---|---|
| `~/00_Arno/01_Projetos/` | <lista de projetos> | tarefa sobre projeto digital ativo |
| `~/00_Arno/02_Conteúdo/` | <lista de pastas de mídia> | tarefa sobre vídeo, reels, edição |
| `~/00_Arno/03_Vida/` | <pastas pessoais> | tarefa sobre família, finanças, patrimônio |
| `~/00_Arno/04_Recursos/` | <material de estudo> | tarefa que consulta referência |

**Convenção:** ao escrever em pasta-irmã, anunciar o caminho exato antes.
```

Considerar adicionar regra **bytes pesados vs metadata leve** se houver paralelo entre `01_Projetos/<X>/` e `Cowork/Empresas/<X>/`:
> Assets de produção (vídeo, áudio) → `01_Projetos/`. Metadata operacional (planejamento, brand) → `Cowork/Empresas/`.

## Atualizar MEMORY.md do Cowork

Adicionar seção **Topologia do home** no topo, com a data e o mapa das pastas-irmãs.

## Reverter

Script tem 3 partes: devolve as pastas, restaura `CLAUDE.md`, restaura `MEMORY.md`. Usar Python para edições inline:

```python
import re, sys
s = open(sys.argv[1]).read()
s = re.sub(r"## Rotas externas.*?(?=## )", "", s, flags=re.DOTALL)
open(sys.argv[1], "w").write(s)
```

## Anti-padrão

- **Não** mexer em pastas-sistema do macOS (não cabem na super-mãe — vivem na home por design)
- **Não** mexer em `Candidatos_a_Apagar/` (vive na home, fora da super-mãe — é zona de descarte)
- **Não** numerar o Cowork como `05_` se ele entrar na super-mãe (natureza diferente)
- **Não** quebrar as rotas internas do Cowork — só **acrescentar** uma seção de rotas externas
