---
name: organizar-pasta-por-tipo
description: Organiza uma pasta caótica (típico de Downloads) agrupando arquivos por tipo em subpastas + cria pasta de Candidatos a Apagar com critério de idade. Use quando o usuário pede organização de Downloads, Desktop, ou qualquer pasta-monte com centenas de arquivos misturados.
---

# Organizar pasta por tipo

## Quando usar

Pasta com muitos arquivos heterogêneos (typically `~/Downloads/`, `~/Desktop/`). Sinais: usuário diz "organize", "está uma bagunça", "pasta cheia de arquivos antigos".

## O que faz

1. Cria estrutura `_Organizado/{Documentos, PDFs, Imagens, Planilhas, Vídeos, Áudio, Compactados, Instaladores, Web, Outros, Pastas}` dentro da pasta-alvo
2. Cria pasta `_Candidatos_a_Apagar/` no mesmo nível
3. Move para Candidatos os itens com idade ≥ N dias que casam padrões de lixo
4. Move o restante para a subpasta certa por extensão

## Critério para Candidatos a Apagar (primeira passada)

Idade mínima: **N dias** (padrão 15). Padrões:
- Duplicatas com sufixo `(1)`, `(2)`, `(3)`, `(4)`, `(5)`
- Instaladores `.dmg`, `.pkg` (já instalados)
- `.zip` (provavelmente já extraído)
- Imagens `.webp` (lixo web)
- Arquivos sem nome (ex: ` .pdf`)
- Pastas de dump de site (`*_files/`, `drive-download-*/`)

**Não tocar em:** arquivos pessoais (IMG_*.JPG/MOV do iPhone), documentos jurídicos/financeiros, IP autoral.

## Passos de execução

1. Listar conteúdo, contar extensões com `find . -maxdepth 1 -type f -name "*.<ext>"`
2. Criar a estrutura `_Organizado/` e `_Candidatos_a_Apagar/`
3. Mover candidatos PRIMEIRO (antes da organização por tipo, para usar nomes originais)
4. Para cada extensão restante, mover para a subpasta apropriada com `case` em zsh:

```zsh
case "$ext_lower" in
  docx|doc|pages|rtf|txt|md|odt|pptx|ppt|key) dest="Documentos" ;;
  pdf|epub) dest="PDFs" ;;
  xlsx|xls|csv|numbers) dest="Planilhas" ;;
  png|jpg|jpeg|webp|heic|svg|gif|bmp|tiff) dest="Imagens" ;;
  mov|mp4|avi|mkv|m4v) dest="Vídeos" ;;
  mp3|m4a|wav|flac|ogg) dest="Áudio" ;;
  zip|rar|7z|tar|gz) dest="Compactados" ;;
  dmg|pkg|app) dest="Instaladores" ;;
  html|htm) dest="Web" ;;
  *) dest="Outros" ;;
esac
```

5. Pastas avulsas vão para `_Organizado/Pastas/`
6. Reportar contagem final por subpasta + total em Candidatos

## Reversibilidade

Não é estritamente necessária aqui — usuário pode reorganizar manualmente. Mas se quiser, registre cada mv num TSV `_MANIFESTO.tsv` com `arquivo<TAB>origem<TAB>destino`.

## Anti-padrão

- **NÃO** apagar nada nesta skill — só mover. Apagar é skill separada (`apagar-candidatos`).
- **NÃO** ser agressivo demais na primeira passada. A segunda passada (skill `filtrar-candidatos-apagar`) é onde se aplica julgamento de "alta importância".
