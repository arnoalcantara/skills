---
name: apagar-candidatos
description: Move a pasta de Candidatos a Apagar (`_Candidatos_a_Apagar/` ou `Candidatos_a_Apagar/`) para a Lixeira do macOS de forma recuperável, com timestamp no nome. Use quando o usuário disser "apaga os candidatos", "manda pra lixeira", ou aprovar o descarte após revisão.
---

# Apagar candidatos

## Quando usar

Após o usuário revisar a pasta de Candidatos e dar o ok para apagar. Triggers:
- "apaga os candidatos"
- "manda pra lixeira"
- "pode apagar"
- "ok, descarta"

## O que NÃO fazer

- **Nunca** usar `rm -rf` — operação irreversível.
- **Nunca** apagar sem que o usuário tenha aprovado explicitamente.
- **Nunca** apagar pastas grandes (>5 GB) sem mostrar o tamanho antes.

## Procedimento

1. Contar itens e tamanho da pasta de candidatos:
   ```zsh
   COUNT=$(ls _Candidatos_a_Apagar/ | wc -l | tr -d ' ')
   SIZE=$(du -sh _Candidatos_a_Apagar/ | cut -f1)
   echo "Movendo $COUNT itens ($SIZE) para a Lixeira do macOS..."
   ```
2. Mover a pasta inteira para `~/.Trash/` com timestamp:
   ```zsh
   mv _Candidatos_a_Apagar ~/.Trash/Candidatos_a_Apagar_$(date +%Y%m%d_%H%M%S)
   ```
3. Confirmar com `ls` na origem que a pasta sumiu.
4. Reportar: "Pasta foi para a Lixeira (recuperável até esvaziar)."

## Vantagens do macOS Trash

- Recuperável até o usuário esvaziar a Lixeira (Finder → Esvaziar Lixeira)
- Não consome espaço durante esse período (mesma partição)
- Sobrevive a reinicializações
- Funciona com pastas grandes (não tem limite prático)

## Anti-padrão

- `rm -rf` direto
- Apagar sem confirmação explícita
- Não preservar o manifesto de reversão (se existir, o usuário pode querer entender o que foi descartado)
