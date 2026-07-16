# Adoção por Sessão

## Visão Geral

Esta subunit implementa `rtk session`, um relatório de cobertura de comandos RTK nas sessões recentes do Claude Code. 🟢

## Responsabilidades

- Descobrir sessões dos últimos 30 dias e ignorar arquivos de subagents. 🟢
- Processar as dez sessões de nível superior mais recentes. 🟢
- Contar partes de comandos encadeados e identificar cobertura explícita ou por hook. 🟢
- Mostrar comandos, cobertura, barra de progresso, saída estimada e média global. 🟢

## Regras de Negócio

- Um comando é coberto se começa com `rtk ` ou se `classify_command` o classifica como suportado. 🟢
- Cada cadeia é dividida antes da contagem, para não considerar uma cadeia inteira como único comando. 🟢
- Adoção é zero quando a sessão não possui comandos. 🟢
- Falhas ao extrair uma sessão individual são ignoradas. 🟢

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Descobrir e ordenar sessões válidas. | Must | Só os 10 JSONL principais mais recentes são considerados. |
| RF-02 | Classificar cobertura por parte de comando. | Must | Comandos explícitos e reescrevíveis entram no numerador. |
| RF-03 | Exibir tabela e adoção média. | Should | Saída contém dados por sessão e total agregado. |

## Critérios de Aceitação

```gherkin
Cenário: sessão com comandos mistos
  Dado que uma sessão contém comandos RTK, suportados e não suportados
  Quando `rtk session` é executado
  Então a adoção usa somente os comandos explícitos ou suportados

Cenário: ausência de sessões
  Dado que não há sessões Claude recentes
  Quando `rtk session` é executado
  Então recebe mensagem de orientação sem erro
```

## Rastreabilidade de Código

| Arquivo | Função / Classe | Cobertura |
|---|---|---|
| `src/analytics/session_cmd.rs` | `run`, `count_rtk_commands`, `SessionSummary` | 🟢 |
| `src/discover/provider.rs` | descoberta e extração de comandos | 🟢 |
| `src/discover/registry.rs` | split e classificação | 🟢 |
