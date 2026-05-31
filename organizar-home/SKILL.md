---
name: organizar-home
description: Super-skill que orquestra as 7 skills de organização (organizar-pasta-por-tipo, filtrar-candidatos-apagar, apagar-candidatos, varrer-home-candidatos, aplicar-sistema-para, criar-super-mae, refinar-estrutura-interna) em 4 etapas. Pode rodar o pipeline completo OU pular direto para uma etapa específica. Anuncia cada etapa antes de executar e aguarda ok do usuário. Use quando o usuário pede pipeline completo OU uma etapa específica de organização — "/organizar-home", "limpa e organiza tudo", "pula pra etapa N", "só PARA", "só limpeza".
---

# Organizar Home — Pipeline Completo

## Quando usar

Usuário pede reorganização da home. Pode ser:

**Pipeline completo (4 etapas em sequência):**
- "/organizar-home"
- "organiza minha home toda"
- "limpa e estrutura tudo"
- "pipeline completo"
- "faz aquele processo todo de organização"

**Etapa específica (pular direto):**
- "/organizar-home etapa 3" ou "/organizar-home 3"
- "pula direto para as 4 pastas mães"
- "só quero o PARA"
- "só a limpeza de Downloads"
- "só a varredura de caches"
- "só o refinamento"

Para chamadas muito atômicas (uma única skill, sem cerimônia de anúncio), o usuário deve invocar a skill direto (ex: `organizar-pasta-por-tipo`). A super-skill `organizar-home` é para quando há valor no **anúncio + confirmação + reportagem ao final** — mesmo de uma etapa só.

## Princípio operacional

A super-skill **não executa as skills internas em sequência cega**. Ela:
1. **Anuncia cada etapa antes de começar** — em uma mensagem curta, descrevendo o que vai fazer, com que skills, e o impacto esperado.
2. **Aguarda confirmação do usuário** ("ok", "pode", "segue", "vai", "manda"). Se o usuário pedir ajuste ou pular, respeitar.
3. **Reporta o resultado ao final de cada etapa** antes de anunciar a próxima (ou de encerrar, se foi etapa única).
4. **Para se algo der errado.** Não tenta a próxima etapa se a anterior falhou.

## Modos de invocação

### Modo A — Pipeline completo

Roda Etapa 1 → 2 → 3 → 4, com anúncio e confirmação entre cada uma.

### Modo B — Etapa específica

Detectar qual etapa o usuário quer pelo trigger:

| Pedido do usuário | Etapa |
|---|---|
| "etapa 1", "só Downloads", "só limpeza superficial" | 1 |
| "etapa 2", "só varredura", "só limpeza profunda", "só caches" | 2 |
| "etapa 3", "só PARA", "pula direto pras pastas mães", "só estrutura" | 3 |
| "etapa 4", "só refinamento", "só diagnóstico" | 4 |

**Verificar pré-requisitos antes de pular:**

- **Etapa 2** pressupõe que Downloads não está mais inchada (Etapa 1 feita ou desnecessária). Se Downloads tem 500+ arquivos misturados, avisar: "Você está pulando direto pra Etapa 2, mas Downloads tem N arquivos bagunçados. Recomendo rodar Etapa 1 antes. Forçar?"
- **Etapa 3** pressupõe que a home não está cheia de candidatos óbvios. Se há node_modules de vários GB ou pastas vazias na raiz, avisar.
- **Etapa 4** pressupõe que a super-mãe `00_<nome>/` existe. Se não existir, **bloquear** com mensagem: "Etapa 4 precisa de uma super-mãe já criada (Etapa 3). Quer rodar a Etapa 3 antes, ou apontar onde está a super-mãe?"

Se o usuário insistir em pular pré-requisito, respeitar (avisar uma vez, depois prosseguir).

### Modo C — Múltiplas etapas mas não todas

Ex: "roda etapa 2 e 3" ou "pula a Etapa 1, faz o resto".

Confirmar a sequência ("vou rodar 2 → 3 → 4, ok?") e rodar normalmente, com anúncio entre cada.

## Estrutura — 4 etapas

### Etapa 1 — Limpeza superficial (Downloads)

**Anúncio modelo:**
> "Vou começar a **Etapa 1 — Limpeza de Downloads**. Três passos encadeados:
> 1. Organizar `~/Downloads/` por tipo (Documentos, PDFs, Imagens, Vídeos, etc.)
> 2. Identificar candidatos a apagar (duplicatas, instaladores, lixo web 15+ dias)
> 3. Apagar candidatos (após você revisar)
>
> Reversível até a etapa de apagar. Ok seguir?"

**Skills internas:**
- `organizar-pasta-por-tipo` → `~/Downloads/`
- `filtrar-candidatos-apagar` → segunda passada em cada subpasta `_Organizado/`
- `apagar-candidatos` → após mostrar contagem e pedir confirmação explícita ("apaga?")

**Critério de sucesso:** Downloads tem só `_Organizado/` na raiz, candidatos foram para Lixeira.

### Etapa 2 — Limpeza profunda (home inteira)

**Anúncio modelo:**
> "Etapa 1 ok. Agora vou **Etapa 2 — Limpeza sistêmica da home**:
> 1. Apagar direto pastas vazias/irrelevantes do raiz (sem confirmar — só se forem realmente vazias)
> 2. Criar `~/Candidatos_a_Apagar/` com 4 subpastas-justificativa: Cache_e_Build, Duplicatas, Rascunhos_e_Testes, Midia_Antiga
> 3. Varrer cada pasta da home, mover candidatos com critério explícito (caches de node_modules, screenshots antigos, brutas de projetos encerrados, etc.)
> 4. Apagar candidatos (após revisão)
>
> Antes de começar, confirma o que NÃO posso tocar — geralmente: Olavo, música, documentos importantes, Library, Cripto. Algum acréscimo?"

**Skills internas:**
- `varrer-home-candidatos` (inclui apagar pastas vazias direto + criar Candidatos + varrer + mover)
- `apagar-candidatos` (após revisão)

**Critério de sucesso:** home enxugada (multi-GB liberados), candidatos na Lixeira ou em `Candidatos_a_Apagar/` aguardando revisão.

### Etapa 3 — Reestruturação (PARA + super-mãe)

**Anúncio modelo:**
> "Limpeza pronta. Agora **Etapa 3 — Reestruturação no padrão PARA**:
> 1. Agrupar as pastas-projeto da home em 4 mães numeradas: `01_Projetos/`, `02_Conteúdo/`, `03_Vida/`, `04_Recursos/`
> 2. Criar uma super-mãe (sugiro `00_<seu_nome>/`) para isolar isso das pastas-sistema do macOS
> 3. Se houver workspace de IA (Claude Cowork, etc.), incluir como peer dentro da super-mãe e atualizar o CLAUDE.md dele com rotas para as pastas-irmãs
>
> Ainda reversível. Qual nome quer pra super-mãe?"

**Skills internas:**
- `aplicar-sistema-para` (classifica e move 19+ pastas para 4 mães)
- `criar-super-mae` (cria umbrella, move 4 mães + Cowork, atualiza CLAUDE.md/MEMORY.md)

**Decisões a coletar antes:** nome da super-mãe, se inclui o Cowork, lista de pastas intocáveis.

**Critério de sucesso:** home tem só `<super-mãe>/` + pastas-sistema + `Candidatos_a_Apagar/`.

### Etapa 4 — Refinamento (diagnóstico + Opção A)

**Anúncio modelo:**
> "Estrutura montada. Última etapa — **Refinamento interno**:
> 1. Vou analisar a super-mãe e detectar inconsistências (sobreposições, nomes ruins, pastas no lugar errado, sacos-de-gato)
> 2. Vou apresentar duas opções de correção — uma enxuta e uma ambiciosa — com recomendação
> 3. Você escolhe; eu executo
>
> Ao final desta etapa, vou criar `WORK_IN_PROGRESS.md` e `PROMPTS_DE_ORGANIZACAO_DE_PASTAS.md` na super-mãe para você poder continuar em outra sessão. Ok seguir?"

**Skills internas:**
- `refinar-estrutura-interna` (diagnóstico + 2 opções + execução)

**Sempre cria ao final:**
- `<super-mãe>/WORK_IN_PROGRESS.md` — cronologia, estado, pendências
- `<super-mãe>/PROMPTS_DE_ORGANIZACAO_DE_PASTAS.md` — prompts da sessão

**Critério de sucesso:** estrutura interna consistente, documentos de continuidade gravados.

## Convenções da super-skill

- **Idioma:** Português, exceto código
- **Tom:** Direto. Anúncios curtos. Sem florear.
- **Confirmação entre etapas:** breve ("ok seguir?"). Não pedir confirmação dentro de uma etapa, exceto para `apagar-candidatos` (irreversível-ish).
- **Reportagem ao final de etapa:** 1-2 frases com o ganho concreto (espaço liberado, contagem de itens organizados).
- **Manifesto e reverter por etapa:** cada skill interna já mantém o próprio. A super-skill **não cria manifesto único** — preserva a granularidade para reversão seletiva.
- **Em caso de erro:** parar, reportar, perguntar como prosseguir. Não tentar a próxima etapa.

## Lista de reverter ao final

Sempre encerrar a sessão listando os scripts e manifestos criados:

```
~/_MANIFESTO_REORGANIZACAO.tsv  +  ~/_reverter_reorganizacao.sh
~/_MANIFESTO_SUPERMAE.tsv       +  ~/_reverter_supermae.sh
~/_MANIFESTO_REFINAMENTO.tsv    +  ~/_reverter_refinamento.sh
~/Candidatos_a_Apagar/_MANIFESTO_REVERSAO.tsv  +  ~/Candidatos_a_Apagar/_reverter.sh
```

E lembrar a ordem de reversão: **do mais recente para o mais antigo** (Etapa 4 → 3 → 2 → 1).

## Anti-padrão

- **Não** rodar etapas em sequência sem anúncio (mesmo etapa única deve anunciar)
- **Não** executar `apagar-candidatos` sem confirmação explícita do usuário
- **Não** assumir o nome da super-mãe — perguntar
- **Não** mover pastas declaradas intocáveis (Olavo, música, documentos importantes)
- **Não** criar manifesto único — cada etapa precisa do próprio (reversão granular)
- **Não** repetir trabalho — se uma etapa já foi feita, perguntar se quer pular ou refazer
- **Não** bloquear cegamente quando o usuário pula pré-requisito — avisar uma vez, deixar decidir
- **Não** rodar Etapa 4 sem super-mãe existir — esse é o único pré-requisito **bloqueante**, porque a skill `refinar-estrutura-interna` precisa de alvo
