---
name: aplicar-sistema-para
description: Aplica o sistema PARA simplificado (Jeff Su / Tiago Forte) à home — agrupa N pastas-projeto em 4 pastas mãe numeradas (01_Projetos, 02_Conteúdo, 03_Vida, 04_Recursos), com reversibilidade. Use quando o usuário pede para "reorganizar a home", "agrupar pastas", "sistema inteligente de organização".
---

# Aplicar sistema PARA à home

## Quando usar

Usuário pede organização estrutural da home com base em algum sistema (geralmente PARA ou Jeff Su). Triggers:
- "reorganize minhas pastas"
- "agrupa as pastas em pastas mães"
- "sistema baseado em Jeff Su"
- "PARA"

## Sistema (PARA simplificado, 4 categorias)

| Pasta | Critério |
|---|---|
| `01_Projetos/` | Coisas com output deliverable em construção ativa (código, produto, projeto com prazo) |
| `02_Conteúdo/` | Produção de mídia (reels, vídeo, podcast, palestras) |
| `03_Vida/` | Pessoal: família, patrimônio, finanças, identidade |
| `04_Recursos/` | Material de estudo e referência (livros, cursos terceiros, aprendizados em andamento) |

**Por que numerar?** Força ordem por frequência de uso (Projetos > Conteúdo > Vida > Recursos), em vez de alfabética.

**Por que 4?** Jeff Su é explícito — mais que isso introduz atrito na decisão de "onde salvar". Menos que isso vira saco-de-gato.

**Diferença Projetos vs Recursos:** Projeto tem deadline e output. Recurso é leitura/consulta. Se ficou em dúvida, é Recurso.

## Antes de mover, mapear

1. Listar pastas visíveis da home (excluindo dotfiles e sistema)
2. Identificar **pastas intocáveis** (lista de exclusão):
   - Sistema macOS: `Applications/`, `Desktop/`, `Documents/`, `Downloads/`, `Library/`, `Movies/`, `Music/`, `Pictures/`, `Public/`
   - Adobe: `Creative Cloud Files .../` (sync quebra se mover)
   - Material declarado intocável pelo usuário
3. Classificar cada pasta restante em uma das 4 categorias

## Critério de classificação

| Sinal | Vai para |
|---|---|
| Tem código sendo modificado, package.json, etc. | `01_Projetos/` |
| Tem footage, exports de vídeo, edições em andamento | `02_Conteúdo/` |
| Tem documentos da família, contratos, chave cripto | `03_Vida/` |
| Tem livros, aulas, material que se consulta | `04_Recursos/` |

**Aulas/cursos:** se você é aluno e ainda estuda → `04_Recursos/`. Se você está produzindo o curso → `01_Projetos/`.

## Execução

```zsh
cd ~ && \
mkdir -p "01_Projetos" "02_Conteúdo" "03_Vida" "04_Recursos" && \
MAN=~/_MANIFESTO_REORGANIZACAO.tsv && \
printf "pasta\torigem\tdestino\n" > "$MAN" && \
move_to() {
  local src="$1"; local destdir="$2"
  [ ! -e "$src" ] && return 0
  local base=$(basename "$src")
  printf "%s\t%s\t%s\n" "$base" "$HOME/$src" "$HOME/$destdir/$base" >> "$MAN"
  mv "$src" "$destdir/"
} && \
move_to "Projeto X" "01_Projetos" && \
# ... e assim por diante
```

## Reversibilidade

Script `_reverter_reorganizacao.sh`:

```zsh
#!/bin/zsh
MAN="$HOME/_MANIFESTO_REORGANIZACAO.tsv"
tail -n +2 "$MAN" | while IFS=$'\t' read -r pasta origem destino; do
  if [ -e "$destino" ]; then
    mv "$destino" "$origem"
    echo "✓ $pasta → $origem"
  fi
done
for mae in "01_Projetos" "02_Conteúdo" "03_Vida" "04_Recursos"; do
  if [ -d "$HOME/$mae" ] && [ -z "$(ls -A "$HOME/$mae" 2>/dev/null)" ]; then
    rmdir "$HOME/$mae"
  fi
done
```

## Justificativa ao usuário

Sempre justificar:
- Por que escolheu PARA (separação por acionabilidade, não tema)
- Por que 4 e não mais (atrito na decisão de salvar)
- Por que numerar (ordem por frequência)
- Casos de fronteira (ex: "por que X em Vida e não em Recursos?")

## Refinamento opcional (sub-agrupamento)

Em pastas com muitos itens heterogêneos, criar sub-agrupamento por afinidade:
- `04_Recursos/Filosofia/`, `04_Recursos/Música/`, `04_Recursos/Programação/`
- `01_Projetos/<Empresa>/` (subnível por empresa quando faz sentido)

Não fazer isso na primeira passada — só quando o problema de "muita coisa numa pasta" se manifestar.

## Anti-padrão

- **Não** criar mais que 4 mães na primeira passada
- **Não** mexer em pastas-sistema do macOS
- **Não** mexer em pasta declarada intocável
- **Não** assumir que "Empresa" é uma categoria — não é (Projetos por empresa pode virar subnível em `01_`)
