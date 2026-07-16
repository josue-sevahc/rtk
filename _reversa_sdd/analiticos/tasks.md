# Analíticos, Tarefas de Implementação

## Pré-requisitos

- [ ] Disponibilizar a API read-only equivalente a `core::tracking::Tracker`. 🟢
- [ ] Registrar `gain`, `cc-economics` e `session` no dispatcher CLI. 🟢

## Tarefas

- [ ] T-01 — Implementar a camada analítica read-only e o dashboard de ganhos.
  - Origem no legado: `src/analytics/README.md`, `src/analytics/gain.rs`
  - Critério de pronto: nenhum caminho analítico grava métricas; resumo e períodos são exibidos.
  - Confiança: 🟢
- [ ] T-02 — Implementar exports e visões alternativas de `gain`.
  - Origem no legado: `src/analytics/gain.rs`
  - Critério de pronto: JSON, CSV, períodos e escopo de projeto obedecem às flags.
  - Confiança: 🟢
- [ ] T-03 — Implementar relatórios de economia e adoção das subunits.
  - Origem no legado: `src/analytics/cc_economics.rs`, `src/analytics/session_cmd.rs`
  - Critério de pronto: ausências de dados são tratadas sem panic.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01 — Cobrir tracking vazio, com registros e reset recusado.
- [ ] TT-02 — Cobrir fallback/indisponibilidade de `ccusage`.
- [ ] TT-03 — Cobrir comandos explícitos, reescritos e encadeados em sessões.

## Ordem Sugerida

1. Implementar consultas e formato de `gain`.
2. Implementar adaptador opcional `ccusage` e relatório econômico.
3. Implementar descoberta de sessões e testes de classificação.

## Lacunas Pendentes (🔴)

Nenhuma lacuna bloqueante identificada no código analisado.
