# Relatórios de Economia

## Visão Geral

Esta subunit implementa `rtk cc-economics`: relaciona custos/tokens do Claude Code obtidos por `ccusage` com tokens economizados no tracking RTK. 🟢

## Responsabilidades

- Mesclar métricas diária, semanal ou mensal de duas fontes pelo período. 🟢
- Calcular economia primária com custo por token de entrada ponderado. 🟢
- Expor texto, JSON e CSV e degradar quando `ccusage` estiver indisponível. 🟢

## Regras de Negócio

- Unidades ponderadas: entrada + 5×saída + 1,25×cache write + 0,1×cache read. 🟢
- A economia primária é `tokens economizados × CPT de entrada ponderado`. 🟢
- A semana RTK registrada em sábado é deslocada dois dias para alinhar à segunda ISO de `ccusage`. 🟢
- Métricas active e blended são referenciais; a primeira superestima e a segunda subestima. 🟢

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Buscar e unir métricas por período. | Must | Períodos presentes em qualquer fonte aparecem. |
| RF-02 | Calcular totais e economia ponderada apenas com dados suficientes. | Must | Campos indisponíveis são representados como ausentes. |
| RF-03 | Exportar JSON/CSV ou tabela de terminal. | Must | A opção de formato preserva os campos documentados. |

## Critérios de Aceitação

```gherkin
Cenário: dados combinados
  Dado que existem métricas mensais de RTK e ccusage
  Quando o usuário executa `rtk cc-economics`
  Então vê gasto, tokens poupados e economia ponderada

Cenário: ccusage indisponível
  Dado que ccusage não pode ser executado
  Quando o relatório é gerado
  Então o processo não falha apenas por essa ausência
```

## Rastreabilidade de Código

| Arquivo | Função / Classe | Cobertura |
|---|---|---|
| `src/analytics/cc_economics.rs` | `run`, merges, totais e exports | 🟢 |
| `src/analytics/ccusage.rs` | `fetch`, parser e fallback | 🟢 |
