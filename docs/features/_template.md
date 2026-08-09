# <Nome do Domínio/Módulo>

> Documentação viva — reflete o comportamento **atual** deste módulo,
> não o histórico. Para histórico, ver "Specs Relacionadas" abaixo.
> Mantida por `/docs-sync`, chamado automaticamente pelo `/review-pr`.

## O que este módulo faz

<1-2 parágrafos, linguagem simples — o que existe hoje, não o que
existia antes de cada correção>

## Comportamentos-chave e regras de negócio

- <regra 1>
- <regra 2>

## Contrato de API (referência)

<link ou resumo dos endpoints ativos deste domínio — não repita o
contrato inteiro aqui; aponte para o `contracts/` da spec mais recente
que o definiu, ou para a documentação central de API do projeto
(Swagger, etc.), se houver uma>

## Limitações conhecidas

- <limitação 1>

<!-- Ao corrigir um bug via /docs-sync, remova daqui o que foi
resolvido — não deixe limitação "fantasma" listada depois de corrigida -->

---

## Specs Relacionadas

Histórico de specs que tocaram este módulo, **mais recente primeiro**.
Use esta tabela para detectar recorrência: se um bug parecido com um
já corrigido aqui reaparecer, comece lendo a spec anterior (`spec.md`
+ `research.md`, seção de causa raiz) antes de investigar do zero — a
causa raiz documentada lá pode não ter sido de fato eliminada.

| # | Spec | Tipo | Resumo | Data |
|---|------|------|--------|------|
| <NNN> | [<NNN>-<slug>](../../specs/<NNN>-<slug>/) | ✨ Feature / 🐛 Bug fix | <resumo em 1 linha> | AAAA-MM-DD |

<!-- Exemplo (apagar ao usar):
| 021 | [021-fix-rounding](../../specs/021-fix-rounding/) | 🐛 Bug fix | Corrigido arredondamento em cálculo de margem para categorias com desconto | 2026-07-14 |
| 017 | [017-margin-calc](../../specs/017-margin-calc/) | ✨ Feature | Introduzido cálculo de margem híbrido por categoria | 2026-05-02 |
| 012 | [012-fix-price-precision](../../specs/012-fix-price-precision/) | 🐛 Bug fix | Corrigido erro de precisão decimal em preços com 3+ casas | 2026-03-19 |
-->

<!-- Se dois bug fixes na mesma área aparecerem aqui (como 021 e 012
no exemplo acima — ambos sobre precisão/arredondamento de preço), é
um sinal forte de causa raiz não resolvida na primeira tentativa. O
/specify-tech verifica isso automaticamente antes de investigar um
bug novo neste domínio. -->
