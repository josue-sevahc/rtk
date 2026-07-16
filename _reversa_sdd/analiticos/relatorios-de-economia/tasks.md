# Relatórios de Economia, Tarefas de Implementação

## Pré-requisitos

- [ ] API de leitura do tracking por dia, semana e mês. 🟢
- [ ] Executor seguro de subprocessos para a integração externa. 🟢

## Tarefas

- [ ] T-01 — Implementar parser tolerante a `date/week/month` e `period` do ccusage.
  - Origem no legado: `src/analytics/ccusage.rs`
  - Critério de pronto: respostas antigas e atuais são desserializadas.
  - Confiança: 🟢
- [ ] T-02 — Mesclar os períodos e calcular CPT/economia ponderados.
  - Origem no legado: `src/analytics/cc_economics.rs`
  - Critério de pronto: dados parciais não causam divisão inválida.
  - Confiança: 🟢
- [ ] T-03 — Emitir texto, JSON e CSV.
  - Origem no legado: `src/analytics/cc_economics.rs`
  - Critério de pronto: formatos respeitam períodos selecionados.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01 — Testar conversão sábado→segunda e merge parcial.
- [ ] TT-02 — Testar zero tokens e ausência de dados de ccusage.
- [ ] TT-03 — Testar parse dos aliases de período e defaults de cache.

## Ordem Sugerida

1. Adaptador ccusage; 2. merge/cálculo; 3. renderização e exports.

## Lacunas Pendentes (🔴)

Nenhuma.
