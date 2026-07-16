# Formatacao Estruturada, Tarefas de Implementacao

> Sequencia para reconstruir a formatacao de `src/parser/formatter.rs`.

## Pre-requisitos

- [ ] 🟢 Disponibilizar os tipos `TestResult`, `TestFailure`, `DependencyState` e `Dependency`.
- [ ] 🟢 Disponibilizar o limite `CAP_INVENTORY` para listagens compactas.

## Tarefas

- [ ] T-01, Implementar `FormatMode` e o mapeamento de verbosidade. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: 0 mapeia para Compact, 1 para Verbose e valores maiores para Ultra.
- [ ] T-02, Definir `TokenFormatter` com os tres formatos e o dispatcher comum. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: `format` delega exatamente ao metodo do modo selecionado.
- [ ] T-03, Formatar `TestResult` preservando ignorados, mensagens multiline, limite de cinco falhas e duracao opcional. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: resumo compacto e detalhado seguem os limites e campos observados.
- [ ] T-04, Implementar a representacao Ultra de testes com contadores curtos e duracao padrao zero quando ausente. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: a string usa os tres marcadores e uma duracao numerica.
- [ ] T-05, Formatar `DependencyState`, distinguindo inventario simples, estado atualizado e atualizacoes pendentes. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: inventario nunca e rotulado falsamente como atualizado.
- [ ] T-06, Aplicar limites de listagem e incluir wanted version no detalhado quando divergir da latest. Confianca: 🟢
  - Origem no legado: `src/parser/formatter.rs`.
  - Criterio de pronto: saidas alem dos limites exibem contagem residual e detalhes wanted aparecem no verbose.
- [ ] T-07, Separar o contrato estruturado da apresentacao humana localizavel. Confianca: 🟢
  - Origem: decisao do usuario em 2026-07-16.
  - Criterio de pronto: codigos, chaves, enums e schemas permanecem em ingles; `en-US` e obrigatorio e `pt-BR` e o primeiro locale adicional para textos humanos.

## Tarefas de Teste

- [ ] TT-01, Testar verbosidade 0, 1 e maior que 1. Confianca: 🟢 `src/parser/formatter.rs`
- [ ] TT-02, Testar resumo de testes com aprovados, falhas, ignorados, duracao e mais de cinco falhas. Confianca: 🟢 `src/parser/formatter.rs`
- [ ] TT-03, Testar mensagem de falha multiline com expected/received e call log no modo compacto. Confianca: 🟢 `src/parser/formatter.rs`
- [ ] TT-04, Testar inventario de dependencias sem `latest_version` e ausencia real de desatualizados. Confianca: 🟢 `src/parser/formatter.rs`
- [ ] TT-05, Testar limite de dependencias, wanted version e formato Ultra. Confianca: 🟢 `src/parser/formatter.rs`
- [ ] TT-06, Validar legibilidade e economia de tokens com saidas reais de ferramentas. Confianca: 🔴
- [ ] TT-07, Testar estabilidade dos campos estruturados e snapshots de apresentacao em `en-US` e `pt-BR`. Confianca: 🟢 Decisao do usuario em 2026-07-16.

## Ordem Sugerida

1. T-01 e T-02 estabelecem a interface e o dispatcher.
2. T-03 e T-04 completam a formatacao de testes.
3. T-05 e T-06 completam dependencias e os limites de inventario.

## Lacunas Pendentes (🔴)

- 🟢 Preservar os valores atuais no perfil versionado `legacy-v1`; qualquer perfil recalibrado exige dataset, benchmark, falsos positivos, economia e protecao `never_worse`. Decisao do usuario em 2026-07-16.
