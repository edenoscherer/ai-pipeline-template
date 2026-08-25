---
name: linear-sync
description: Lógica compartilhada de leitura e write-back de cards do Linear, usada por specify, specify-tech, plan, tasks, implement e review-pr. Opt-in via LINEAR_ENABLED em .pipeline/config.md, degrada graciosamente quando a integração está desligada ou a tool MCP do Linear não está conectada.
---

# Linear Sync

Objetivo: um único lugar para toda interação com o Linear, para que os
seis comandos que a usam não dupliquem a lógica de detecção,
disponibilidade e escrita — mesmo raciocínio pelo qual
`clarification-protocol` existe como skill em vez de texto repetido em
`/specify` e `/specify-tech`.

## Pré-condição: Linear está disponível?

Sempre a primeira coisa a checar, em qualquer seção abaixo. Duas
condições, **ambas** necessárias:

1. `LINEAR_ENABLED: true` em `.pipeline/config.md`.
2. Alguma tool MCP do Linear está presente **nesta sessão atual**.

Se qualquer uma falhar, a integração degrada graciosamente: comporte-se
exatamente como se `LINEAR_ENABLED: false` estivesse configurado — não
interrompa o pipeline, não reporte isso como erro, apenas prossiga sem
tocar o Linear. `LINEAR_ENABLED: true` no arquivo é necessário mas não
suficiente; a tool pode estar desconectada nesta sessão mesmo com a
configuração ligada, e isso não é uma condição de falha — é o caso
normal de degradação.

Todas as seções a seguir assumem que esta pré-condição já foi checada
e passou. Se não passou, pule a seção inteira sem executar nada dela.

## Detectar referência a um card

Usada por `/specify` e `/specify-tech` sobre a entrada do usuário
(`$ARGUMENTS` ou descrição livre), antes da lógica normal de
interpretar a entrada como descrição de feature/bug.

Reconhece, nesta ordem:
1. **URL completa do Linear**: contém `linear.app` e `/issue/`.
2. **Identificador nu**: `<LINEAR_TEAM_KEY>-<número>` (ex.: `EDE-123`),
   só se `LINEAR_TEAM_KEY` estiver configurado (não vazio). Sem
   `LINEAR_TEAM_KEY`, apenas URLs completas são reconhecidas.

Se nada for reconhecido (ou a pré-condição de disponibilidade não
passou), trate a entrada inteira como descrição livre — comportamento
idêntico ao que os comandos já têm hoje sem esta skill.

## Ler o card

1. Busque o card pelo identificador detectado.
2. Extraia título, descrição, labels e comentários. **Nunca invente**
   conteúdo ausente — se a descrição do card for insuficiente para uma
   spec completa, isso é uma lacuna normal e continua passando pela
   skill `clarification-protocol`, exatamente como uma descrição livre
   incompleta passaria.
3. Use título + descrição como entrada da fase de geração da spec
   (Passo 2 de `/specify` ou Passo 3 de `/specify-tech`), no lugar da
   descrição livre que o comando usaria.
4. Grave o identificador do card em `linear_issue_id` no
   `feature-state.json` da feature, na criação (Passo 1 de
   `/specify`/`/specify-tech`).
5. Se o card **não** tiver a label `LINEAR_LABEL_AGENT_TASK`, avise o
   usuário (ex.: *"O card `<id>` não tem a label `<LINEAR_LABEL_AGENT_TASK>`
   — confirma que quer mesmo usá-lo como origem desta spec?"*) e peça
   confirmação antes de prosseguir. Isso não bloqueia — é só um aviso
   de que o card pode não ter sido destinado a um agente.

## Write-back de progresso

Usada por `/specify`, `/specify-tech`, `/plan`, `/tasks` e `/implement`
no fechamento de fase, sempre que `linear_issue_id` (do
`feature-state.json` da feature) não for `null`.

- **specify / specify-tech**: comentário curto no card com resumo da
  spec gerada + link/caminho do artefato (`spec.md`).
- **plan**: comentário com resumo do plano + caminho dos artefatos
  gerados (`research.md`, `data-model.md`, `contracts/`, `quickstart.md`,
  os que existirem).
- **tasks**: comentário com resumo (nº de tasks geradas) + link do
  `tasks.md`.
- **implement**: comentário incluindo `task_progress`
  (`completed`/`failed`/`total`) e `quality_gates_status` do
  `feature-state.json`.

Em nenhum caso esta escrita muda o status/workflow state do card — só
adiciona um comentário. Sempre **best-effort**: se a escrita falhar
(rate limit, permissão, card removido), reporte como aviso ao usuário
e continue o pipeline normalmente — nunca pare uma fase já concluída
por causa de uma falha de write-back no Linear.

## Estados de exceção

Usada por `specify`, `specify-tech`, `plan`, `tasks` e `implement`
sempre que o usuário pedir explicitamente para marcar a feature como
`blocked`/`cancelled`/`failed` (ver `.pipeline/feature-state.schema.md`),
e `linear_issue_id` não for `null`.

- **Ao entrar em exceção**: comente `status_detail` no card. Se o
  estado for `blocked`, aplique a label `LINEAR_LABEL_BLOCKED`. Se o
  motivo (`status_detail`) sugerir uma decisão de produto ou
  arquitetura pendente (ex.: "esperando decisão sobre X"), aplique
  também `LINEAR_LABEL_DECISION_NEEDED`, independente de `blocked`,
  `cancelled` ou `failed`.
- **Ao sair de exceção**: comente a retomada no card (ex.: "Retomando
  — bloqueio resolvido") e remova as labels aplicadas no passo
  anterior.

Mesma regra de best-effort da seção anterior: falha de escrita vira
aviso, nunca bloqueia a transição de estado no `feature-state.json`
(que já aconteceu independentemente do Linear).

## Fechamento

Usada **só** por `/review-pr`, e só quando `linear_issue_id` não for
`null`. Segue a mesma governança de aprovação humana já aplicada à
escrita no GitHub pelo próprio `/review-pr` — nenhuma escrita no Linear
acontece antes da aprovação explícita do usuário.

1. **Redigir (Etapa 5 de `/review-pr`, em memória, nada em disco/API
   ainda)**: componha o comentário final de fechamento do card
   (resumo do que a spec entregou, link da PR) e inclua-o no resumo
   mostrado ao usuário junto com o resto do fechamento da feature.
2. **Publicar (Etapa 7 de `/review-pr`, só após a confirmação da Etapa
   6)**: publique o comentário redigido no passo 1. **Nunca** execute
   este passo se o evento do review for `REQUEST_CHANGES` — mesma
   regra que já impede o fechamento de estado/roadmap/docs-sync de
   serem escritos nesse caso.

Esta seção **não** altera o workflow state (a "coluna"/status) do card
no Linear — nomes de estado variam por workspace e não há mapeamento
confiável e genérico entre eles. Fica como extensão futura documentada,
não implementada agora.

## O que esta skill nunca faz

- Não cria cards novos no Linear.
- Não muda assignee, prioridade, projeto ou team de um card.
- Não decide sozinha que uma feature está bloqueada, cancelada ou
  falhou — só espelha no Linear uma decisão já tomada no pipeline (ver
  "Estados de exceção" em `.pipeline/feature-state.schema.md`: essas
  transições são sempre a pedido explícito do usuário).
- Não muda o workflow state (status/coluna) do card.
