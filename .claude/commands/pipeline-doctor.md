# Comando: Pipeline Doctor

Verifica a **saúde da configuração do pipeline em si** — não o
progresso de nenhuma feature (isso é `/pipeline-status`). Comando
**somente leitura**: não modifica nada, não corrige nada, não avança
nenhuma fase. Útil logo após copiar este pacote para um projeto novo,
ou periodicamente para detectar configuração que apodreceu com o
tempo (arquivo apagado, roadmap movido, etc.).

## Configuração

Leia `.pipeline/config.md` para obter todos os parâmetros a verificar
(`IDIOMA_ARTEFATOS`, `SPECS_DIR`, `ESTADO_DIR`, `ARQUIVO_REGRAS`,
`ARQUIVO_ARQUITETURA`, `ARQUIVO_PRODUTO`, `ARQUIVO_QUALITY_GATES`,
`ARQUIVO_ROADMAP`, `ARQUIVO_DECISIONS_LOG`, `DOCS_FEATURES_DIR`).

## Execução

Rode as verificações abaixo, na ordem, sem parar em falhas — o
objetivo é um relatório completo, não interromper no primeiro
problema.

1. **`.pipeline/config.md` existe.**
2. **Campos obrigatórios preenchidos**: `IDIOMA_ARTEFATOS`,
   `SPECS_DIR`, `ESTADO_DIR` têm valor (não vazios, não placeholder).
3. **`ARQUIVO_REGRAS` existe** no caminho configurado. Se não existir:
   `⚠`, não `✗` — o pipeline já degrada graciosamente na ausência
   deste arquivo (ver `.pipeline/config.md`), então isso não é erro
   fatal, mas vale sinalizar.
4. **`ARQUIVO_ARQUITETURA` existe** no caminho configurado. Mesma
   regra do item 3: ausência é `⚠`, não `✗`.
5. **`ARQUIVO_PRODUTO`** — se configurado (não vazio em `config.md`),
   confirme que o caminho existe. Ausente mas configurado: `⚠`, mesma
   regra dos itens 3 e 4 (degradação graciosa, `/specify` e o
   `software-dev-panel` já lidam com a ausência). Se deixado vazio de
   propósito, reporte `— (não usado neste projeto)`, não como falha.
6. **`ARQUIVO_QUALITY_GATES` existe e está preenchido**: abra o
   arquivo (`.pipeline/quality-gates.md` por padrão) e confirme que
   nenhuma célula da tabela ainda contém o placeholder `<preencher>`
   sem edição. Arquivo ausente ou com `<preencher>` remanescente conta
   como não passou.
7. **`ARQUIVO_ROADMAP`** — se configurado (não vazio em `config.md`),
   confirme que o caminho existe. Se `config.md` deixou vazio de
   propósito, reporte `— (não usado neste projeto)`, não como falha.
8. **`ARQUIVO_DECISIONS_LOG`** — mesma regra do item 7.
9. **`DOCS_FEATURES_DIR`** — mesma regra do item 7 (confirma que o
   diretório existe, não que tem conteúdo).
10. **Comandos esperados existem em `.claude/commands/`** (ou
    `.cursor/commands/`, conforme a ferramenta em uso): `specify`,
    `specify-tech`, `plan`, `tasks`, `implement`, `review-pr`,
    `pipeline-status`, `docs-sync`, `pipeline-doctor` — 9 no total.
    Reporte quantos dos 9 foram encontrados.
11. **Skills esperadas existem em `.claude/skills/`**:
    `clarification-protocol`, `software-dev-panel` — 2 no total.
12. **Para cada domínio em `DOCS_FEATURES_DIR`** (se configurado e o
    diretório existir): para cada arquivo `.md` que não seja
    `_template.md`, confirme que contém uma seção "Specs
    Relacionadas" com uma tabela. Se `DOCS_FEATURES_DIR` estiver vazio
    ou não houver nenhum doc de domínio ainda, pule esta verificação
    sem contar como falha (nada para checar ainda).
13. **`.pipeline/version` existe e é legível** — reporte a versão
    detectada. Ausência conta como não passou (ver
    `.pipeline/version` e a seção de versionamento do `README.md`).

## Saída

```
## Pipeline Doctor

✓ .pipeline/config.md
✓ Campos obrigatórios preenchidos (IDIOMA_ARTEFATOS, SPECS_DIR, ESTADO_DIR)
⚠ ARQUIVO_REGRAS (memory/constitution.md) — não encontrado
⚠ ARQUIVO_ARQUITETURA (docs/arquitetura.md) — não encontrado
⚠ ARQUIVO_PRODUTO (docs/product.md) — não encontrado
✗ quality-gates.md — placeholders <preencher> não editados
✓ roadmap.md
✓ decisions-log.md
✓ docs/features/
⚠ docs/features/onboarding.md — sem tabela "Specs Relacionadas"
✓ Comandos (9/9)
✓ Skills (2/2)
✓ Versão do pipeline: 1.0.0

Pipeline: 62% saudável (8/13 verificações passaram)
```

`✓` = passou. `⚠` = degradação graciosa, não fatal, mas sinalizada
(conta como não-passou no percentual). `✗` = falhou. `—` = item não
configurado no projeto (não conta nem a favor nem contra).

Não sugira correções automáticas nem ofereça editar nenhum arquivo —
se o usuário quiser corrigir algo apontado, ele pede explicitamente em
seguida. Este comando só diagnostica.
