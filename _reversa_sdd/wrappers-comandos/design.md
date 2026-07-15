# Wrappers de Comandos, Design Tecnico

> Implementacao distribuida em `src/cmds/` e na infraestrutura compartilhada de `src/core/runner.rs` e `src/core/stream.rs`. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Interface

🟢 Os wrappers expõem, em geral, funcoes `run` que recebem subcomando ou argumentos e `verbose`, retornando `Result<i32>`. A entrada e transformada em `std::process::Command`; a saida e texto filtrado ou streams heredados, e a resposta semantica e o exit code original da ferramenta.

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `cmds::<ecossistema>::run` | varia por ferramenta, normalmente `(args/subcommand, verbose) -> Result<i32>` | exit code | 🟢 Monta argumentos especificos e seleciona o runner/filtro. |
| `runner::run` | `(Command, tool_name, args_display, RunMode, RunOptions) -> Result<i32>` | exit code | 🟢 Esqueleto comum de medicao, execucao, tracking e propagacao de status. |
| `runner::run_filtered` | `(Command, tool_name, args_display, filter_fn, RunOptions) -> Result<i32>` | exit code | 🟢 Encapsula captura e filtro pos-execucao. |
| `runner::run_streamed` | `(Command, tool_name, args_display, StreamFilter, RunOptions) -> Result<i32>` | exit code | 🟢 Encapsula filtro incremental de stdout. |
| `runner::run_passthrough` | `(tool, &[OsString], verbose) -> Result<i32>` | exit code | 🟢 Executa com TTY herdado e tracking neutro. |
| `StreamFilter` | `feed_line`, `flush`, `on_exit` | texto opcional | 🟢 Contrato para filtros linha a linha e resumo final. |
| `BlockHandler` | deteccao de bloco, continuidade e resumo | comportamento de bloco | 🟢 Base para parsers de erros em blocos. |

## Fluxo Principal

1. 🟢 `main` despacha o subcomando para o modulo de ecossistema registrado em `src/cmds/mod.rs`.
2. 🟢 O wrapper interpreta argumentos, preserva flags sensiveis e monta `Command` por `resolved_command` ou por logica especifica da ferramenta.
3. 🟢 O wrapper escolhe `RunMode::Filtered`, `FilteredWithExit`, `Streamed` ou `Passthrough`; fluxos que combinam multiplos arquivos/processos podem executar captura manual.
4. 🟢 `runner::run` inicia `TimedExecution` e deriva `cmd_label` da ferramenta e argumentos exibiveis.
5. 🟢 Em modo filtrado, `run_captured_filter` coleta output e aplica funcao pura, podendo conhecer o exit code no modo `FilteredWithExit`.
6. 🟢 Em modo streaming, `stream::run_streaming` envia cada linha a `StreamFilter::feed_line`, chama `flush()` no final e permite `on_exit()` produzir resumo baseado no status e raw.
7. 🟢 Em modo passthrough, `stream::run_streaming` herda TTY para stdin e usa `FilterMode::Passthrough`, sem buffer de output para economia de tokens.
8. 🟢 Quando configurado, o runner anexa hint de `tee` para recuperar output removido; o tracking registra raw versus texto filtrado, ou duracao neutra no passthrough.
9. 🟢 O runner devolve `Ok(exit_code)` ao wrapper, e a entrada CLI torna esse codigo o status do processo.

## Estrategias de Filtragem

| Estrategia | Mecanismo | Aplicacao observada | Comportamento de degradacao |
|------------|-----------|---------------------|-----------------------------|
| 🟢 Captura estruturada | `run_filtered`/captura manual e parser JSON, XML, NDJSON, binlog ou TRX | AWS, GH/GLab, Go, .NET, PHPStan, RuboCop, Vitest, Playwright | 🟢 Falha de parse usa raw, passthrough ou filtro textual explicitamente previsto. |
| 🟢 Filtro buffered | funcao sobre output completo | tabelas, listas, diffs, status e saidas curtas | 🟢 Guard de tokens substitui compacto maior por raw. |
| 🟢 Streaming | `StreamFilter` por linha | Gradle, logs longos, filtros progressivos | 🟢 `flush` e `on_exit` encerram estado e preservam resumo final. |
| 🟢 Maquina de estados/blocos | `BlockHandler` ou implementacao de `StreamFilter` | Maven, Cargo, Pytest, PHPUnit, Rake | 🟢 Texto inesperado cai em estado conservador ou preserva dados relevantes. |
| 🟢 Passthrough | TTY herdado | formato solicitado pelo usuario ou subcomando nao suportado | 🟢 Nao transforma output e devolve status do filho. |

## Fluxos Alternativos

- 🟢 **Flag de formato do usuario:** Git/forjas e outros wrappers detectam pedidos como `--json`, `--output`, `--web` ou formato especial e evitam injecao dupla por passthrough.
- 🟢 **Formato estruturado elegivel:** o wrapper pode inserir JSON, NDJSON, binlog ou TRX antes da execucao para obter dados com menor ambiguidade de parsing.
- 🟢 **Erro de ferramenta:** o runner preserva o exit code e filtros podem usar esse codigo para decidir quais detalhes/hints emitir.
- 🟢 **Tee configurado:** output filtrado/streamed recebe caminho de recuperacao quando ha perda ou falha; o hint e emitido apos o resultado do filtro.
- 🟢 **Erro de parsing:** wrappers como GH/GLab retornam stdout bruto quando JSON e inesperadamente invalido, em vez de sintetizar uma resposta sem suporte nos dados.
- 🟢 **Argumentos livres:** wrappers de Git, Cargo e similares preservam argumentos nativos, incluindo casos como flags apos `--` quando a ferramenta exige restauracao posicional.

## Dependencias

- 🟢 `core::runner`: concentra `RunMode`, tracking, tee, guard e propagacao de exit code.
- 🟢 `core::stream`: executa processo, encaminha streams e abstrai `FilterMode`, `StreamFilter`, `BlockHandler` e `LineHandler`.
- 🟢 `core::tracking`: mede duracao, estima tokens e persiste economia; passthrough usa registro de tokens zero.
- 🟢 `core::tee` e `core::guard`: fornecem recuperacao e o invariante `never_worse`.
- 🟢 `core::utils::resolved_command`: resolve binario externo antes de construir a execucao.
- 🟢 `parser`: atende filtros que dependem de parsing/tokenizacao reutilizavel, especialmente resultados de teste e JSON embutido.
- 🟢 `discover`: participa de roteamento e classificacao compartilhada em casos cross-ecosystem.
- 🟢 CLIs externas: definem formatos, flags e codigos de saida que cada wrapper precisa preservar.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| Wrappers conhecem a semantica da ferramenta; runner conhece o ciclo comum de processo, tracking e status. | `src/cmds/README.md`, `src/core/runner.rs:159-296` | 🟢 |
| Captura e streaming sao modos distintos porque parsers estruturados precisam do blob completo, enquanto logs longos precisam de baixa latencia/memoria. | `src/cmds/README.md`, `src/core/stream.rs` | 🟢 |
| Passthrough herda o TTY e registra tokens zero para nao fingir economia onde nao houve captura. | `src/core/runner.rs:256-268`, `src/core/tracking.rs:1392-1402` | 🟢 |
| Formato explicitamente solicitado pelo usuario prevalece sobre a compactacao do RTK. | `src/cmds/git/README.md` | 🟢 |
| O uso de JSON e preferido quando suportado, mas filtros textuais continuam necessarios para ferramentas, subcomandos e locales sem schema confiavel. | `src/cmds/README.md`, `_reversa_sdd/code-analysis.md` | 🟢 |
| O agrupamento por ecossistema reduz acoplamento conceitual e facilita regras de execucao compartilhadas por toolchain. | `src/cmds/mod.rs`, `src/cmds/README.md` | 🟡 |

## Estado Interno

- 🟢 `RunOptions` define tee, separacao de stdout, encerramento antecipado em falha, newline e heranca de stdin por invocacao.
- 🟢 `BlockStreamFilter` mantem `in_block`, `current_block` e `blocks_emitted` para detectar e emitir blocos completos.
- 🟢 Implementacoes de `StreamFilter` e `BlockHandler` mantem estado especifico da ferramenta, como fase Maven, contagem de erros Cargo ou falhas de teste.
- 🟢 `StreamResult` retorna `exit_code`, raw combinado, raw por stream e texto filtrado ao runner.
- 🟡 Regras de cada wrapper sao predominantemente stateless entre invocacoes; o estado persistente observado pertence ao tracking em `core`.

## Observabilidade

- 🟢 `TimedExecution` mede cada execucao filtrada, streamed ou passthrough.
- 🟢 Tracking de filtro recebe comando original, comando RTK, raw e texto mostrado; passthrough registra duracao sem tokens.
- 🟢 `verbose > 0` faz wrappers de passthrough relatarem ferramenta e argumentos em stderr.
- 🟢 Hints de tee tornam output omitido recuperavel por comando/caminho informado ao usuario.
- 🟢 Falha de spawn e erro de I/O recebem contexto com o nome da ferramenta na infraestrutura de runner/stream.

## Riscos e Lacunas

- 🔴 A extracao nao executou o catalogo de ferramentas externas, portanto nao confirma compatibilidade com versoes, locales e formatos instalados no ambiente alvo.
- 🔴 Filtros extensos de Git, Cargo, AWS, .NET, Maven e search ainda merecem specs de caso de uso dedicadas para equivalencia detalhada.
- 🟡 Forcar formatos estruturados pode encontrar flags incompatíveis em versoes antigas; os fallbacks observados reduzem, mas nao eliminam, esse risco.
- 🟡 O uso de texto por state machine depende de marcadores da ferramenta e pode degradar sob mudanca de locale ou formato.
