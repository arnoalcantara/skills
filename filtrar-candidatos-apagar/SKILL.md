---
name: filtrar-candidatos-apagar
description: Segunda passada agressiva de seleção de candidatos a apagar — preserva apenas arquivos de ALTA IMPORTÂNCIA, move todo o resto. Use depois que a pasta já foi organizada por tipo e usuário pede para "filtrar mais" ou "preservar só o essencial".
---

# Filtrar candidatos a apagar — passada agressiva

## Quando usar

Depois de `organizar-pasta-por-tipo`. Usuário diz coisas como:
- "agora vá nas pastas organizadas e eleja mais candidatos"
- "preserve apenas arquivos de alta importância"
- "seja mais agressivo"

## Hierarquia do que é alta importância

Em ordem decrescente:

1. **Sobrevivência/jurídico/financeiro** — passaportes, IDs, certidões, contratos, statements bancários do ano corrente, declarações de IR, chaves de carteira cripto. **Nunca move.**
2. **IP autoral do usuário** — textos próprios, métodos, frameworks, templates de voz, cartas/cursos próprios, brand books. **Nunca move sem confirmação.**
3. **Fotos/vídeos pessoais** — `IMG_*.JPG/HEIC/MOV` do iPhone, vídeos de família, fotos de filhos. **Conservador.**
4. **Produtos ativos/recentes** — campanhas em veiculação, materiais de cursos em vendas. **Conservador.**
5. **Logos, marcas, identidade visual** — masters de logos próprios.

## O que move sem dó

- **Transcrições/atas** de reuniões com mais de 30 dias
- **Material de cursos de terceiros** quando re-baixável (transcrições rbop_*, aulas em .pages)
- **Versões intermediárias** (`v1`, `v2`, `(1)`, `(2)`, `Teste 1`, `vv*`, `tmp_`)
- **PDFs duplicados de DOCX próprios** (mesmo conteúdo em dois formatos)
- **Receipts/exames antigos** (prescrições, planos alimentares dos meses anteriores)
- **Relatórios de teste/simulação** (`Simulacao_*`, `Respostas_Simuladas_*`, dashboards de teste)
- **Imagens geradas por IA** em testes (nomes longos com hash, "Hopeful Kitchen", "Silent Room", etc.)
- **Thumbnails web** (notícias, capas que não são do próprio canal)
- **WhatsApp Image antigas** (típico download de grupo)
- **Capturas de tela antigas** (`Captura de Tela*`)
- **Instaladores `.dmg` antigos**
- **HTMLs offline de aulas** (re-acessíveis no portal)

## Processo

1. **Listar conteúdo de cada subpasta organizada** (Documentos, PDFs, Imagens, Planilhas, etc.)
2. **Para cada arquivo, classificar mental** em "alta importância" ou "move"
3. **Construir lista de moves explícita** com nome exato — evitar globs muito largos
4. **Mover para `_Candidatos_a_Apagar/`** mantendo manifesto TSV
5. **Reportar contagem antes/depois por subpasta**
6. **Justificar 2 candidatos movidos + 2 mantidos** ao final (transparência)

## Critério para o que justificar

Escolher exemplos extremos:
- O candidato de maior tamanho movido (explicar por que está OK descartar)
- Um candidato menor mas conceitualmente subtle (mostrar nuance)
- Um preservado que aparentemente seria descartável (defender)
- Um preservado obviamente crítico (afirmar)

## Reversibilidade obrigatória

Manifesto TSV: `arquivo<TAB>pasta_origem`. Script `_reverter.sh` que lê o manifesto e devolve cada arquivo:

```zsh
revert_one() {
  local file="$1"
  local folder="$2"
  if [ -e "$CAND/$file" ]; then
    mkdir -p "$ORG/$folder"
    mv -n "$CAND/$file" "$ORG/$folder/"
  fi
}
```

## Anti-padrão

- **NÃO** mover IMG_*.JPG/HEIC/MOV (são iPhone, podem ser memórias)
- **NÃO** mover sem ter manifesto pronto
- **NÃO** confiar em padrão de nome — verificar mtime
