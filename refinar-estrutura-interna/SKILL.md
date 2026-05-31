---
name: refinar-estrutura-interna
description: Analisa a estrutura interna de uma super-mãe PARA já criada, detecta inconsistências (sobreposições, nomes ruins, pastas no lugar errado) e propõe duas opções de refatoração — uma enxuta e uma ambiciosa — com recomendação. Use quando o usuário pede para "analisar a pasta X e propor reorganização" depois que o esqueleto já está montado.
---

# Refinar estrutura interna

## Quando usar

Depois que a super-mãe PARA está montada (`aplicar-sistema-para` + `criar-super-mae` já rodaram). Usuário pede:
- "analise a pasta 00_Arno e proponha reorganização"
- "tem coisa fora do lugar?"
- "limpa o que ainda está bagunçado"

## O que faz

1. **Mapeia** a estrutura interna de cada pasta-mãe (01-04 + workspace de IA, se houver)
2. **Detecta** 6 tipos de problema:
   - Sobreposições (mesma coisa em 2 lugares)
   - Pastas no lugar errado (Casa Airbnb dentro do workspace de IA, por ex.)
   - Pastas-projeto que viraram saco-de-gato sem subdivisão
   - Convenções de nome inconsistentes (singular/plural, posse/descrição, capitalização)
   - Arquivos soltos na raiz do workspace (precisam de Inbox/ ou redistribuição)
   - Pastas com propósito ambíguo (não é Projeto, não é Recurso — o quê?)
3. **Distingue bug de feature**: nem toda aparente duplicação é problema. Ex: `01_Projetos/Ventura/` (assets pesados) e `Cowork/Empresas/Ventura/` (metadata leve) é divisão correta, não duplicação.
4. **Apresenta duas opções**:
   - **Opção A — Enxuta:** só corrige inconsistências, mantém profundidade atual
   - **Opção B — Ambiciosa:** introduz nova axis (ex: subnível por empresa)
5. **Recomenda uma** com justificativa.
6. **NÃO executa** sem o usuário escolher.

## Critérios de detecção

### Sobreposição vs divisão legítima

Mesma "coisa" em 2 lugares pode ser:
- **Bug** se conteúdo se sobrepõe (mesmos arquivos, mesma intenção)
- **Feature** se papéis são distintos (assets vs metadata, dados vs configuração)

Regra: se você consegue articular o critério "X fica em A, Y em B" e ele faz sentido, é feature. Se a resposta é "não sei", é bug.

### Pasta no lugar errado

Sinais:
- Pasta de patrimônio dentro do workspace de produção (Casa Treze Tílias em Cowork)
- Pasta de experimento dentro de Projetos ativos (Agentes GPT solto)
- Pasta de música em Recursos genéricos (deveria ter sub-agrupamento)

### Saco-de-gato sem subdivisão

`04_Recursos/` com 7+ pastas heterogêneas pede sub-agrupamento por afinidade (`Filosofia/`, `Música/`, `Programação/`).
`Playground/` com 10+ experimentos misturados com skills em produção pede promoção dos itens maduros.

### Convenção de nomes

Padronizar:
- Plural consistente (Reels não Reel)
- Sem nome de pessoa quando é categoria (Xadrez, não "Xadrez GM Evandro")
- Sem prefixo redundante ("Programa Araceli Casais" não "Documentação Programa Araceli Casais")
- Posse padronizada (`Reels - Arno` vs `Meus Reels` — escolher um padrão)

## Formato da proposta ao usuário

Para cada problema:
1. Citar o problema com caminho exato
2. Indicar gravidade (bug claro vs ajuste opcional)
3. Sugerir correção

Para a proposta final:
- **Opção A (enxuta):** lista de N renomeações/movimentações curtas
- **Opção B (ambiciosa):** lista de movimentações com nova estrutura
- **Custo de cada uma** (bytes movidos, profundidade adicionada, referências a quebrar)
- **Recomendação explícita** com motivo

Aguarda escolha do usuário antes de executar.

## Execução (depois da escolha)

Mesmo padrão dos outros skills: manifesto TSV + script de reversão.

```zsh
MAN=~/_MANIFESTO_REFINAMENTO.tsv
printf "operacao\torigem\tdestino\n" > "$MAN"

op() { printf "%s\t%s\t%s\n" "$1" "$2" "$3" >> "$MAN"; }
op "rename" "$origem" "$destino"; mv "$origem" "$destino"
op "move" "$origem" "$destino"; mv "$origem" "$destino"
```

Reversor deve suportar tanto rename quanto move (mesma operação no fundo: `mv "$destino" "$origem"`).

## Atualização de documentação acoplada

Se a reorganização afeta um workspace com `CLAUDE.md`/`MEMORY.md` (Cowork, etc.), **atualizar o `CLAUDE.md`** com nomes novos nas rotas externas. Reverter deve restaurar o `CLAUDE.md` também.

## Anti-padrão

- **Não** propor reorganização sem ter mapeado a estrutura completa primeiro
- **Não** confundir divisão legítima com sobreposição (perguntar-se "qual o critério?")
- **Não** introduzir mais profundidade do que o problema justifica
- **Não** executar antes do usuário escolher entre as opções
