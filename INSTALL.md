# Instalando este pipeline em um projeto

Este arquivo é o manifesto de adoção: o que copiar deste repositório
para um projeto-alvo, o que **não** copiar, e os passos de configuração
depois de copiar. Ele mesmo **não é copiado** — fica só aqui, no
repositório de autoria (ver "Não copiar" abaixo).

## O que copiar

Todo arquivo/diretório deste repositório se enquadra em um destes
quatro grupos. Nenhum arquivo fica de fora da lista.

### 1. Metodologia — copiar e nunca editar no projeto-alvo

```
.claude/commands/specify.md
.claude/commands/specify-tech.md
.claude/commands/plan.md
.claude/commands/tasks.md
.claude/commands/implement.md
.claude/commands/review-pr.md
.claude/commands/pipeline-status.md
.claude/commands/docs-sync.md
.claude/commands/pipeline-doctor.md
.claude/skills/clarification-protocol/SKILL.md
.claude/skills/software-dev-panel/SKILL.md
.pipeline/feature-state.schema.md
.pipeline/version
```

Estes arquivos não têm stack, idioma ou domínio hardcoded — toda
informação específica de projeto vem de `.pipeline/config.md`. Editar
qualquer um deles para inserir conteúdo do projeto-alvo (nome de
módulo, exemplo de código, framework) é o acoplamento que este pacote
existe para evitar; a informação pertence a `ARQUIVO_REGRAS` ou
`ARQUIVO_ARQUITETURA`. Sendo puramente metodologia, podem ser
re-copiados por cima quando este template evoluir (`.pipeline/version`
subir), sem perda de conteúdo do projeto.

**Exceção**: `.claude/skills/software-dev-panel/SKILL.md` tem uma
seção "Sobre Você" no topo, feita para ser editada **uma vez** com
contexto pessoal (nível, preferências de comunicação) — não é
conteúdo de projeto, mas também não deve ser sobrescrita numa
atualização futura do template como o resto do grupo. Ao re-copiar
este arquivo por cima numa atualização, preserve manualmente essa
seção antes de sobrescrever o restante.

### 2. Esqueleto — copiar e preencher no projeto-alvo

```
.pipeline/config.md            → editar campo a campo (ver abaixo)
.pipeline/quality-gates.md     → trocar <preencher> pelos comandos reais
.pipeline/roadmap.md           → trocar <Nome do Projeto> e as linhas de exemplo
.pipeline/decisions-log.md     → começa vazio, populado por /review-pr
.pipeline/state/               → diretório vazio (só .gitkeep)
docs/features/_template.md     → fica como referência para /docs-sync
.gitignore                     → MESCLAR com o do projeto-alvo, nunca sobrescrever
CLAUDE.example.md              → copiar RENOMEANDO para CLAUDE.md (ver seção própria)
```

Diferente do grupo 1, estes arquivos são o ponto de partida — depois
de copiados, passam a ter conteúdo específico do projeto-alvo e **não**
devem ser sobrescritos numa atualização futura do template.

### 3. Não copiar — específico deste repositório de autoria

```
README.md
CLAUDE.md
INSTALL.md
specs/                          (se existir aqui)
.pipeline/state/*.json          (se existir aqui, além do .gitkeep)
```

Estes arquivos descrevem **este** repositório — o pacote que autora e
evolui a pipeline — não um projeto que a usa. Em especial, **nunca
copie o `CLAUDE.md` deste repositório**: ele instrui explicitamente
qualquer Claude Code a tratar o repositório como "não é uma aplicação"
e a "nunca adicionar código-fonte de produto, `package.json`, `src/`".
Num projeto real, isso faria o agente recusar o próprio trabalho que
deveria fazer. Se `specs/` ou `.pipeline/state/*.json` existirem aqui,
são specs sobre a própria pipeline (ex.: "adicionar um décimo
comando"), não sobre o produto do projeto-alvo.

O projeto-alvo não fica sem `CLAUDE.md` por causa disso — ver
`CLAUDE.example.md` no grupo 2.

### 4. Nunca copiar — segurança

```
.claude/settings.local.json
.cursor/settings.local.json
.env / .env.local / .env.*.local
```

Não fazem parte do pacote em nenhum projeto — são locais de cada
máquina/desenvolvedor. `.claude/settings.local.json` em particular
acumula allowlist de comandos Bash que frequentemente embutem
credenciais em texto puro conforme o histórico de uso cresce (ver
comentário no `.gitignore` deste repo). Se você clonar este repositório
inteiro em vez de copiar arquivo por arquivo, confirme que nenhum
desses três chegou a existir aqui antes de usar o clone como base.

## Passos de configuração pós-cópia

1. Copie `.gitignore`, `.pipeline/` e `.claude/` (ou `.cursor/`)
   inteiros para o projeto-alvo, seguindo o manifesto acima — mescle
   `.gitignore` se o projeto já tiver um.
2. Copie `CLAUDE.example.md` como `CLAUDE.md` na raiz do projeto-alvo
   (ou mescle a seção do pipeline nele, se já existir um `CLAUDE.md`
   lá).
3. Edite **apenas** `.pipeline/config.md`:
   - `ARQUIVO_PRODUTO` — aponte para o documento de visão de produto
     (problema, público-alvo, diferencial, escopo de MVP). Lido por
     `/specify` e pelo `software-dev-panel`, não pelos comandos
     técnicos. Se ainda não existir, crie-o separadamente ou deixe
     vazio se o projeto não quiser manter esse documento.
   - `ARQUIVO_REGRAS` e `ARQUIVO_ARQUITETURA` — aponte para os
     documentos de constitution/arquitetura do projeto. Se ainda não
     existirem, crie-os separadamente (não fazem parte deste pacote —
     são conteúdo específico de projeto, não metodologia).
   - `ARQUIVO_ROADMAP` — mantenha `.pipeline/roadmap.md` se quiser
     rastrear specs planejadas desde o início, ou deixe vazio.
   - `ARQUIVO_DECISIONS_LOG` — mantenha `.pipeline/decisions-log.md`
     ou deixe vazio se preferir manter isso só em `research.md`.
   - `DOCS_FEATURES_DIR` — mantenha `docs/features/` ou deixe vazio.
   - `IDIOMA_ARTEFATOS` — ajuste se não for pt-BR.
   - `MODO_EXECUCAO` — `supervisionado` (default, mais seguro) ou
     `encadeado` (avança sozinho entre fases).
4. Preencha `.pipeline/quality-gates.md` com os comandos reais do
   projeto (typecheck, test, lint, build).
5. Se for usar `ARQUIVO_ROADMAP`, preencha `.pipeline/roadmap.md` com
   as specs já planejadas (ou deixe o `software-dev-panel` gerar isso
   ao final de uma discussão de fundação).
6. Rode `/pipeline-doctor` para confirmar que a configuração está
   correta antes de rodar `/specify` pela primeira vez — ele aponta
   caminhos configurados que não existem, comandos/skills que faltaram
   na cópia, e `quality-gates.md` ainda com `<preencher>`.
7. **Nunca edite os arquivos dentro de `.claude/commands/`** para
   inserir conteúdo específico do projeto. Se sentir essa necessidade,
   é sinal de que a informação deveria estar em `ARQUIVO_REGRAS` ou
   `ARQUIVO_ARQUITETURA`, não no comando.

## Variante Cursor

Se o projeto-alvo usa Cursor em vez de Claude Code, copie `.claude/`
como `.cursor/` mantendo a mesma estrutura interna (`commands/`,
`skills/`) — só a pasta muda de nome, o conteúdo e a sintaxe de
invocação seguem o adaptado pelo próprio Cursor.

## Se o projeto-alvo já tiver outro pipeline/spec-kit

Decida qual fica e remova o outro — não mantenha os dois ativos ao
mesmo tempo. Rodar dois sistemas de spec-kit em paralelo no mesmo
projeto é exatamente uma das duas falhas reais que motivaram este
pacote (specs com convenções inconsistentes entre si, sem ninguém
notando qual é a fonte da verdade).

## Atualizando depois de uma nova versão do template

Quando este repositório evoluir (`.pipeline/version` subir), o grupo 1
("Metodologia") pode ser re-copiado por cima da cópia existente no
projeto-alvo sem perda — são arquivos sem conteúdo específico de
projeto. O grupo 2 ("Esqueleto") nunca deve ser sobrescrito dessa
forma, pois já contém configuração e histórico do projeto-alvo;
mudanças nele (ex.: um campo novo em `config.md`) precisam ser
aplicadas manualmente, comparando com a versão nova deste repositório.