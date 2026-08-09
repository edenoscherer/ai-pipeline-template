# Formato do feature-state.json

Cada feature ativa mantém um arquivo de estado em:

```
<ESTADO_DIR>/<slug>.json
```

onde `<slug>` é o nome curto da feature (ex.: `user-auth`, sem o
prefixo numérico). Todo comando do pipeline lê e atualiza este arquivo
— nunca redigite ou re-derive o diretório da feature a partir de
conversa/memória quando o estado já existir.

## Schema

```json
{
  "feature_dir": "specs/017-user-auth",
  "branch": "017-user-auth",
  "short_name": "user-auth",
  "current_phase": "tasks",
  "phases_completed": ["specify", "plan"],
  "phases_pending": ["tasks", "implement", "review"],
  "clarifications_asked": 2,
  "last_updated": "2026-08-08T00:00:00Z",
  "quality_gates_status": {
    "typecheck": null,
    "test": null,
    "lint": null
  }
}
```

### Campos

| Campo | Tipo | Descrição |
|---|---|---|
| `feature_dir` | string | Caminho completo do diretório da feature em `SPECS_DIR` |
| `branch` | string | Nome da branch git associada |
| `short_name` | string | Slug usado para nomear o próprio arquivo de estado |
| `current_phase` | string | Uma de: `specify`, `plan`, `tasks`, `implement`, `review`, `done` |
| `phases_completed` | string[] | Fases já concluídas, na ordem em que terminaram |
| `phases_pending` | string[] | Fases restantes, na ordem esperada |
| `clarifications_asked` | number | Total de perguntas de clarificação já feitas nesta feature (soma entre specify/specify-tech) |
| `last_updated` | string (ISO 8601) | Timestamp da última atualização do estado |
| `quality_gates_status` | object | Resultado do último gate rodado por `/implement`: `null` (não rodado), `"pass"` ou `"fail"` |

## Regras de uso

1. `/specify` (ou `/specify-tech`) cria este arquivo na primeira
   execução para a feature.
2. Todo comando subsequente (`plan`, `tasks`, `implement`, `review-pr`)
   lê `feature_dir` e `branch` daqui antes de qualquer outra ação —
   nunca pergunta ao usuário "qual é o diretório da feature" se o
   arquivo já existir e a referência for inequívoca.
3. Ao concluir uma fase com sucesso, o comando responsável:
   - move a fase de `phases_pending` para `phases_completed`
   - atualiza `current_phase` para a próxima fase pendente
   - atualiza `last_updated`
4. Se houver mais de uma feature com estado incompleto no mesmo
   projeto, `/pipeline-status` lista todas e pede ao usuário para
   indicar qual retomar — nenhum comando deve escolher sozinho qual
   feature continuar se houver ambiguidade.
5. Quando `current_phase` chega a `"done"`, o arquivo pode ser mantido
   (histórico) ou arquivado — isso é decisão do projeto, não do pipeline.
