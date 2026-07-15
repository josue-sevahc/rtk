# Nucleo, Design Tecnico

> Design reconstruido de `src/core/`. Afirmacoes marcadas com 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 indicam lacunas.

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `runner::run` | `(Command, &str, &str, RunMode, RunOptions)` | `Result<i32>` | Orquestra execucao, emissao e tracking. 🟢 `src/core/runner.rs:159` |
| `stream::run_streaming` | `(&mut Command, StdinMode, FilterMode)` | `Result<StreamResult>` | Executa com captura, streaming ou passthrough. 🟢 `src/core/stream.rs:247` |
| `guard::never_worse` | `(&str, &str)` | `&str` | Escolhe a saida que nao aumenta a estimativa de tokens. 🟢 `src/core/guard.rs:6` |
| `TomlFilterRegistry::load` | `()` | `TomlFilterRegistry` | Monta registry a partir de fontes confiaveis e built-ins. 🟢 `src/core/toml_filter.rs:185` |
| `apply_filter_with_info` | `(&CompiledFilter, &str)` | `(String, Lossiness)` | Executa a DSL e classifica perda de informacao. 🟢 `src/core/toml_filter.rs:515` |
| `Tracker::new` | `()` | `Result<Tracker>` | Abre e prepara o banco SQLite de tracking. 🟢 `src/core/tracking.rs` |
| `Tracker::record` | `(&str, &str, usize, usize, u64)` | `Result<()>` | Persiste uma execucao e sua economia estimada. 🟢 `src/core/tracking.rs` |
| `tee::tee_and_hint` | `(&str, &str, i32)` | `Option<String>` | Persiste raw elegivel e devolve hint de recuperacao. 🟢 `src/core/tee.rs:184` |
| `telemetry::maybe_ping` | `()` | `()` | Avalia elegibilidade e agenda ping nao bloqueante. 🟢 `src/core/telemetry.rs:22` |

### Tipos Principais

- `RunMode` modela quatro caminhos: filtro capturado, filtro que recebe exit code, filtro streaming e passthrough. 🟢 `src/core/runner.rs:84`
- `RunOptions` carrega label de tee, politica de filtro em erro, escopo de stdout, nova linha e heranca de stdin. 🟢 `src/core/runner.rs:33`
- `StreamResult` devolve exit code, raw combinado, raw por descritor e saida filtrada. 🟢 `src/core/stream.rs:247`
- `Lossiness` diferencia perda nenhuma, cauda recuperavel e perda integral. 🟢 `src/core/toml_filter.rs:503`
- `TeeConfig` e `TeeMode` controlam habilitacao, politica e limites do arquivo de recuperacao. 🟢 `src/core/tee.rs:238`

## Fluxo Principal

1. Um wrapper cria `Command`, escolhe `RunMode` e fornece `RunOptions` a `runner::run`. 🟢 `src/core/runner.rs:159`
2. O runner inicia `TimedExecution` e forma o rótulo do comando para tracking. 🟢 `src/core/runner.rs:166`
3. Nos modos filtrados, `run_captured_filter` chama `stream::run_streaming` com `CaptureOnly`, recebendo stdout, stderr, raw combinado e exit code. 🟢 `src/core/runner.rs:91`, `src/core/runner.rs:107`
4. Se a politica for pular filtro em erro e o processo falhar, o runner reemite os fluxos brutos, registra raw contra raw e devolve o exit code. 🟢 `src/core/runner.rs:114`
5. Caso contrario, o runner seleciona stdout ou raw combinado, aplica a funcao de filtro e opcionalmente cria uma recuperacao tee. 🟢 `src/core/runner.rs:125`, `src/core/runner.rs:138`
6. `emit_guarded` combina o hint com a saida filtrada, aplica `never_worse`, imprime e devolve exatamente a saida emitida. 🟢 `src/core/runner.rs:9`
7. O timer registra a entrada bruta e a saida efetivamente mostrada; o exit code do processo e preservado. 🟢 `src/core/runner.rs:150`

### Captura e Streaming

1. Em passthrough, stdio e herdado, o processo e aguardado e o resultado retorna raw vazio. 🟢 `src/core/stream.rs:252`
2. Em captura ou streaming, o processo recebe pipes; stdout e stderr sao lidos em threads separadas. 🟢 `src/core/stream.rs:273`, `src/core/stream.rs:324`, `src/core/stream.rs:340`
3. Cada stream e acumulado ate `RAW_CAP` de 10 MiB; excedentes emitem aviso e deixam de ser capturados. 🟢 `src/core/stream.rs:244`, `src/core/stream.rs:369`
4. `ChildGuard` aguarda o filho em `Drop`; o codigo de encerramento por sinal Unix e convertido para `128 + sinal`. 🟢 `src/core/stream.rs:230`, `src/core/stream.rs:284`

### Filtro TOML

1. O registry coleta caminhos submetidos ao gate de confianca e so adiciona conteudo `Trusted` ou `EnvOverride`. 🟢 `src/core/toml_filter.rs:191`, `src/core/toml_filter.rs:203`
2. Os filtros built-in sao compilados mesmo quando fontes externas falham; erros de parse e de definicoes invalidas apenas escrevem warnings. 🟢 `src/core/toml_filter.rs:195`, `src/core/toml_filter.rs:225`
3. O filtro aplica transformacoes na sequencia fixa de ANSI, replace, match_output/unless, linhas, truncamento, head/tail, max_lines e `on_empty`. 🟢 `src/core/toml_filter.rs:515`
4. A classificacao `Lossiness` informa se a saida omitida pode ser recuperada por cauda ou exige o raw integral. 🟢 `src/core/toml_filter.rs:631`

## Fluxos Alternativos

- **Falha sem filtro:** `skip_filter_on_failure` faz reemissao raw e retorno imediato com o exit code original. 🟢 `src/core/runner.rs:114`
- **Passthrough:** o core nao captura nem filtra a saida, mas ainda mede e registra uma execucao passthrough. 🟢 `src/core/runner.rs:207`
- **Tee inelegivel:** variavel `RTK_TEE=0`, configuracao desabilitada, modo incompativel, sucesso em modo `failures` ou raw pequeno retornam `None`. 🟢 `src/core/tee.rs:76`, `src/core/tee.rs:151`
- **Filtro TOML invalido ou nao confiavel:** e ignorado; o registry continua com os filtros disponiveis. 🟢 `src/core/toml_filter.rs:203`, `src/core/toml_filter.rs:225`
- **Telemetria inelegivel:** a rotina retorna sem enviar quando falta algum pre-requisito de endpoint, configuracao, consentimento ou intervalo diario. 🟢 `src/core/telemetry.rs:22`

## Dependencias

- `rusqlite`: persistencia de tracking e agregacoes locais. 🟢 `src/core/tracking.rs:34`
- `serde` e `toml`: desserializacao e compilacao do formato declarativo de filtros. 🟢 `src/core/toml_filter.rs:225`
- `hooks::trust`: decisao de confianca de filtros externos. 🟢 `src/core/toml_filter.rs:191`
- `config`, `constants` e `utils`: configuracao, caminhos, estimativa de tokens, resolucao de executaveis e normalizacoes compartilhadas. 🟢 `src/core/runner.rs:260`, `src/core/tracking.rs:64`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| A saida filtrada e limitada pela estimativa de tokens da saida original. | `src/core/guard.rs:6`, `src/core/runner.rs:17` | 🟢 |
| A saida realmente exibida, e nao apenas a filtrada, alimenta o tracking. | `src/core/runner.rs:150` | 🟢 |
| Captura de stdout e stderr usa threads independentes e limite por stream. | `src/core/stream.rs:324`, `src/core/stream.rs:340` | 🟢 |
| Filtros externos sao trust-gated e falhas de parse nao sao fatais. | `src/core/toml_filter.rs:203`, `src/core/toml_filter.rs:225` | 🟢 |
| Tracking por projeto escolhe `GLOB` em vez de `LIKE` para nao interpretar `_` e `%` como curingas. | `src/core/tracking.rs:51` | 🟢 |
| Telemetria e realizada fora do caminho bloqueante do CLI. | `src/core/telemetry.rs:22` | 🟢 |

## Estado Interno

- O banco SQLite mantem historico de comandos, tokens, duracao e caminho de projeto; as consultas oferecem agregacoes temporais e por projeto. 🟢 `src/core/tracking.rs:1`
- O registry de filtros mantem uma lista ordenada de `CompiledFilter`; fontes locais confiaveis sao adicionadas antes dos filtros built-in. 🟢 `src/core/toml_filter.rs:185`
- Arquivos tee sao nomeados com epoch e slug do comando e sofrem rotacao por quantidade maxima de arquivos. 🟢 `src/core/tee.rs:105`
- O estado de consentimento, salt/identificador e marcador de ultimo ping sao usados pela telemetria. 🟢 `src/core/telemetry.rs:22`

## Observabilidade

- Tracking registra estimativa de economia, percentual, duracao e contexto de projeto em SQLite. 🟢 `src/core/tracking.rs:1`
- A captura acima do limite emite warnings para stdout ou stderr, conforme o stream que excedeu `RAW_CAP`. 🟢 `src/core/stream.rs:369`
- Erros de parse de filtros TOML e definicoes invalidas geram warnings com a origem do arquivo. 🟢 `src/core/toml_filter.rs:195`, `src/core/toml_filter.rs:240`
- Telemetria e um canal de observabilidade de produto separado da execucao sincrona do CLI. 🟢 `src/core/telemetry.rs:22`

## Riscos e Lacunas

- 🔴 A analise estatica nao validou os caminhos especificos de Windows para resolucao de executaveis e sinais de processos.
- 🔴 Nao houve exercicio de concorrencia real de SQLite/WAL com multiplas instancias do CLI.
- 🟡 Embora o README defina o Nucleo como agnostico de hooks, `toml_filter.rs` e `telemetry.rs` usam recursos de `hooks`; a fronteira arquitetural requer validacao humana. `src/core/README.md:5`, `src/core/toml_filter.rs:191`
