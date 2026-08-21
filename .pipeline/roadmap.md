# Roadmap — <Nome do Projeto>

Lista mestre de specs planejadas, em ordem de implementação. Diferente
de `<ESTADO_DIR>/*.json` (estado detalhado por feature, usado pelos
comandos), este arquivo é a **visão agregada e legível** do projeto —
serve pra você, um colaborador, ou qualquer pessoa abrir no GitHub sem
rodar comando nenhum e entender o que existe, o que está em andamento
e o que falta.

**Como este arquivo é mantido**: os comandos do pipeline atualizam o
status automaticamente ao final de cada fase (ver `.pipeline/config.md`
→ `ARQUIVO_ROADMAP`). Você pode editar manualmente para adicionar
specs novas planejadas (ainda sem `/specify` rodado) ou reordenar
prioridade — mas não edite o status de uma spec já iniciada; deixe o
pipeline atualizar.

---

## Fase 1 — <nome da fase, ex.: Fundação>

| # | Spec | Status | Última atualização |
|---|------|--------|---------------------|
| 001 | <slug-da-feature> | 🔲 Pendente | — |
| 002 | <slug-da-feature> | 🔲 Pendente | — |

## Fase 2 — <nome da fase, ex.: Features Core>

| # | Spec | Status | Última atualização |
|---|------|--------|---------------------|
| 003 | <slug-da-feature> | 🔲 Pendente | — |

<!--
Adicione quantas fases fizerem sentido para o seu projeto, ou remova
o agrupamento por fase inteiramente e use uma tabela única se preferir
um roadmap mais simples, sem fases.
-->

---

## Legenda

- 🔲 **Pendente** — ainda sem `/specify` (ou `/specify-tech`) rodado
- 🟡 **Em andamento** — alguma fase (`specify`/`plan`/`tasks`/`implement`/`review`) já rodou, mas não concluiu o ciclo
- ✅ **Concluído** — PR revisada, com fechamento incluído na própria PR
  (`review-pr` Etapa 5/7) e mergeada
- 🚧 **Bloqueado** — `current_phase: blocked` no estado da feature (ver `status_detail` no JSON ou `/pipeline-status` para o motivo)
- ⛔ **Cancelado** — `current_phase: cancelled`
- ❌ **Falhou** — `current_phase: failed`

Os três últimos são mapeamento direto do `current_phase` de exceção em
`<ESTADO_DIR>/<slug>.json` (ver "Estados de exceção" em
`.pipeline/feature-state.schema.md`) — nunca setados manualmente aqui
sem o estado correspondente refletir o mesmo valor.

## Notas

- A numeração (`#`) segue a mesma sequência usada em `<SPECS_DIR>/<NNN>-<slug>/`
  — não pule números nem reordene depois de atribuídos.
- Se uma spec for cancelada, bloqueada ou falhar, não delete a linha —
  marque com o símbolo correspondente da legenda e mantenha o
  histórico visível.
