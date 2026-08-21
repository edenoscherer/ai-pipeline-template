# CLAUDE.md

Este arquivo orienta o Claude Code (claude.ai/code) ao trabalhar neste
repositório.

> Copiado de `CLAUDE.example.md` do pacote de pipeline. Preencha os
> placeholders `<...>` abaixo e adicione as seções específicas do seu
> projeto ao final. Se este projeto já tinha um `CLAUDE.md`, mescle só
> a seção "Este projeto usa um pipeline formal" nele.

## `<Nome do projeto>`

<1-2 frases: o que este projeto é e faz.>

## Este projeto usa um pipeline formal: specify → plan → tasks → implement → review-pr

Features e correções passam pelos comandos abaixo, nesta ordem. Cada
comando é autodescritivo — para saber exatamente o que um deles faz,
leia `.claude/commands/<nome>.md` diretamente; não redocumente o
comportamento aqui, porque desatualiza.

| Comando | Quando usar |
|---|---|
| `/specify` | Nova feature — o quê/por quê, nunca o como |
| `/specify-tech` | Bug, débito técnico ou refatoração |
| `/plan` | Desenhar o como, a partir da spec |
| `/tasks` | Quebrar o plano em tasks ordenadas e testáveis |
| `/implement` | Executar as tasks, comitando por task |
| `/review-pr` | Revisar e aprovar merge (exige aprovação humana explícita) |
| `/pipeline-status` | Visão geral de todas as features em andamento |
| `/docs-sync` | Atualizar `docs/features/<domínio>.md` (roda automático no review-pr) |
| `/pipeline-doctor` | Checar a saúde da configuração do pipeline em si |

## Configuração do pipeline

`.pipeline/config.md` é a fonte única dos parâmetros deste projeto —
nunca assuma idioma, stack ou caminho fora dele. Os documentos que ele
referencia:

- `ARQUIVO_REGRAS` (`<caminho, ex.: memory/constitution.md>`) —
  princípios de engenharia não-negociáveis deste projeto.
- `ARQUIVO_ARQUITETURA` (`<caminho, ex.: docs/arquitetura.md>`) —
  decisões técnicas e desenho do sistema.
- `ARQUIVO_PRODUTO` (`<caminho, ex.: docs/product.md>`) — visão de
  produto: problema, público-alvo, escopo de MVP.

## Regras não-negociáveis herdadas do pipeline

- **Nunca edite `.claude/commands/*.md`** para inserir conteúdo deste
  projeto (stack, exemplos, nomes de módulo). Se um comando parecer
  precisar disso, a informação pertence a `ARQUIVO_REGRAS` ou
  `ARQUIVO_ARQUITETURA` — é isso que mantém o pacote atualizável.
- `.pipeline/state/*.json` é **intencionalmente comitado**, não
  ignorado pelo git — permite retomar uma feature em outra
  máquina/sessão. Não adicione ao `.gitignore`.
- `/review-pr` **sempre** exige aprovação humana explícita antes de
  escrever no GitHub, independente de `MODO_EXECUCAO`.
- O fechamento de uma feature aprovada (estado → `done`, roadmap,
  decisions-log, docs-sync, documentos de status) entra como commit na
  própria branch da PR, revisado junto com o resto — nunca como commit
  direto na branch principal depois do merge. Isso vale com ou sem
  branch protection ativa.
- Se houver mais de uma feature em estado ambíguo (`.pipeline/state/`
  com múltiplos arquivos incompletos), pergunte ao usuário qual retomar
  em vez de adivinhar.

## Regras de engenharia deste projeto

Regras de código (convenções, padrões DEVE/NÃO DEVE) vivem em
`ARQUIVO_REGRAS`, não aqui — este arquivo orienta *como trabalhar no
repositório e com o pipeline*, a constitution define *como o código
deve ser escrito*. Duplicar regras de engenharia aqui cria divergência
entre os dois documentos com o tempo.

## `<Seções específicas deste projeto>`

<Build, testes, lint, comandos de desenvolvimento, arquitetura de
alto nível — o que for específico deste projeto e não vier de nenhum
dos arquivos referenciados acima.>
