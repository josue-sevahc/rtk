# Execucao Filtrada, Design Tecnico

> Implementacao observada em `src/core/runner.rs`, `src/core/stream.rs` e `src/core/tracking.rs`. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Interface

🟢 O runner recebe um `Command`, nome/argumentos para rotulo, `RunMode` e `RunOptions`; ele retorna `Result<i32>`. A camada de stream recebe o processo mutavel, modo de stdin e `FilterMode`, e retorna `StreamResult` com status, raw combinado, raw por stream e texto filtrado.

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `runner::run` | `(Command, &str, &str, RunMode, RunOptions) -> Result<i32>` | exit code | 🟢 Seleciona o caminho capturado, streaming ou passthrough. |
| `run_captured_filter` | `(Command, tool_name, cmd_label, filtro, opcoes, timer) -> Result<i32>` | exit code | 🟢 Aplica filtro pos-captura e emite texto guardado. |
| `stream::run_streaming` | `(&mut Command, StdinMode, FilterMode) -> Result<StreamResult>` | streams e status | 🟢 Executa filho e coleta/encaminha streams. |
| `status_to_exit_code` | `(ExitStatus) -> i32` | exit code | 🟢 Usa codigo normal ou `128 + signal` em Unix; fallback `1`. |
| `exec_capture` | `(&mut Command) -> Result<CaptureResult>` | stdout, stderr e status | 🟢 Captura manual com stdin nulo. |
| `exec_capture_stdin` | `(&mut Command) -> Result<CaptureResult>` | stdout, stderr e status | 🟢 Captura manual preservando stdin do chamador. |

## Fluxo Principal: Runner Capturado

1. 🟢 `runner::run()` inicia `TimedExecution` e cria `cmd_label` com ferramenta e argumentos.
2. 🟢 Para `Filtered`, a funcao do wrapper e adaptada para ignorar o exit code; para `FilteredWithExit`, o status convertido e passado ao filtro.
3. 🟢 `run_captured_filter()` escolhe `StdinMode::Inherit` somente com `inherit_stdin`; caso contrario escolhe `StdinMode::Null`.
4. 🟢 `stream::run_streaming()` e chamado com `FilterMode::CaptureOnly`, resultando em stdout e stderr capturados sem transformar stdout nessa etapa.
5. 🟢 Quando `skip_filter_on_failure` esta ativo e `exit_code != 0`, stdout e reemitido em stdout, stderr em stderr, tracking usa raw contra raw e o fluxo retorna sem chamar o filtro.
6. 🟢 Caso contrario, o filtro recebe raw combinado ou apenas `raw_stdout`, dependendo de `filter_stdout_only`.
7. 🟢 Com `tee_label`, `print_with_hint()` compoe filtro e hint e chama `emit_guarded`; sem tee, `never_worse` compara direto o filtrado com a referencia de tracking e imprime respeitando `no_trailing_newline`.
8. 🟢 `timer.track()` registra comando, rotulo, referencia raw e texto efetivamente exibido; o runner devolve o exit code original.

## Fluxo Principal: Stream

1. 🟢 Para `RunMode::Streamed`, `runner::run()` chama `run_streaming()` com stdin nulo e `FilterMode::Streaming(filter)`.
2. 🟢 A funcao inicia o processo com stdout/stderr canalizados e cria uma thread leitora por stream, conectadas ao consumidor por canal MPSC.
3. 🟢 O consumidor atualiza `raw_stdout` e `raw_stderr` ate `RAW_CAP` de 10 MiB por stream; ao ultrapassar o limite, marca o stream como capped e emite warning.
4. 🟢 Cada linha e entregue a `filter.feed_line`; texto retornado e acumulado em `filtered` e escrito no mesmo descritor de origem, stdout ou stderr.
5. 🟢 Ao fechar o canal, o runner chama `filter.flush()`, escreve o restante no ultimo descritor filtrado e junta as threads leitoras.
6. 🟢 Depois de aguardar o filho, `on_exit(exit_code, raw)` pode adicionar texto de resumo; o resultado final e devolvido ao runner.
7. 🟢 O runner pode imprimir hint de tee para raw, rastreia raw versus `filtered` e devolve o status do filho.

## Fluxo Principal: Passthrough

1. 🟢 Para `RunMode::Passthrough`, a camada de stream trata o modo antes de criar pipes.
2. 🟢 Stdin e herdado apenas para `StdinMode::Inherit`; stdout e stderr sempre usam `Stdio::inherit()`.
3. 🟢 O processo e aguardado diretamente e `StreamResult` retorna strings vazias para raw e filtered, com status convertido.
4. 🟢 O runner chama `track_passthrough()`, que registra duracao com tokens zero, e devolve o exit code.

## Fluxos Alternativos

- 🟢 **Filtro buffered interno:** `FilterMode::Buffered` coleta stdout e executa a funcao dentro de `catch_unwind`; panic gera warning e usa `raw_stdout`.
- 🟢 **Capture-only:** coleta stdout e stderr sem imprimir na camada de stream; a camada runner decide como filtrar, proteger e emitir.
- 🟢 **Stdin filtrado:** `StdinMode::Filter` cria thread que le stdin, aplica `StdinFilter` por linha e fecha a entrada do filho apos `flush`.
- 🟢 **Reader thread de stderr em modo nao streaming:** uma thread coleta stderr enquanto a thread principal le stdout, evitando bloqueio entre pipes.
- 🟢 **Thread de stderr em panic:** join retorna string vazia e emite warning, preservando a continuidade do processo.
- 🟢 **Broken pipe ao escrever resultado:** escrita em stdout/stderr ignora especificamente `BrokenPipe`; outros erros de escrita retornam falha.
- 🟢 **Termino por sinal:** em Unix, `ExitStatusExt::signal()` vira `128 + sinal`; sem codigo e sem sinal observavel, o retorno e `1`.

## Dependencias

- 🟢 `std::process::{Command, Stdio, ExitStatus}`: spawn, configuracao de descritores e status do processo filho.
- 🟢 `std::thread`, `mpsc`, `BufReader` e `BufWriter`: transporte concorrente de stdout, stderr e stdin filtrado.
- 🟢 `core::guard::never_worse`: limita a resposta emitida pela estimativa de tokens da referencia.
- 🟢 `core::tee`: cria hint recuperavel para output removido ou falha de ferramenta.
- 🟢 `core::tracking::TimedExecution`: mede duracao e persiste contagens de tokens ou metricas neutras.
- 🟢 Implementacoes de `StreamFilter`/`StdinFilter`: encapsulam parsing por ferramenta sem alterar o mecanismo de processo.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| A execucao de processo e centralizada no stream; politicas de filtragem, tee e tracking ficam no runner. | `src/core/runner.rs`, `src/core/stream.rs` | 🟢 |
| `CaptureOnly` separa coleta de I/O da transformacao final para que o runner consiga escolher raw combinado, stdout-only ou comportamento por exit code. | `src/core/runner.rs:80-155`, `src/core/stream.rs` | 🟢 |
| Streaming usa threads e canal MPSC para ler stdout e stderr sem risco de um pipe bloquear o outro. | `src/core/stream.rs:300-420` | 🟢 |
| A captura e limitada por stream, mas o processo continua; o limite protege memoria e pode reduzir a qualidade do filtro/tracking em outputs extremos. | `src/core/stream.rs:244`, `src/core/stream.rs:330-390` | 🟢 |
| Filtros em panic preservam raw stdout para preferir continuidade e fidelidade a encerramento do wrapper. | `src/core/stream.rs:445-460` | 🟢 |
| Passthrough nao tenta medir economia porque o output nao e capturado, evitando estatistica artificial. | `src/core/runner.rs:190-202`, `src/core/tracking.rs:1392-1402` | 🟢 |

## Estado Interno

- 🟢 `RunOptions` e imutavel por invocacao e controla tee, fonte de filtro, comportamento em falha, newline e stdin.
- 🟢 `TimedExecution` armazena o instante inicial usado para calcular `elapsed_ms` no momento do tracking.
- 🟢 `ChildGuard` envolve o filho e chama `wait()` no drop para reduzir risco de processo nao aguardado.
- 🟢 `raw_stdout`, `raw_stderr` e `filtered` acumulam output da invocacao; `capped_out` e `capped_err` impedem buffer maior que 10 MiB.
- 🟢 No streaming, `saved_filter` retem o filtro depois do loop para que `on_exit` possa observar status e raw completo/capturado.
- 🟢 `filter_fd_is_stderr` escolhe o descritor do `flush` e de qualquer texto post-execucao.

## Observabilidade

- 🟢 Tracking filtrado registra `cmd_label`, rotulo `rtk <cmd_label>`, raw de referencia, output emitido e duracao.
- 🟢 Tracking passthrough registra o mesmo comando e duracao, com input/output tokens zero.
- 🟢 Warnings em stderr informam captura excedendo 10 MiB, panic de filtro e panic da thread de stderr.
- 🟢 Hints de tee sao impressos quando habilitados pelo runner e quando `tee_and_hint` encontra conteudo recuperavel.
- 🟢 Contexto `Failed to run <tool>` e aplicado quando `run_streaming` retorna erro para o runner.

## Riscos e Lacunas

- 🟢 Ao atingir 10 MiB por stream, a captura deixa de acumular os bytes seguintes e emite warning; por isso filtro, tee e tracking podem observar somente o prefixo capturado, enquanto o processo continua. `src/core/stream.rs:245`, `src/core/stream.rs:376`
- 🔴 A equivalencia de sinal e descritores em Windows nao foi exercitada; a conversao `128+sinal` e especificamente Unix.
- 🟡 Em streaming, `flush` e `on_exit` usam o ultimo descritor filtrado, o que pode ser surpreendente para filtros que alternam output entre stdout e stderr.
- 🟡 O tratamento de panic cobre `FilterMode::Buffered`; filtros invocados no caminho `CaptureOnly` pelo wrapper seguem a politica de panic do chamador.
