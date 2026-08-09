# CLAUDE.md

Este arquivo orienta o Claude Code (claude.ai/code) ao trabalhar neste repositório.

## Escopo deste repositório — leia antes de qualquer tarefa

> Este repositório serve **só para autorar e evoluir as pipelines e agentes** (os comandos em `.claude/commands/`, as skills em `.claude/skills/`, e os templates em `.pipeline/`). Ele **não é** onde desenvolvimento de produto acontece. O pacote inteiro é depois **copiado** para outros projetos, e é *lá* — não aqui — que `/specify`, `/plan`, `/tasks`, `/implement` etc. rodam sobre código real.

Consequências práticas disso para qualquer instância do Claude Code atuando aqui:

- Um pedido do tipo "implementa X" ou "corrige o bug Y" neste repo quase certamente significa **editar o texto de um comando/skill/config** para que ele produza X ou evite Y quando rodar em um projeto-alvo — não escrever código de aplicação, porque não há aplicação aqui.
- Nunca adicionar código-fonte de produto, dependências, `package.json`, `src/`, etc. a este repositório — isso vazaria escopo de "metodologia" para "projeto específico", exatamente o tipo de acoplamento que este pacote existe para evitar.
- `specs/` (`SPECS_DIR`) e `.pipeline/state/*.json` deste repositório, se surgirem, referem-se a specs sobre **a própria pipeline** (ex.: "adicionar um oitavo comando"), nunca a specs de produto — o produto vive nos projetos-alvo, cada um com seu próprio `specs/` depois de copiado o pacote.

## O que é este repositório

Isto **não é uma aplicação** — é um pipeline portável e agnóstico de stack, feito de comandos e skills do Claude Code, que implementa o fluxo `specify → plan → tasks → implement → review-pr`, pensado para ser copiado inteiro para outros projetos. Não há código-fonte, build, gerenciador de pacotes ou suíte de testes aqui; "desenvolver" neste repo significa editar os próprios arquivos markdown de comando/skill/config.

O desenho responde a duas falhas reais observadas em pipelines em produção: comandos com stack/idioma hardcoded que depois divergiram da constitution real do projeto, e múltiplos sistemas de spec-kit coexistindo de forma inconsistente no mesmo repo. Cada decisão de design abaixo existe para evitar um desses dois problemas.

## Trabalhando neste repo (sem build/test/lint)

Não há comandos de qualidade de código para rodar aqui — mudanças são validadas lendo o markdown, não executando nada. Quando pedirem para "adotar este pipeline" em um projeto-alvo, o procedimento é:

1. Copiar `.gitignore`, `.pipeline/` e `.claude/` (ou `.cursor/`) inteiros para o projeto-alvo (mesclar entradas do `.gitignore` se já existir um lá).
2. Editar **apenas** `.pipeline/config.md` no projeto-alvo (idioma, caminhos dos docs de regras/arquitetura, modo de execução, etc.).
3. Preencher `.pipeline/quality-gates.md` com os comandos reais de typecheck/test/lint/build daquele projeto.
4. Opcionalmente popular `.pipeline/roadmap.md` com specs planejadas.
5. **Nunca** editar `.claude/commands/*.md` para injetar conteúdo específico de projeto (stack, exemplos de código, nomes de módulo). Se um comando parecer precisar disso, a informação pertence a `ARQUIVO_REGRAS`/`ARQUIVO_ARQUITETURA` (docs próprios do projeto-alvo) — é isso que mantém o pacote portável para o *próximo* projeto também.

Neste repositório especificamente, `.pipeline/config.md` ainda aponta para placeholders (`memory/constitution.md`, `docs/arquitetura.md`) que não existem, e `.pipeline/quality-gates.md` tem células `<preencher>` não preenchidas — isso é esperado para o template em si, não é um bug a corrigir.

## Arquitetura

### Fonte única de configuração do projeto

`.pipeline/config.md` é o **único** lugar de onde qualquer comando pode ler parâmetros específicos de projeto (idioma, caminhos dos docs de produto/regras/arquitetura, quality gates, modo de execução, layout de diretórios). Nenhum comando pode ter idioma, stack ou caminho hardcoded — essa é a regra que mantém o pacote copiável entre projetos. Parâmetros principais: `IDIOMA_ARTEFATOS` (idioma dos artefatos gerados — atualmente `pt-BR`), `ARQUIVO_PRODUTO` (visão de produto — problema, público-alvo, diferencial, escopo de MVP; lido só por `/specify` e `software-dev-panel`, não pelos comandos técnicos), `ARQUIVO_REGRAS`/`ARQUIVO_ARQUITETURA` (docs de constitution/arquitetura do projeto), `MODO_EXECUCAO` (`supervisionado` = para ao fim de cada fase, `encadeado` = avança sozinho), `MAX_PERGUNTAS_CLARIFICACAO`, `SPECS_DIR`, `ESTADO_DIR`, `COMMIT_POR_FASE`, `ARQUIVO_ROADMAP`, `ARQUIVO_DECISIONS_LOG`, `DOCS_FEATURES_DIR`, `ARQUIVOS_STATUS`.

### Os nove comandos (`.claude/commands/`)

Todos rodam como comando direto, deliberadamente **não** como subagent — o pipeline depende de pontos de interação (perguntas de clarificação no `/specify`, revisão de plano/tasks, aprovação obrigatória no `/review-pr`) que um subagent isolado, que só devolve um resultado final, quebraria.

`/plan`, `/tasks`, `/implement` e `/review-pr` abrem com uma seção `## Pré-condições` — checagem explícita e numerada que bloqueia com mensagem clara (qual condição falhou) em vez de o modelo decidir implicitamente se pode prosseguir; `/review-pr` não bloqueia PRs sem `feature-state.json` associado (fora do pipeline), só pula a Etapa 8.

| Comando | Papel | Produz | Lê estado, escreve `current_phase` → |
|---|---|---|---|
| `/specify` | Product Owner — o *quê/por quê*, nunca o *como* | `spec.md` | `plan` |
| `/specify-tech` | Spec técnica para bugs/débito técnico/refatoração — checa `docs/features/<dominio>.md` por recorrência antes de tratar um bug como novo | `spec.md` técnica | `plan` |
| `/plan` | Arquiteto — o *como*, validado contra `ARQUIVO_REGRAS`/`ARQUIVO_ARQUITETURA` | `research.md`, `data-model.md`, `contracts/`, `quickstart.md` | `tasks` |
| `/tasks` | QA/Tech Lead — quebra o plano em tasks ordenadas e testáveis (Setup → Testes → Core → Integração → Polish, ordem TDD, `[P]` para paralelizável); grava `task_progress.total` no estado | `tasks.md` | `implement` |
| `/implement` | Dev — executa as tasks em ordem, comita por task, roda quality gates, atualiza `task_progress.completed`/`.failed`, registra decisões não previstas em `research.md` | código + commits | `review` |
| `/review-pr` | Revisor sênior — **sempre** exige aprovação humana explícita antes de escrever no GitHub, independente do `MODO_EXECUCAO`; Etapa 5 separa evidência mecânica (`quality_gates`) de julgamento do revisor (`review_judgment`) num bloco estruturado; pós-merge (Etapa 8) atualiza estado/roadmap/decisions-log/docs-sync | review + rastreio de merge | `done` |
| `/pipeline-status` | Somente leitura. Cruza `<ESTADO_DIR>/*.json` contra `ARQUIVO_ROADMAP`, reporta divergências, progresso de tasks e features em estado de exceção | relatório de status | — |
| `/docs-sync` | Atualiza incrementalmente `docs/features/<dominio>.md` a partir de uma spec concluída; chamado automaticamente pelo `/review-pr` Etapa 8 | atualização de doc de domínio | — |
| `/pipeline-doctor` | Somente leitura. Verifica a saúde da **configuração do pipeline em si** (não o progresso de nenhuma feature) — caminhos configurados que existem, quality-gates preenchido, comandos/skills presentes, versão do `.pipeline/version` | relatório de saúde | — |

Todos os cinco comandos que avançam fase (`specify`, `specify-tech`, `plan`, `tasks`, `implement`) reconhecem, a qualquer momento, um pedido explícito do usuário para marcar a feature como `blocked`/`cancelled`/`failed` (`status_detail` com o motivo) — nunca inferem essa condição sozinhos. Ver `feature-state.schema.md`.

### Duas skills (`.claude/skills/`), não comandos — disparadas por intenção, não por `/nome`

- **`clarification-protocol`** — processo de perguntas compartilhado por `/specify` e `/specify-tech`: recomendação primeiro, tabela com 2-4 opções, aceita "sim"/"recomendado" como atalho, respeita `MAX_PERGUNTAS_CLARIFICACAO` para a sessão inteira, marca lacunas não resolvidas como `[PRECISA ESCLARECIMENTO: ...]` ou `[PRECISA INVESTIGAÇÃO: ...]` em vez de travar o fluxo.
- **`software-dev-panel`** — painel de discussão multi-persona (CodeGPT, PM/PO, Architect, Programmer, Questioner, Critic, Topic Expert) auto-disparado por linguagem natural ("quero refinar isso", "pensar em voz alta sobre..."), não por comando. Escala de 3 especialistas fixos (Modo Refinamento) até os 7 (Modo Completo). Lê `ARQUIVO_REGRAS`/`ARQUIVO_ARQUITETURA` em tempo real, igual aos comandos. No fechamento, pode encaminhar para `/specify`, `/specify-tech` ou — se ainda não existir `config.md`/arquivo de regras nenhum — gerar `constitution.md`, `docs/arquitetura.md` e `.pipeline/config.md` iniciais a partir da discussão. Nunca gera artefato nenhum sem confirmação explícita do usuário sobre qual caminho seguir.

### Estado persistido (`.pipeline/state/<slug>.json`, schema em `feature-state.schema.md`)

Um JSON por feature ativa — **intencionalmente comitado**, não ignorado pelo git, para permitir retomar uma feature em outra máquina/sessão. Todo comando lê `feature_dir`/`branch`/`current_phase` daqui primeiro, e nunca re-deriva ou pergunta de novo se o arquivo de estado já existir sem ambiguidade. Se houver mais de uma feature em andamento, `/pipeline-status` lista todas e pergunta qual retomar — nenhum comando escolhe sozinho em caso de ambiguidade.

### Três artefatos de "histórico" diferentes — não confundir

- **`.pipeline/roadmap.md`** — visão agregada e legível por humanos de todas as specs planejadas e seu status (🔲/🟡/✅, mais 🚧/⛔/❌ para os estados de exceção — mapeamento direto de `current_phase`), atualizada automaticamente pelos comandos a cada fim de fase. Opcional, mas recomendado desde o início.
- **`.pipeline/decisions-log.md`** — log cronológico de decisões tomadas *durante a implementação* que não estavam no plano original, adicionado uma vez por spec pelo `/review-pr` Etapa 8 (pós-merge). Lido só sob demanda, nunca carregado automaticamente. Entradas recorrentes aqui são sinal de que o padrão deveria virar princípio formal em `ARQUIVO_REGRAS`.
- **`docs/features/<dominio>.md`** — documentação viva do comportamento atual de cada módulo (não histórico), um arquivo por domínio, atualizado incrementalmente por `/docs-sync`. Cada doc mantém uma tabela "Specs Relacionadas" usada pelo `/specify-tech` para detectar bugs recorrentes na mesma área antes de investigar do zero.

### Regras não-configuráveis (não tornar condicionais ao `MODO_EXECUCAO`)

- `/review-pr` nunca escreve no GitHub sem aprovação explícita do usuário sobre o relatório de review antes.
- Comandos nunca têm stack/idioma/caminho de projeto hardcoded — sempre resolvem via `.pipeline/config.md`.
- Quando o estado de feature for ambíguo (múltiplas features em andamento), perguntar ao usuário em vez de adivinhar.
