# Wrappers de Comandos, Tarefas de Implementacao

> Sequencia para reconstruir os wrappers por ecossistema e sua infraestrutura de filtragem, com rastreabilidade ao legado.

## Pre-requisitos

- [ ] 🟢 Disponibilizar criacao segura de processos, streams, conversao de `ExitStatus`, resolucao de binarios e propagacao de erros com contexto. Origem: `src/core/runner.rs`, `src/core/stream.rs`, `src/core/utils.rs`.
- [ ] 🟢 Disponibilizar tracking, guard `never_worse` e tee de recuperacao para runners compartilhados. Origem: `src/core/tracking.rs`, `src/core/guard.rs`, `src/core/tee.rs`.
- [ ] 🟢 Definir contratos de parsing reutilizaveis para JSON/XML/NDJSON e saidas de teste quando aplicavel. Origem: `src/parser/`, `src/cmds/*`.
- [ ] 🔴 Disponibilizar fixtures representativas ou ambientes controlados para cada CLI externa suportada, incluindo variacoes de versao e locale. Origem: `_reversa_sdd/code-analysis.md:315`.

## Tarefas

- [ ] T-01, Criar o registro de ecossistemas e a fronteira de responsabilidade de wrappers.
  - Origem no legado: `src/cmds/mod.rs`, `src/cmds/README.md`.
  - Criterio de pronto: system, git, rust, js, python, go, dotnet, cloud, php, ruby e jvm possuem ponto de extensao proprio; logica compartilhada sem execucao externa fica fora dos wrappers.
  - Confianca: 🟢

- [ ] T-02, Implementar o runner comum para os modos filtered, filtered-with-exit, streamed e passthrough.
  - Origem no legado: `src/core/runner.rs:159-296`.
  - Criterio de pronto: cada modo inicia medicao, executa o filho, devolve seu exit code e registra tracking adequado; wrappers so precisam fornecer `Command`, rotulo, modo e opcoes.
  - Confianca: 🟢

- [ ] T-03, Implementar infraestrutura de streaming e filtros de linha/bloco.
  - Origem no legado: `src/core/stream.rs`.
  - Criterio de pronto: `StreamFilter` recebe linhas, finaliza com `flush` e pode resumir em `on_exit`; `BlockHandler` preserva inicio/continuidade de blocos e produz resumo baseado em raw e exit code.
  - Confianca: 🟢

- [ ] T-04, Integrar guard de tokens, tee e tracking ao caminho filtrado.
  - Origem no legado: `src/core/runner.rs`, `src/core/guard.rs`, `src/core/tee.rs`, `src/core/tracking.rs`.
  - Criterio de pronto: o texto rastreado coincide com o texto exibido; hints tornam omissoes recuperaveis; forma filtrada+hints maior que raw e substituida pela referencia bruta.
  - Confianca: 🟢

- [ ] T-05, Implementar passthrough transparente e metricas neutras.
  - Origem no legado: `src/core/runner.rs:256-268`, `src/core/tracking.rs:1392-1402`.
  - Criterio de pronto: stdin/stdout/stderr sao herdados, argumentos `OsString` permanecem intactos, exit code e devolvido e tracking usa tokens zero.
  - Confianca: 🟢

- [ ] T-06, Implementar filtros estruturados por familia de ferramenta.
  - Origem no legado: `src/cmds/cloud/aws_cmd.rs`, `src/cmds/go/`, `src/cmds/dotnet/`, `src/cmds/git/gh_cmd.rs`, `src/cmds/git/glab_cmd.rs`, `src/cmds/js/`, `src/cmds/php/`, `src/cmds/ruby/`.
  - Criterio de pronto: cada wrapper injeta ou consome JSON/XML/NDJSON/binlog/TRX somente em subcomandos elegiveis e devolve raw/passthrough quando o parser nao consegue confirmar a estrutura.
  - Confianca: 🟢

- [ ] T-07, Implementar filtros textuais stateful para saidas sem schema confiavel.
  - Origem no legado: `src/cmds/jvm/mvn_cmd.rs`, `src/cmds/jvm/gradlew_cmd.rs`, `src/cmds/rust/cargo_cmd.rs`, `src/cmds/python/pytest_cmd.rs`, `src/cmds/php/phpunit_cmd.rs`, `src/cmds/ruby/rake_cmd.rs`.
  - Criterio de pronto: blocos de erro, falhas de teste, warnings relevantes e resumo final sobrevivem a compactacao; ruido e boilerplate podem ser removidos sem esconder estado de falha.
  - Confianca: 🟢

- [ ] T-08, Implementar regras de preservacao de argumentos e formato solicitado pelo usuario.
  - Origem no legado: `src/cmds/git/README.md`, `src/cmds/rust/README.md`, `src/cmds/system/search.rs`.
  - Criterio de pronto: flags de output/JSON e subcomandos desconhecidos escolhem passthrough; argumentos apos `--`, opcoes globais Git e flags de busca que alteram formato mantem sua semantica.
  - Confianca: 🟢

- [ ] T-09, Implementar os wrappers de maior impacto antes dos complementares.
  - Origem no legado: `src/cmds/git/git.rs`, `src/cmds/rust/cargo_cmd.rs`, `src/cmds/cloud/aws_cmd.rs`, `src/cmds/dotnet/dotnet_cmd.rs`, `src/cmds/jvm/mvn_cmd.rs`, `src/cmds/system/search.rs`.
  - Criterio de pronto: Git, Cargo, AWS, .NET, Maven e search possuem cobertura de filtros, fallback e fixtures antes de expandir o restante do catalogo.
  - Confianca: 🟡

- [ ] T-10, Implementar roteamento cross-ecosystem e utilitarios de sistema.
  - Origem no legado: `src/cmds/js/lint_cmd.rs`, `src/cmds/system/format_cmd.rs`, `src/cmds/system/summary.rs`, `src/cmds/system/pipe_cmd.rs`.
  - Criterio de pronto: deteccao de projeto ou comando encaminha para o filtro correto sem duplicar infraestrutura de processo; filtros genericos preservam fallback para formato desconhecido.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar infraestrutura de streams e status de processo.
  - Origem no legado: `src/core/stream.rs:581-800`.
  - Criterio de pronto: exit codes `0`, nao zero e sinal sao convertidos corretamente; modos passthrough, buffered, capture-only e streaming preservam streams e raw esperado.
  - Confianca: 🟢

- [ ] TT-02, Testar filtros de linha e bloco.
  - Origem no legado: `src/core/stream.rs:887-1120`.
  - Criterio de pronto: blocos completos sao emitidos com continuidade por indentacao, prefixos ruidosos sao descartados somente quando configurados e resumo recebe exit code correto.
  - Confianca: 🟢

- [ ] TT-03, Testar o invariante de tracking, tee e `never_worse`.
  - Origem no legado: `src/core/runner.rs`, `src/core/tracking.rs`, ADR 001.
  - Criterio de pronto: raw e texto visivel coincidam com o registro; passthrough usa tokens zero; output compacto maior que raw nao e exibido; hint recupera trecho omitido quando habilitado.
  - Confianca: 🟢

- [ ] TT-04, Criar suites de fixtures para pelo menos um caso de cada estrategia.
  - Origem no legado: `tests/fixtures/`, `src/cmds/*`.
  - Criterio de pronto: incluir um parser estruturado, um filtro buffered, um state machine/streaming, um passthrough por flag e uma falha de parse que preserva raw.
  - Confianca: 🟢

- [ ] TT-05, Testar Git e forjas contra flags que exigem passthrough e exit code de falha.
  - Origem no legado: `src/cmds/git/git.rs:2073+`, `src/cmds/git/README.md`.
  - Criterio de pronto: opcoes globais e argumentos livres sao preservados; flags `--json`/`--output` nao sofrem dupla formatacao; falha de repositorio retorna codigo nao zero.
  - Confianca: 🟢

- [ ] TT-06, Testar Cargo, Maven/Gradle e Pytest com fixtures de sucesso, warning e falha.
  - Origem no legado: `src/cmds/rust/cargo_cmd.rs:1446+`, `src/cmds/jvm/mvn_cmd.rs:971+`, `src/cmds/python/pytest_cmd.rs:320+`.
  - Criterio de pronto: erros e falhas de teste permanecem visiveis; blocos sao delimitados corretamente; output de sucesso e compactado sem inventar falhas.
  - Confianca: 🟢

- [ ] TT-07, Testar degradacao de JSON, XML, NDJSON, binlog e TRX.
  - Origem no legado: `src/cmds/cloud/`, `src/cmds/go/`, `src/cmds/dotnet/`, `src/cmds/js/`, `src/cmds/php/`, `src/cmds/ruby/`.
  - Criterio de pronto: payload malformado ou formato inesperado nunca produz resumo inventado e seleciona raw, passthrough ou fallback textual documentado.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 Os wrappers apenas produzem eventos de tracking; schema, retencao e migracoes pertencem ao modulo `core`.

## Ordem Sugerida

1. Implementar T-01 a T-05 para tornar o runner confiavel antes de multiplicar filtros de ferramenta.
2. Implementar T-08 junto de T-06/T-07: preservar flags e formatos e condicao para qualquer otimizacao ser segura.
3. Priorizar T-09 por alcance transversal e quantidade de regras observadas; depois completar T-06, T-07 e T-10 por ecossistema.
4. Executar TT-01 a TT-04 como guardrails de infraestrutura em toda mudanca; usar TT-05 a TT-07 como suites de regressao por ferramenta.

## Lacunas Pendentes (🔴)

- 🔴 Validar compatibilidade de cada wrapper contra as versoes reais das ferramentas, seus locales e formatos de output.
- 🔴 Definir um conjunto minimo de fixtures de contrato por ferramenta antes de declarar equivalencia comportamental.
- 🔴 Avaliar se todos os filtros de alto impacto devem ter specs aninhadas adicionais, especialmente Git, Cargo, AWS, .NET, Maven e search.
