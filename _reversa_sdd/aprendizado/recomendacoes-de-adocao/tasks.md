# Recomendações de Adoção, Tarefas de Implementação

> Sequência para reconstruir a publicação de recomendações de `rtk learn`.

## Pré-requisitos

- [ ] 🟢 Disponibilizar `CorrectionPair` e `CorrectionRule` produzidos pela detecção.
- [ ] 🟢 Definir o caminho local de regras e garantir permissão de escrita quando a opção for usada.

## Tarefas

- [ ] T-01, Aplicar filtros de confiança e recorrência na ordem observada. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`.
  - Critério de pronto: confiança filtra pares antes da deduplicação e recorrência filtra regras depois dela.
- [ ] T-02, Serializar o relatório JSON com metadados e regras. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`.
  - Critério de pronto: a saída contém campos de sessão, total e regras no formato previsto.
- [ ] T-03, Implementar relatório textual para conjuntos vazios e não vazios. Confiança: 🟢
  - Origem no legado: `src/learn/report.rs`.
  - Critério de pronto: texto mostra cabeçalho, recorrência e fragmento de erro quando presentes.
- [ ] T-04, Implementar a escrita Markdown agrupada e ordenada por comando-base dentro de secoes gerenciadas. Confiança: 🟢
  - Origem no legado: `src/learn/report.rs`.
  - Critério de pronto: somente o trecho entre marcadores gerenciados e atualizado; conteudo manual externo e preservado; sem marcadores, a operacao recusa escrita, gera arquivo separado ou exige `--force` com backup.
- [ ] T-05, Ligar a escrita somente ao modo textual, `--write-rules` e existência de regras. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`.
  - Critério de pronto: JSON e listas vazias não produzem arquivo.

## Tarefas de Teste

- [ ] TT-01, Testar JSON com regras e limiares. Confiança: 🟢 `src/learn/mod.rs`
- [ ] TT-02, Testar relatório textual vazio, recorrente e com primeira linha de erro. Confiança: 🟢 `src/learn/report.rs`
- [ ] TT-03, Testar criação de diretório, agrupamento alfabético e atualização idempotente da seção gerenciada. Confiança: 🟢 `src/learn/report.rs` e decisao do usuario em 2026-07-16.
- [ ] TT-04, Testar preservacao de conteudo manual, recusa sem marcadores e `--force` com backup. Confiança: 🟢 Decisao validada pelo usuario em 2026-07-16.

## Ordem Sugerida

1. T-01 estabelece o conjunto publicável.
2. T-02 e T-03 entregam os canais de saída.
3. T-04 e T-05 fecham a persistência opcional e suas condições.

## Lacunas Pendentes (🔴)

- 🟢 Usar secoes gerenciadas e preservar conteudo manual; sem marcadores, recusar, gerar arquivo separado ou exigir `--force` com backup. Decisao do usuario em 2026-07-16.
