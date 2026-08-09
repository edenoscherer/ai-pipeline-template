# Pipeline Genérico: Specify → Plan → Tasks → Implement → Review

Conjunto portável de comandos para Claude Code (ou Cursor, adaptando a
sintaxe de invocação e a pasta `.claude/` para `.cursor/`) que
implementa o fluxo specify → plan → tasks → implement → review-pr, sem
acoplamento a stack, idioma ou domínio de projeto específico.

Desenhado a partir da análise comparativa de dois pipelines reais em
produção (um com contradição de stack entre comandos e constitution;
outro com quatro sistemas de spec-kit coexistindo e usados de forma
inconsistente) — as decisões de design abaixo existem especificamente
para evitar essas duas falhas se repetirem.

## Estrutura

```
.gitignore                     # protege settings.local.json e .env de vazar
.pipeline/
  config.md                    # único lugar com parâmetros de projeto
  decisions-log.md             # log agregado de decisões (opcional)
  feature-state.schema.md      # formato do estado por feature
  quality-gates.md             # comandos de verificação (preencher)
  roadmap.md                   # lista mestre de specs (opcional, mas recomendado)
  state/                       # gerado automaticamente, um json por feature
docs/
  features/
    _template.md                # estrutura de referência para novos docs de domínio
    <dominio>.md                 # um por módulo/bounded context (gerado por /docs-sync)
.claude/
  commands/                    # (ou .cursor/commands/, conforme a ferramenta)
    specify.md
    specify-tech.md
    plan.md
    tasks.md
    implement.md
    review-pr.md
    pipeline-status.md
    docs-sync.md
  skills/
    clarification-protocol/    # lógica de perguntas compartilhada por
      SKILL.md                 # specify e specify-tech (evita duplicação)
    software-dev-panel/        # painel de discussão/refinamento — auto-
      SKILL.md                 # disparado por linguagem natural, não por comando
```

## Por que uma Skill e não um Subagent

Nenhum comando deste pacote precisa rodar como **subagent**
(`.claude/agents/`) — subagents executam em contexto isolado e só
devolvem um resultado final, o que quebraria os pontos de interação que
o pipeline depende (perguntas de clarificação em `/specify`, revisão de
plano/tasks antes de avançar, aprovação obrigatória em `/review-pr`).
Comando direto é a escolha certa para todos os 7.

Já a **Skill** `clarification-protocol` existe porque `/specify` e
`/specify-tech` precisam do mesmo processo de pergunta (recomendação +
tabela + atalho de aceite + limite de perguntas) — mantê-lo como skill
evita que os dois comandos fiquem com o mesmo texto copiado, e qualquer
ajuste no protocolo (ex.: mudar o formato da tabela) é feito uma vez só.

Se sua sessão de `/implement` começar a ficar pesada em projetos com
muitas tasks, considere rodá-lo como subagent isolado (padrão
"background agent") — mas isso é otimização opcional, não parte
obrigatória deste pacote.

## O skill `software-dev-panel`

Diferente dos 7 comandos, este skill **não é invocado por `/comando`**
— ele é auto-disparado pela relevância do que você escreve ("quero
refinar essa demanda", "pensar em voz alta sobre..."), igual ao
`clarification-protocol`, mas pensado para uso interativo direto pelo
usuário, não só como sub-rotina de outro comando.

Ele opera em 3 modos que escalam de leve (3 especialistas fixos) até
completo (7 especialistas), com o painel decidindo sozinho quando
precisa trazer mais gente pra discussão — você não precisa escolher
antecipadamente entre "pergunta rápida" e "convocar todo mundo".

**Portabilidade**: assim como os comandos, ele nunca tem stack ou
domínio de negócio escrito no próprio texto — lê `ARQUIVO_REGRAS` e
`ARQUIVO_ARQUITETURA` (via `.pipeline/config.md`) em tempo real. Isso
permite manter uma única cópia dele em `~/.claude/skills/` (global,
segue você entre todos os projetos) sem precisar editar o arquivo a
cada projeto novo — só a seção "Sobre Você" (nível, preferências
pessoais) é conteúdo fixo, porque isso legitimamente não muda entre
projetos.

**Fecha o loop com o pipeline**: ao final de uma discussão, o painel
pergunta se você quer formalizar como `/specify`, `/specify-tech`, ou
— se a conversa foi sobre fundar um projeto do zero — gerar os
arquivos iniciais (`ARQUIVO_REGRAS`, `ARQUIVO_ARQUITETURA` e
`.pipeline/config.md`) diretamente a partir do que foi decidido. Isso
cobre o cenário que os comandos formais não cobrem: você ainda não tem
`config.md` nenhum, está literalmente começando o projeto, e quer
discutir antes de ter qualquer artefato pra formalizar.

## O arquivo `roadmap.md`

Diferente de `<ESTADO_DIR>/*.json` (estado técnico detalhado, um por
feature, pensado para os comandos lerem), `roadmap.md` é a visão
agregada e legível por humanos: lista todas as specs planejadas, em
ordem, com status simples. É opcional (`ARQUIVO_ROADMAP` pode ficar
vazio em `config.md`), mas recomendado desde o início — é comum
adiar a criação dele "pra depois" e isso nunca acontecer, deixando
specs esquecidas em progresso parcial sem ninguém perceber.

`/pipeline-status` cruza o roadmap com o estado real de cada feature e
aponta divergências (roadmap desatualizado, specs concluídas não
refletidas, specs feitas fora do planejamento) — é a mesma verificação
que teria evitado o cenário real que motivou este pacote: um projeto
com `IMPLEMENTATION_STATUS.md` desatualizado e cinco specs paradas em
0% sem ninguém notar.

## O arquivo `decisions-log.md`

Registra decisões técnicas/produto tomadas **durante a implementação**
que não estavam previstas no plano — deliberadamente separado do
`roadmap.md` para não comprometer sua leitura rápida. Duas camadas:

1. **Por feature**: `research.md` ganha uma seção "Decisões durante a
   implementação", preenchida pelo `/implement` quando algo surge fora
   do planejado (workaround de biblioteca, edge case descoberto na
   hora). Fica co-localizado com a spec, sem arquivo novo.
2. **Agregado**: `/review-pr` (Etapa 8, pós-merge) adiciona uma entrada
   curta em `.pipeline/decisions-log.md` — um log cronológico de
   leitura ocasional, não carregado automaticamente por nenhum comando
   durante execução normal.

O valor do log agregado aparece com o tempo: se o mesmo tipo de
decisão se repete em várias entradas, é sinal de que deveria virar
princípio formal em `ARQUIVO_REGRAS`, em vez de continuar sendo
"redescoberto" spec a spec — o mesmo raciocínio de "o que você
reexplica toda sessão deveria estar no CLAUDE.md" aplicado a decisões
técnicas em vez de preferências de trabalho.

## A documentação viva em `docs/features/`

Diferente de `research.md` (decisões técnicas de UMA spec) e de
`decisions-log.md` (histórico cronológico agregado), `docs/features/`
responde uma pergunta diferente: **"como este módulo funciona hoje?"**
— organizado por domínio, não por ordem cronológica de spec.

- **Criado e atualizado incrementalmente** por `/docs-sync`, chamado
  automaticamente pelo `/review-pr` (Etapa 8, pós-merge). Nunca
  regenerado do zero — só a parte afetada pela spec concluída.
- **Feature nova** → adiciona/atualiza comportamento no doc do domínio.
- **Bug fix** → corrige o texto que descrevia o comportamento errado,
  e remove de "Limitações conhecidas" o que foi resolvido.
- **Tabela "Specs Relacionadas"** em cada doc de domínio funciona como
  índice de recorrência: se dois bug fixes na mesma área aparecem ali,
  é sinal de causa raiz não resolvida na primeira tentativa.

**Detecção automática de recorrência**: `/specify-tech` consulta essa
tabela **antes** de investigar um bug novo. Se encontrar um bug fix
anterior no mesmo domínio com sintoma parecido, ele lê a spec antiga,
usa a causa raiz documentada lá como hipótese de partida, e sinaliza
explicitamente que pode ser recorrência — em vez de tratar como
problema do zero e repetir a mesma investigação superficial.

**O que fica fora deste mecanismo** (de propósito, para não virar
carga obrigatória a cada spec): DFD, diagramas de classe/sequência,
diagrama de deployment, documento de segurança, plano de manutenção,
manual do usuário. Esses continuam sendo os "Entregáveis Disponíveis"
do `software-dev-panel` — gerados sob demanda quando fizer sentido
(auditoria, onboarding, maturidade do produto), não a cada feature ou
bug fix.

## Adoção em um projeto novo

1. Copie `.gitignore`, `.pipeline/` e `.claude/` (ou `.cursor/`)
   inteiros para o novo projeto — se o projeto já tiver um
   `.gitignore`, apenas mescle as entradas em vez de sobrescrever.
2. Edite **apenas** `.pipeline/config.md`:
   - `ARQUIVO_REGRAS` e `ARQUIVO_ARQUITETURA` — aponte para os
     documentos de constitution/arquitetura do projeto. Se ainda não
     existirem, crie-os separadamente (não fazem parte deste pacote —
     são conteúdo específico de projeto, não metodologia).
   - `ARQUIVO_ROADMAP` — mantenha `.pipeline/roadmap.md` se quiser
     rastrear specs planejadas desde o início, ou deixe vazio se
     preferir decidir isso mais tarde.
   - `ARQUIVO_DECISIONS_LOG` — mantenha `.pipeline/decisions-log.md`
     se quiser um histórico agregado de decisões de implementação, ou
     deixe vazio se preferir manter isso só em `research.md` por
     feature.
   - `DOCS_FEATURES_DIR` — mantenha `docs/features/` se quiser
     documentação viva por módulo, com detecção automática de bugs
     recorrentes, ou deixe vazio se preferir não manter isso.
   - `IDIOMA_ARTEFATOS` — ajuste se não for pt-BR.
   - `MODO_EXECUCAO` — `supervisionado` (default, mais seguro) ou
     `encadeado` (menos fricção, avança sozinho entre fases).
3. Preencha `.pipeline/quality-gates.md` com os comandos reais do
   projeto (typecheck, test, lint, build).
4. Se for usar `ARQUIVO_ROADMAP`, preencha `.pipeline/roadmap.md` com
   as specs já planejadas (ou deixe o `software-dev-panel` gerar isso
   ao final de uma discussão de fundação — ver seção abaixo).
5. **Nunca edite os arquivos dentro de `.claude/commands/`** para
   inserir conteúdo específico do projeto (stack, exemplos de código,
   nome de módulos). Se sentir essa necessidade, é sinal de que a
   informação deveria estar em `ARQUIVO_REGRAS` ou
   `ARQUIVO_ARQUITETURA`, não no comando — essa é a regra que garante
   que este pacote continue portável para o próximo projeto.

## O que fica de fora deste pacote (de propósito)

- **`constitution.md` / `docs/arquitetura.md`** — conteúdo específico
  de cada projeto, não fazem parte da metodologia. (Podem ser gerados
  pelo próprio `software-dev-panel` na opção "Arquivos Iniciais de
  Projeto", mas o conteúdo em si nasce da discussão, não do pacote.)
- **Consultor técnico especializado em referências de mercado** (o
  equivalente a `tech-expert.md` visto em projetos reais) — fica de
  fora porque normalmente é invocado como subagent dedicado por outro
  comando (ex.: um `/specify-tech` customizado poderia chamá-lo), e
  não é essencial ao fluxo specify→review. Se quiser adicionar, siga o
  mesmo princípio de portabilidade: sem stack/domínio hardcoded, tudo
  via `ARQUIVO_REGRAS`/`ARQUIVO_ARQUITETURA`.
- **Templates de spec/plan/tasks** — cada projeto pode manter os seus
  em `templates/`; os comandos leem o template se existir e usam
  estrutura mínima caso contrário.

## Modo de uso

```
[conversa livre, sem /comando]            → dispara software-dev-panel
                                             automaticamente se for
                                             refinamento/arquitetura/discussão

/specify <descrição da feature>          → spec.md
/specify-tech <descrição de bug/débito>  → spec técnica (alternativa ao /specify)
/plan                                     → research.md, data-model.md, contracts/
/tasks                                    → tasks.md
/implement                                → código + commits + quality gates
/review-pr <número da PR>                 → review + aprovação humana + merge tracking
/docs-sync <slug>                         → atualiza docs/features/ (roda automático no review-pr)
/pipeline-status                          → visão geral de todas as features
```

## Princípios de design (por que está assim)

1. **Nenhum comando conhece o projeto diretamente** — tudo passa por
   `.pipeline/config.md`. Isso evita o problema de comandos citando
   stack ou idioma errado depois que o projeto muda de direção.
2. **Estado persistido, nunca re-derivado** — `feature-state.json`
   elimina a necessidade de redigitar caminho de feature a cada chat
   novo, e permite que `/pipeline-status` saiba exatamente onde cada
   feature parou.
3. **Um único sistema, não vários coexistindo** — este pacote assume
   que será o único pipeline formal do projeto. Se o projeto já tiver
   outro (ex.: um spec-kit genérico importado por template), decida
   qual fica e remova o outro — não mantenha os dois ativos ao mesmo
   tempo, isso gera specs com convenções inconsistentes entre si.
4. **Aprovação humana em pontos irreversíveis é fixa, não configurável**
   — `review-pr` sempre pede confirmação antes de escrever no GitHub,
   independente de `MODO_EXECUCAO`. Automação de velocidade não deve
   remover controle humano sobre ações que saem do repositório local.
