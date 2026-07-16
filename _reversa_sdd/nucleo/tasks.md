# Nucleo, Tarefas de Implementacao

> Sequencia para reconstruir `src/core/` preservando seus contratos observados. Cada tarefa identifica a origem e a confianca.

## Pre-requisitos

- [ ] 🟢 Disponibilizar dependencias Rust equivalentes a `anyhow`, `serde`, `toml`, `regex`, `rusqlite`, `chrono` e acesso a diretorios do usuario. Origem: `Cargo.toml`, `src/core/*.rs`.
- [ ] 🟢 Definir configuracao carregavel para tracking, filtros, tee, telemetria, hooks e limites. Origem: `src/core/config.rs`, `src/core/constants.rs`.
- [ ] 🟡 Definir a fronteira com o modulo de confianca de hooks antes de conectar filtros TOML externos. Origem: `src/core/toml_filter.rs:191`.

## Tarefas

- [ ] T-01, Estruturar o modulo compartilhado e expor apenas contratos agnosticos de dominio.
  - Origem no legado: `src/core/README.md:5`, `src/core/mod.rs`.
  - Criterio de pronto: consumidores de comandos acessam os contratos compartilhados sem que o modulo dependa de filtros de comando especificos.
  - Confianca: 🟢

- [ ] T-02, Implementar configuracao, constantes e utilitarios de base, incluindo resolucao de executavel e estimativa de tokens.
  - Origem no legado: `src/core/config.rs`, `src/core/constants.rs`, `src/core/utils.rs`.
  - Criterio de pronto: configuracoes ausentes usam defaults observados e o executor consegue construir processos externos por nome.
  - Confianca: 🟢

- [ ] T-03, Implementar `stream::run_streaming` com modos de stdin, passthrough, captura e streaming de stdout/stderr.
  - Origem no legado: `src/core/stream.rs:247-380`.
  - Criterio de pronto: o processo filho recebe o stdin esperado, stdout e stderr sao tratados conforme o modo e o resultado retorna codigo e buffers consistentes.
  - Confianca: 🟢

- [ ] T-04, Limitar a captura bruta por stream a 10 MiB, preservar codigos de saida e aguardar filhos em limpeza RAII.
  - Origem no legado: `src/core/stream.rs:230-245`, `src/core/stream.rs:284-289`.
  - Criterio de pronto: output acima do limite e sinalizado sem crescimento ilimitado, termino por sinal Unix vira `128 + sinal` e nenhum filho permanece sem espera ao sair do escopo.
  - Confianca: 🟢

- [ ] T-05, Implementar o guard `never_worse` e a emissao que registra exatamente o texto mostrado ao usuario.
  - Origem no legado: `src/core/guard.rs:6`, `src/core/runner.rs:9-19`, `src/core/runner.rs:150-156`.
  - Criterio de pronto: uma compactacao com mais tokens estimados que o raw e substituida pelo raw; a saida emitida e a usada no tracking sao identicas.
  - Confianca: 🟢

- [ ] T-06, Implementar `runner::run` e os quatro `RunMode`, incluindo reemissao raw quando `skip_filter_on_failure` estiver ativo.
  - Origem no legado: `src/core/runner.rs:84-156`, `src/core/runner.rs:159-285`.
  - Criterio de pronto: filtros capturados, filtros com exit code, streaming e passthrough retornam o codigo do filho; falha com early exit nao passa por filtro.
  - Confianca: 🟢

- [ ] T-07, Implementar tee de recuperacao com politica por configuracao, `RTK_TEE=0`, tamanho minimo, slug seguro e rotacao.
  - Origem no legado: `src/core/tee.rs:76-168`, `src/core/tee.rs:184-235`.
  - Criterio de pronto: output elegivel cria arquivo de recuperacao e hint; output nao elegivel nao cria arquivo; a quantidade maxima configurada e respeitada.
  - Confianca: 🟢

- [ ] T-08, Implementar registry de filtros TOML, compilacao validada e gate de confianca para fontes externas.
  - Origem no legado: `src/core/toml_filter.rs:185-245`.
  - Criterio de pronto: filtros confiaveis validos sao adicionados, built-ins continuam disponiveis e falhas de parse ou confianca apenas geram aviso/ignorancia.
  - Confianca: 🟢

- [ ] T-09, Implementar a DSL TOML e a classificacao `Lossiness` na ordem de transformacao observada.
  - Origem no legado: `src/core/toml_filter.rs:503-650`.
  - Criterio de pronto: `strip_ansi`, replace, `match_output`/`unless`, filtros de linha, truncamento, head/tail, `max_lines` e `on_empty` executam em ordem, com perda recuperavel classificada como `Tail` quando aplicavel.
  - Confianca: 🟢

- [ ] T-10, Implementar tracking SQLite com migracoes idempotentes, registros de execucao, retencao e filtros por projeto sem `LIKE`.
  - Origem no legado: `src/core/tracking.rs:1-64`, `src/core/tracking.rs`.
  - Criterio de pronto: uma execucao armazena tokens, duracao e caminho canonico; consultas por projeto usam igualdade ou `GLOB` com separador de caminho; falha auxiliar de tracking nao derruba a execucao do comando.
  - Confianca: 🟢

- [ ] T-11, Implementar telemetria opt-in, com opt-out, marcador diario e envio assincrono nao bloqueante.
  - Origem no legado: `src/core/telemetry.rs:22`, `src/core/telemetry_cmd.rs`.
  - Criterio de pronto: sem endpoint, consentimento, configuracao ou com opt-out o ping nao ocorre; quando elegivel, o CLI nao espera pelo envio.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar os quatro modos do runner e a preservacao de exit code, incluindo termino por sinal quando a plataforma suportar.
  - Origem no legado: `src/core/runner.rs:159-285`, `src/core/stream.rs:230-270`.
  - Criterio de pronto: suites comprovam sucesso, falha, passthrough e streaming sem alterar o codigo retornado.
  - Confianca: 🟢

- [ ] TT-02, Testar `never_worse` com saida menor, maior, empatada e vazia.
  - Origem no legado: `src/core/guard.rs:21-54`.
  - Criterio de pronto: somente o caso filtrado estritamente pior retorna raw.
  - Confianca: 🟢

- [ ] TT-03, Testar gate de confianca e falhas de parse de filtros TOML.
  - Origem no legado: `src/core/toml_filter.rs:203-243`.
  - Criterio de pronto: filtros nao confiaveis, alterados e invalidos nao entram no registry; filtros validos continuam aplicaveis.
  - Confianca: 🟢

- [ ] TT-04, Testar todos os estagios da DSL e os tres estados de `Lossiness`.
  - Origem no legado: `src/core/toml_filter.rs:515-650`.
  - Criterio de pronto: testes cobrem `unless`, corte de linhas, `on_empty`, cauda recuperavel e perda integral.
  - Confianca: 🟢

- [ ] TT-05, Testar tee por modo, tamanho, desativacao por ambiente e rotacao de arquivos.
  - Origem no legado: `src/core/tee.rs:76-189`.
  - Criterio de pronto: somente outputs elegiveis criam arquivos e o hint referencia o arquivo produzido.
  - Confianca: 🟢

- [ ] TT-06, Testar tracking com migracoes repetidas, retencao e caminho de projeto contendo `_` ou `%`.
  - Origem no legado: `src/core/tracking.rs:51-61`.
  - Criterio de pronto: consultas de projeto nao tratam esses caracteres como curingas e o banco continua utilizavel apos migracoes repetidas.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] TM-01, Importar opcionalmente o banco legado `tracking.db` para o banco canonico `history.db`.
  - Origem no legado: `src/core/tracking.rs`, `src/core/constants.rs`.
  - Criterio de pronto: a deteccao e automatica, mas a importacao exige confirmacao ou comando explicito; cria backup, e idempotente, deduplica registros e nunca modifica ou apaga o banco legado.
  - Confianca: 🟢 Decisao validada pelo usuario em 2026-07-16.

## Ordem Sugerida

1. Implementar T-01 e T-02 para estabelecer interfaces, configuracao e utilitarios compartilhados.
2. Implementar T-03 e T-04 antes do runner, pois os modos de execucao dependem da captura e do contrato de exit code.
3. Implementar T-05 e T-06 juntos para manter a invariante entre saida exibida e tracking.
4. Implementar T-07 a T-09 depois que a captura estiver pronta; tee e DSL dependem de raw e de filtros.
5. Implementar T-10 e T-11 como integracoes auxiliares nao bloqueantes, e executar TT-01 a TT-06 antes da adocao pelos wrappers.

## Lacunas Pendentes (🔴)

- 🔴 Validar em Windows a semantica de `PATHEXT`, sinais e captura de processos, nao exercitada pela analise estatica.
- 🟢 `history.db` e canonico; `tracking.db` pode ser importado somente com confirmacao ou comando explicito, com backup, deduplicacao e preservacao integral do legado. Decisao do usuario em 2026-07-16.
- 🔴 Executar teste de concorrencia real com multiplas instancias e WAL antes de afirmar equivalencia de persistencia.
