# Dicionario de Dados — Archaeologist

Projeto: `rtk`

## Modulo `main`

### `AgentTarget`

🟢 **CONFIRMADO** — Enum publico com `ValueEnum`, usado para selecionar o agente alvo em `rtk init`.

| Campo/variante | Tipo | Obrigatorio | Descricao |
|---|---|---:|---|
| `Claude` | enum variant | sim | Claude Code, alvo padrao. |
| `Cursor` | enum variant | sim | Cursor Agent. |
| `Windsurf` | enum variant | sim | Windsurf/Cascade. |
| `Cline` | enum variant | sim | Cline/Roo Code. |
| `Kilocode` | enum variant | sim | Kilo Code. |
| `Antigravity` | enum variant | sim | Google Antigravity. |
| `Pi` | enum variant | sim | Pi coding agent. |
| `Hermes` | enum variant | sim | Hermes CLI. |
| `Droid` | enum variant | sim | Factory Droid CLI. |

### `Cli`

🟢 **CONFIRMADO** — Struct raiz parseada pelo `clap`.

| Campo | Tipo | Obrigatorio | Default | Descricao |
|---|---|---:|---|---|
| `command` | `Commands` | sim | n/a | Subcomando escolhido. |
| `verbose` | `u8` | nao | `0` | Nivel `-v`, `-vv`, `-vvv`, reconhecido antes do subcomando. |
| `ultra_compact` | `bool` | nao | `false` | Flag global `--ultra-compact`. |
| `skip_env` | `bool` | nao | `false` | Flag global `--skip-env`, repassada a comandos JS/TS/env-sensitive. |

### `Commands`

🟢 **CONFIRMADO** — Enum principal de subcomandos. Variantes com `trailing_var_arg` aceitam argumentos nativos da ferramenta externa.

| Variante | Campos principais | Destino de dispatch |
|---|---|---|
| `Ls` | `args: Vec<String>` | `cmds::system::ls` |
| `Tree` | `args: Vec<String>` | `cmds::system::tree` |
| `Read` | `files`, `level`, `max_lines`, `tail_lines`, `line_numbers` | `cmds::system::read` |
| `Smart` | `file`, `model`, `force_download` | `cmds::system::local_llm` |
| `Git` | globais Git + `GitCommands` | `cmds::git::git` |
| `Gh` | `subcommand`, `args` | `cmds::git::gh_cmd` |
| `Glab` | `repo`, `group`, `subcommand`, `args` | `cmds::git::glab_cmd` |
| `Aws` | `subcommand`, `args` | `cmds::cloud::aws_cmd` |
| `Psql` | `args` | `cmds::cloud::psql_cmd` |
| `Pnpm` | `filter`, `PnpmCommands` | `cmds::js::pnpm_cmd`/`tsc_cmd` |
| `Err` | `command` | `cmds::rust::runner::run_err` |
| `Test` | `command` | `cmds::rust::runner::run_test` |
| `Json` | `file`, `depth`, `keys_only` | `cmds::system::json_cmd` |
| `Deps` | `path` | `cmds::system::deps` |
| `Env` | `filter` | `cmds::system::env_cmd` |
| `Find` | `args` | `cmds::system::find_cmd` |
| `Diff` | `file1`, `file2` | `cmds::git::diff_cmd` |
| `Log` | `file` | `cmds::system::log_cmd` |
| `Dotnet` | `DotnetCommands` | `cmds::dotnet::dotnet_cmd` |
| `Docker` | `DockerCommands` | `cmds::cloud::container` |
| `Kubectl` | `KubectlCommands` | `cmds::cloud::container` |
| `Oc` | `OcCommands` | `cmds::cloud::container` |
| `Summary` | `command` | `cmds::system::summary` |
| `Grep` | `max_len`, `max`, `context_only`, `file_type`, `extra_args` | `cmds::system::search` |
| `Rg` | `extra_args` | `cmds::system::search` |
| `Init` | agente/modo/patch/trust/uninstall flags | `hooks::init` |
| `Wget` | `url`, `output`, `args` | `cmds::cloud::wget_cmd` |
| `Wc` | `args` | `cmds::system::wc_cmd` |
| `Gain` | filtros/format/reset | `analytics::gain` |
| `CcEconomics` | periodo/format | `analytics::cc_economics` |
| `Config` | `create` | `core::config` |
| `Jest`, `Vitest` | `args` | `cmds::js::vitest_cmd` |
| `Prisma` | `PrismaCommands` | `cmds::js::prisma_cmd` |
| `Tsc`, `Next`, `Lint`, `Prettier`, `Format`, `Playwright` | `args` | modulos JS especificos |
| `Cargo` | `CargoCommands` | `cmds::rust::cargo_cmd` |
| `Npm`, `Npx` | `args` | `cmds::js::npm_cmd` e roteamento inteligente |
| `Curl` | `args` | `cmds::cloud::curl_cmd` |
| `Discover` | `project`, `limit`, `all`, `since`, `format` | `discover::run` |
| `Session` | nenhum | `analytics::session_cmd` |
| `Telemetry` | `TelemetrySubcommand` | `core::telemetry_cmd` |
| `Learn` | filtros e thresholds | `learn::run` |
| `Run` | `command`, `args` | shell cru |
| `Proxy` | `args: Vec<OsString>` | passthrough com tracking |
| `Pipe` | `filter`, `passthrough` | `cmds::system::pipe_cmd` |
| `Trust`, `Untrust` | trust flags | `hooks::trust` |
| `Verify` | `filter`, `require_all` | `hooks::integrity` e `hooks::verify_cmd` |
| `Rewrite` | `args` | `hooks::rewrite_cmd` |
| `Hook` | `HookCommands` | `hooks::hook_cmd` |

### Enums auxiliares de subcomando

🟢 **CONFIRMADO** — Usados para roteamento hierarquico:

- `HookCommands`: `Claude`, `Cursor`, `Gemini`, `Copilot`, `Droid`, `Check`.
- `GitCommands`: `Diff`, `Log`, `Status`, `Show`, `Add`, `Commit`, `Checkout`, `Push`, `Pull`, `Branch`, `Fetch`, `Stash`, `Worktree`, `Other`.
- `PnpmCommands`: `List`, `Outdated`, `Install`, `Typecheck`, `Other`.
- `DockerCommands`: `Ps`, `Images`, `Logs`, `Compose`, `Other`.
- `ComposeCommands`: `Ps`, `Logs`, `Build`, `Other`.
- `KubectlCommands` e `OcCommands`: `Get`, `Pods`, `Services`, `Logs`, `Other`.
- `PrismaCommands`: `Generate`, `Migrate`, `DbPush`.
- `PrismaMigrateCommands`: `Dev`, `Status`, `Deploy`.
- `CargoCommands`: `Build`, `Test`, `Clippy`, `Check`, `Install`, `Nextest`, `Other`.
- `DotnetCommands`: `Build`, `Test`, `Restore`, `Format`, `Other`.
- `GoCommands`: `Test`, `Build`, `Vet`, `Other`.
- `GtCommands`: `Log`, `Submit`, `Sync`, `Restack`, `Create`, `Branch`, `Other`.

### Constantes locais

| Nome | Tipo | Escopo | Descricao |
|---|---|---|---|
| `CAP` | `usize` | `Commands::Proxy` | Limite de captura de stdout/stderr para tracking: `1_048_576` bytes por stream. |
| `PROXY_CHILD_PID` | `AtomicU32` | `Commands::Proxy` | PID do processo filho para handler Unix de `SIGINT`/`SIGTERM`. |

## Modulo `cmds`

### Entidades estruturais

| Entidade | Tipo | Local | Descricao |
|---|---|---|---|
| `FilterResult` | struct | `cloud/aws_cmd.rs` | Saida filtrada de filtros AWS, com texto e metadados de recuperacao. |
| `ContainerCmd` | enum | `cloud/container.rs` | Operacoes Docker/Kubernetes/OpenShift suportadas pelo formatter comum. |
| `BinlogIssue` | struct | `dotnet/binlog.rs` | Diagnostico extraido de binlog/texto .NET. |
| `BuildSummary` | struct | `dotnet/binlog.rs` | Resumo de build .NET. |
| `TestSummary` | struct | `dotnet/binlog.rs` | Resumo de testes .NET/TRX/binlog. |
| `RestoreSummary` | struct | `dotnet/binlog.rs` | Resumo de restore .NET. |
| `GitCommand` | enum | `git/git.rs` | Comandos Git especializados suportados internamente. |
| `GitStatusState` | enum | `git/git.rs` | Estado especial detectado em `git status` (merge, rebase, etc.). |
| `CommitOutcome` | enum | `git/git.rs` | Classificacao do resultado de commit. |
| `VitestJsonOutput` | struct | `js/vitest_cmd.rs` | Modelo parcial do JSON do Vitest/Jest. |
| `VitestParser` | struct | `js/vitest_cmd.rs` | Parser que implementa `OutputParser` para testes JS. |
| `MvnPhase` | enum | `jvm/mvn_cmd.rs` | Fase Maven detectada: test, compile, package ou passthrough. |
| `SurefireBlock` | struct | `jvm/mvn_cmd.rs` | Bloco de saida Surefire analisado por classe de teste. |
| `FailuresSummaryCap` | struct | `jvm/mvn_cmd.rs` | Controle de limite de falhas emitidas. |
| `PhpTestRunner` | enum | `php/utils.rs` | Runner PHP detectado para filtros compartilhados. |
| `MypyError` | struct | `python/mypy_cmd.rs` | Erro tipado de mypy. |
| `PytestCounts` | struct | `python/pytest_cmd.rs` | Contadores extraidos do resumo pytest. |
| `RuffDiagnostic` | struct | `python/ruff_cmd.rs` | Diagnostico JSON do Ruff. |
| `RspecOutput` | struct | `ruby/rspec_cmd.rs` | Modelo do JSON RSpec. |
| `RubocopOutput` | struct | `ruby/rubocop_cmd.rs` | Modelo do JSON RuboCop. |
| `CargoCommand` | enum | `rust/cargo_cmd.rs` | Operacoes Cargo suportadas. |
| `CargoBuildHandler` | struct | `rust/cargo_cmd.rs` | Handler de blocos para saida build/check/clippy. |
| `CargoTestHandler` | struct | `rust/cargo_cmd.rs` | Handler de blocos para saida de testes Cargo. |
| `ErrorStreamFilter` | struct | `rust/runner.rs` | Filtro streaming generico para `rtk err`. |
| `FindArgs` | struct | `system/find_cmd.rs` | Argumentos normalizados de `rtk find`/`find` nativo. |
| `Engine` | enum | `system/search.rs` | Motor de busca: `grep` ou `rg`. |
| `WcMode` | enum | `system/wc_cmd.rs` | Modo de formatacao de `wc`. |
| `OutputType` | enum | `system/summary.rs` | Classificacao heuristica da saida resumida. |

### Constantes e limites recorrentes

| Nome | Local | Descricao |
|---|---|---|
| `CAP_LIST`, `CAP_WARNINGS`, `CAP_INVENTORY` | `core::truncate` importado | Limites compartilhados para listas, warnings/falhas e inventarios. |
| `MAX_ITEMS` | `cloud/aws_cmd.rs` | Limite de itens em filtros AWS. |
| `JSON_COMPRESS_DEPTH` | `cloud/aws_cmd.rs` | Profundidade de compactacao JSON generica AWS. |
| `MAX_LOG_EVENTS` | `cloud/aws_cmd.rs` | Limite de eventos de logs. |
| `MAX_TABLE_ROWS` | `cloud/psql_cmd.rs` | Limite de linhas de tabela psql. |
| `MAX_RESPONSE_SIZE` | `cloud/curl_cmd.rs` | Limite de corpo exibido por curl. |
| `DOTNET_CLI_UI_LANGUAGE` | `dotnet/dotnet_cmd.rs` | Variavel usada para normalizar lingua do dotnet CLI. |
| `SENSITIVE_ENV_VARS` | `dotnet/binlog.rs` | Lista de variaveis sensiveis a serem ocultadas. |
| `MAX_MVN_FAILING_CLASSES` | `jvm/mvn_cmd.rs` | Limite de classes de teste Maven emitidas. |
| `MAX_FAILURES_SHOWN` | `php/phpunit_cmd.rs` | Limite de falhas PHPUnit exibidas. |
| `MAX_PYTEST_FAILURES` | `python/pytest_cmd.rs` | Limite de falhas pytest exibidas. |
| `MAX_TRACEBACK_FRAMES` | `python/uv_cmd.rs` | Limite de frames exibidos por traceback uv/python. |
| `MAX_RSPEC_FAILURES` | `ruby/rspec_cmd.rs` | Limite reduzido de falhas RSpec. |
| `MAX_RUNNER_FAILURES` | `rust/runner.rs` | Limite de falhas no runner generico. |
| `VALUE_FLAGS_SHORT`/`VALUE_FLAGS_LONG` | `system/search.rs` | Flags de grep/rg que consomem valor. |

### Contratos de funcao recorrentes

| Padrao | Assinatura comum | Descricao |
|---|---|---|
| Runner de comando | `pub fn run(args: &[String], verbose: u8) -> Result<i32>` | Entrada padrao para wrappers chamados por `main`. |
| Filtro puro | `fn filter_*(output: &str) -> String` | Transforma raw output em saida compacta. |
| Filtro JSON | `fn filter_*(json_str: &str) -> Option<FilterResult/String>` | Parseia JSON e cai para fallback em erro. |
| Passthrough | `run_passthrough(args: &[OsString], verbose: u8)` | Executa ferramenta sem filtragem, preservando compatibilidade. |
| Stream filter | `impl StreamFilter` ou `impl BlockHandler` | Processa linhas/blocos de saida longa. |

## Modulo `core`

### Configuracao

| Entidade | Tipo | Local | Descricao |
|---|---|---|---|
| `Config` | struct | `src/core/config.rs` | Configuracao raiz carregada de `~/.config/rtk/config.toml`. |
| `TrackingConfig` | struct | `src/core/config.rs` | Liga/desliga tracking, define retencao e caminho opcional do banco. |
| `DisplayConfig` | struct | `src/core/config.rs` | Preferencias de cores, emoji e largura maxima. |
| `FilterConfig` | struct | `src/core/config.rs` | Diretorios/arquivos ignorados por filtros. |
| `TeeConfig` | struct | `src/core/tee.rs` | Configuracao de raw output recovery. |
| `TelemetryConfig` | struct | `src/core/config.rs` | Consentimento e habilitacao de telemetria. |
| `HooksConfig` | struct | `src/core/config.rs` | Exclusoes e prefixos transparentes para rewrite de hooks. |
| `LimitsConfig` | struct | `src/core/config.rs` | Limites globais configuraveis para grep/status/passthrough. |

### Execucao e streaming

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `RunOptions` | struct | `src/core/runner.rs` | `tee_label`, `filter_stdout_only`, `skip_filter_on_failure`, `no_trailing_newline`, `inherit_stdin`. |
| `RunMode` | enum | `src/core/runner.rs` | `Filtered`, `FilteredWithExit`, `Streamed`, `Passthrough`. |
| `StreamFilter` | trait | `src/core/stream.rs` | `feed_line`, `flush`, `on_exit`. |
| `BlockHandler` | trait | `src/core/stream.rs` | Contrato para detectar e formatar blocos. |
| `LineHandler` | trait | `src/core/stream.rs` | Contrato para filtros orientados a linha. |
| `StdinFilter` | trait | `src/core/stream.rs` | Filtro opcional de stdin. |
| `FilterMode` | enum | `src/core/stream.rs` | `Streaming`, `Buffered`, `CaptureOnly`, `Passthrough`. |
| `StdinMode` | enum | `src/core/stream.rs` | `Inherit`, `Filter`, `Null`. |
| `StreamResult` | struct | `src/core/stream.rs` | `exit_code`, `raw`, `raw_stdout`, `raw_stderr`, `filtered`. |
| `CaptureResult` | struct | `src/core/stream.rs` | `stdout`, `stderr`, `exit_code`. |

### Tracking

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `Tracker` | struct | `src/core/tracking.rs` | Wrapper de conexao SQLite. |
| `CommandRecord` | struct | `src/core/tracking.rs` | `timestamp`, `rtk_cmd`, `saved_tokens`, `savings_pct`. |
| `GainSummary` | struct | `src/core/tracking.rs` | Totais, medias, top comandos e serie por dia. |
| `DayStats` | struct | `src/core/tracking.rs` | Estatisticas diarias serializaveis. |
| `WeekStats` | struct | `src/core/tracking.rs` | Estatisticas semanais serializaveis. |
| `MonthStats` | struct | `src/core/tracking.rs` | Estatisticas mensais serializaveis. |
| `ParseFailureRecord` | struct | `src/core/tracking.rs` | Falha de parse individual. |
| `ParseFailureSummary` | struct | `src/core/tracking.rs` | Total, taxa de recuperacao, top comandos e recentes. |
| `TimedExecution` | struct | `src/core/tracking.rs` | Timer usado para registrar execucao e economia. |

### TOML filters

| Entidade | Tipo | Local | Descricao |
|---|---|---|---|
| `TomlFilterDef` | struct | `src/core/toml_filter.rs` | Definicao declarativa de filtro: match, replace, strip/keep, head/tail/max, on_empty. |
| `CompiledFilter` | struct | `src/core/toml_filter.rs` | Filtro compilado com regexes prontas e flags de comportamento. |
| `Lossiness` | enum | `src/core/toml_filter.rs` | `None`, `Tail { tee_payload, tail_offset }`, `Whole`. |
| `TomlFilterTestDef` | struct | `src/core/toml_filter.rs` | Teste inline de filtro TOML. |
| `TestOutcome` | struct | `src/core/toml_filter.rs` | Resultado de um teste inline. |
| `VerifyResults` | struct | `src/core/toml_filter.rs` | Agregado de testes e filtros sem teste. |

### Filtros de codigo e utilitarios

| Entidade | Tipo | Local | Descricao |
|---|---|---|---|
| `FilterLevel` | enum | `src/core/filter.rs` | `None`, `Minimal`, `Aggressive`. |
| `Language` | enum | `src/core/filter.rs` | Rust, Python, JS/TS, Go, C/C++, Java, Ruby, Shell, Data, Unknown. |
| `FilterStrategy` | trait | `src/core/filter.rs` | Estrategia `filter(content, lang) -> String`. |
| `NoFilter` / `MinimalFilter` / `AggressiveFilter` | structs | `src/core/filter.rs` | Implementacoes de reducao de codigo. |
| `TeeMode` | enum | `src/core/tee.rs` | `Failures`, `Always`, `Never`. |
| `TelemetrySubcommand` | enum | `src/core/telemetry_cmd.rs` | `Status`, `Enable`, `Disable`, `Forget`. |

### Constantes

| Nome | Local | Valor/Papel |
|---|---|---|
| `RTK_DATA_DIR` | `src/core/constants.rs` | Diretorio de dados `rtk`. |
| `HISTORY_DB` | `src/core/constants.rs` | Nome do banco `history.db`. |
| `CONFIG_TOML` | `src/core/constants.rs` | Nome do arquivo `config.toml`. |
| `FILTERS_TOML` | `src/core/constants.rs` | Nome do arquivo `filters.toml`. |
| `TRUSTED_FILTERS_JSON` | `src/core/constants.rs` | Registro de trust de filtros. |
| `DEFAULT_HISTORY_DAYS` | `src/core/constants.rs` | Retencao padrao de 90 dias. |
| `RTK_META_COMMANDS` | `src/core/constants.rs` | Subcomandos RTK que nao devem cair em fallback raw. |
| `RAW_CAP` | `src/core/stream.rs` | Limite de captura raw: 10 MiB. |
| `CAP_ERRORS` | `src/core/truncate.rs` | Limite global de erros: 20. |
| `CAP_WARNINGS` | `src/core/truncate.rs` | Limite global de warnings/falhas: 10. |
| `CAP_LIST` | `src/core/truncate.rs` | Limite global de listas: 20. |
| `CAP_INVENTORY` | `src/core/truncate.rs` | Limite global de inventarios: 50. |

## Modulo `hooks`

### Instalacao e lifecycle

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `PatchMode` | enum | `src/hooks/init.rs` | `Ask`, `Auto`, `Skip`. |
| `FilterTrust` | enum | `src/hooks/init.rs` | `Ask`, `Trust`, `Skip`. |
| `PatchResult` | enum | `src/hooks/init.rs` | `Patched`, `AlreadyPresent`, `Declined`, `Skipped`, `WouldPatch`. |
| `InitContext` | struct | `src/hooks/init.rs` | `verbose: u8`, `dry_run: bool`. |
| `DroidLayout` | enum | `src/hooks/init.rs` | Layouts aceitos para hooks/settings do Factory Droid. |
| `DroidHookFile` | struct | `src/hooks/init.rs` | Caminho e layout do arquivo de hook Droid alvo. |

### Processamento de hooks

| Entidade | Tipo | Local | Descricao |
|---|---|---|---|
| `HookFormat` | enum | `src/hooks/hook_cmd.rs` | Formato Copilot/VS Code detectado: `VsCode`, `CopilotCli`, `PassThrough`. |
| `HookDecision` | enum | `src/hooks/hook_cmd.rs` | Decisao compartilhada: `AllowRewrite`, `AskRewrite`, `Defer`, `Deny`. |
| `PayloadAction` | enum | `src/hooks/hook_cmd.rs` | Resultado do processamento Claude: `Rewrite`, `Skip`, `Ignore`. |
| `PermissionVerdict` | enum | `src/hooks/permissions.rs` | `Allow`, `Deny`, `Ask`, `Default`. |
| `Host` | enum | `src/hooks/permissions.rs` | `Claude`, `Cursor`, `Gemini`, `Droid`. |

### Integridade e trust

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `IntegrityStatus` | enum | `src/hooks/integrity.rs` | `Verified`, `Tampered { expected, actual }`, `NoBaseline`, `NotInstalled`, `OrphanedHash`. |
| `HookStatus` | enum | `src/hooks/hook_check.rs` | `Ok`, `Outdated`, `Missing`. |
| `TrustStore` | struct | `src/hooks/trust.rs` | `version`, `trusted: HashMap<String, TrustEntry>`. |
| `TrustEntry` | struct | `src/hooks/trust.rs` | `sha256`, `trusted_at`. |
| `TrustStatus` | enum | `src/hooks/trust.rs` | `Trusted`, `Untrusted`, `ContentChanged`, `EnvOverride`. |
| `AuditEntry` | struct | `src/hooks/hook_audit_cmd.rs` | `timestamp`, `action`, `original_cmd`, `_rewritten_cmd`. |

### Constantes de integracao

| Nome | Local | Valor/Papel |
|---|---|---|
| `REWRITE_HOOK_FILE` | `src/hooks/constants.rs` | Script legado `rtk-rewrite.sh`. |
| `GEMINI_HOOK_FILE` | `src/hooks/constants.rs` | Script Gemini `rtk-hook-gemini.sh`. |
| `CLAUDE_HOOK_COMMAND` | `src/hooks/constants.rs` | Comando nativo `rtk hook claude`. |
| `CURSOR_HOOK_COMMAND` | `src/hooks/constants.rs` | Comando nativo `rtk hook cursor`. |
| `DROID_HOOK_COMMAND` | `src/hooks/constants.rs` | Comando nativo `rtk hook droid`. |
| `PRE_TOOL_USE_KEY` | `src/hooks/constants.rs` | Evento `PreToolUse`. |
| `BEFORE_TOOL_KEY` | `src/hooks/constants.rs` | Evento `BeforeTool`. |
| `STDIN_CAP` | `src/hooks/hook_cmd.rs` | Limite de stdin de hook: 1 MiB. |
| `HASH_FILENAME` | `src/hooks/integrity.rs` | Sidecar `.rtk-hook.sha256`. |
| `CURRENT_HOOK_VERSION` | `src/hooks/hook_check.rs` | Versao esperada do hook legado. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `init::run` | flags + `PatchMode` + `InitContext` -> `Result<()>` | Seleciona modo de instalacao. |
| `patch_settings_json_command` | hook command + mode -> `Result<PatchResult>` | Insere hook em `settings.json`. |
| `hook_cmd::run_claude` | stdin JSON -> `Result<()>` | Processa PreToolUse Claude. |
| `hook_cmd::run_cursor` | stdin JSON -> `Result<()>` | Processa hook Cursor. |
| `hook_cmd::run_gemini` | stdin JSON -> `Result<()>` | Processa BeforeTool Gemini. |
| `hook_cmd::run_copilot` | stdin JSON -> `Result<()>` | Processa Copilot VS Code/CLI. |
| `hook_cmd::run_droid` | stdin JSON -> `Result<()>` | Processa PreToolUse Droid. |
| `rewrite_cmd::run` | `cmd: &str` -> `Result<()>`/exit code | Ponte shell para rewrite. |
| `permissions::check_command_for` | cmd + host -> `PermissionVerdict` | Avalia regras do host. |
| `integrity::verify_hook_at` | path -> `IntegrityStatus` | Verifica hash do hook. |
| `trust::check_trust_with_content` | path -> status + conteudo opcional | Gate de filtros TOML. |

## Modulo `analytics`

### Comandos e summaries

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `SessionSummary` | struct | `src/analytics/session_cmd.rs` | `id`, `date`, `total_cmds`, `rtk_cmds`, `output_tokens`. |
| `ExportData` | struct | `src/analytics/gain.rs` | `summary`, `daily`, `weekly`, `monthly`. |
| `ExportSummary` | struct | `src/analytics/gain.rs` | `total_commands`, `total_input`, `total_output`, `total_saved`, `avg_savings_pct`, `total_time_ms`, `avg_time_ms`. |

### ccusage

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `CcusageMetrics` | struct | `src/analytics/ccusage.rs` | `input_tokens`, `output_tokens`, `cache_creation_tokens`, `cache_read_tokens`, `total_tokens`, `total_cost`. |
| `CcusagePeriod` | struct | `src/analytics/ccusage.rs` | `key`, `metrics`. |
| `Granularity` | enum | `src/analytics/ccusage.rs` | `Daily`, `Weekly`, `Monthly`. |
| `DailyResponse` / `WeeklyResponse` / `MonthlyResponse` | structs internos | `src/analytics/ccusage.rs` | Vetores `daily`, `weekly`, `monthly`. |
| `DailyEntry` / `WeeklyEntry` / `MonthlyEntry` | structs internos | `src/analytics/ccusage.rs` | Chave de periodo com alias `period` + `CcusageMetrics`. |

### Economics

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `PeriodEconomics` | struct | `src/analytics/cc_economics.rs` | `label`, metricas ccusage, metricas RTK, `weighted_input_cpt`, `savings_weighted`, `blended_cpt`, `active_cpt`. |
| `Totals` | struct | `src/analytics/cc_economics.rs` | Totais agregados de custo, tokens, comandos RTK e economias estimadas. |

### Constantes

| Nome | Local | Valor/Papel |
|---|---|---|
| `WEIGHT_OUTPUT` | `src/analytics/cc_economics.rs` | Peso 5.0 para token de output. |
| `WEIGHT_CACHE_CREATE` | `src/analytics/cc_economics.rs` | Peso 1.25 para cache write. |
| `WEIGHT_CACHE_READ` | `src/analytics/cc_economics.rs` | Peso 0.1 para cache read. |
| `ESTIMATED_PRO_MONTHLY` | `src/analytics/gain.rs` | Baseline heuristico de 6.000.000 tokens/mes para quota Pro. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `gain::run` | flags de escopo/formato/periodo/reset -> `Result<()>` | Renderiza ou exporta economia de tokens RTK. |
| `ccusage::fetch` | `Granularity` -> `Result<Option<Vec<CcusagePeriod>>>` | Executa e parseia ccusage com degradacao graciosa. |
| `cc_economics::run` | periodo/formato/verbose -> `Result<()>` | Combina gasto Claude Code com economia RTK. |
| `session_cmd::run` | `verbose: u8` -> `Result<()>` | Mostra adocao de RTK em sessoes Claude Code recentes. |
| `count_rtk_commands` | `&[ExtractedCommand]` -> `(usize, usize, usize)` | Conta comandos totais, comandos cobertos por RTK e output somado. |
| `convert_saturday_to_monday` | `&str` -> `Option<String>` | Alinha semanas RTK legadas ao padrao ISO do ccusage. |

## Modulo `discover`

### Aggregation e comando

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `SupportedBucket` | struct interno | `src/discover/mod.rs` | `rtk_equivalent`, `category`, `count`, `total_output_tokens`, `total_raw_output_tokens`, `command_counts`. |
| `UnsupportedBucket` | struct interno | `src/discover/mod.rs` | `count`, `example`. |

### Lexer

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `TokenKind` | enum | `src/discover/lexer.rs` | `Arg`, `Operator`, `Pipe`, `Redirect`, `Shellism`. |
| `ParsedToken` | struct | `src/discover/lexer.rs` | `kind`, `value`, `offset`. |

### Provider de sessoes

| Entidade | Tipo | Local | Campos/contratos |
|---|---|---|---|
| `ExtractedCommand` | struct | `src/discover/provider.rs` | `command`, `output_len`, `session_id`, `output_content`, `is_error`, `sequence_index`. |
| `SessionProvider` | trait | `src/discover/provider.rs` | `discover_sessions(project_filter, since_days)`, `extract_commands(path)`. |
| `ClaudeProvider` | struct | `src/discover/provider.rs` | Provider atual para JSONL Claude Code. |

### Registry e regras

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `Classification` | enum | `src/discover/registry.rs` | `Supported { rtk_equivalent, category, estimated_savings_pct, status }`, `Unsupported { base_command }`, `Ignored`. |
| `GolangciRunParts` | struct interno | `src/discover/registry.rs` | `global_segment`, `run_segment`. |
| `ExcludePattern` | enum interno | `src/discover/registry.rs` | `Regex(Regex)`, `Prefix(String)`. |
| `RtkRule` | struct | `src/discover/rules.rs` | `pattern`, `rtk_cmd`, `rewrite_prefixes`, `category`, `savings_pct`, `subcmd_savings`, `subcmd_status`. |

### Report

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `RtkStatus` | enum | `src/discover/report.rs` | `Existing`, `Passthrough`, `NotSupported`. |
| `SupportedEntry` | struct | `src/discover/report.rs` | `command`, `count`, `rtk_equivalent`, `category`, `estimated_savings_tokens`, `estimated_savings_pct`, `rtk_status`. |
| `UnsupportedEntry` | struct | `src/discover/report.rs` | `base_command`, `count`, `example`. |
| `AgentIntegrationStatus` | struct | `src/discover/report.rs` | `cursor_hook_installed`, `hermes_plugin_installed`, `copilot_hook_installed`. |
| `DiscoverReport` | struct | `src/discover/report.rs` | `sessions_scanned`, `total_commands`, `already_rtk`, `since_days`, `supported`, `unsupported`, `parse_errors`, `rtk_disabled_count`, `rtk_disabled_examples`, `agent_status`. |

### Constantes

| Nome | Local | Valor/Papel |
|---|---|---|
| `RULES` | `src/discover/rules.rs` | 86 regras de classificacao/rewrite. |
| `IGNORED_PREFIXES` | `src/discover/rules.rs` | Prefixos de comandos shell/operacionais ignorados. |
| `IGNORED_EXACT` | `src/discover/rules.rs` | Comandos exatos ignorados (`cd`, `echo`, `fi`, `done`, etc.). |
| `PHP_TOOL_NAMES` | `src/discover/registry.rs` | Ferramentas PHP normalizadas (`phpunit`, `phpstan`, `ecs`, `pest`, `paratest`, `pint`). |
| `BUILTIN_TRANSPARENT_PREFIXES` | `src/discover/registry.rs` | `noglob`, `command`, `builtin`, `exec`, `nocorrect`. |
| `MAX_PREFIX_DEPTH` | `src/discover/registry.rs` | Limite de recursao de prefixos: 10. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `discover::run` | filtros/periodo/formato -> `Result<()>` | Orquestra scan de sessoes e relatorio. |
| `tokenize` | `&str` -> `Vec<ParsedToken>` | Lexer shell-aware com offsets. |
| `contains_unattestable_construct` | `&str` -> `bool` | Gate de seguranca para permissoes. |
| `split_for_permissions` | `&str` -> `Vec<&str>` | Segmenta comandos para permissao, truncando redirects. |
| `split_on_operators` | `&str`, `stop_at_pipe` -> `Vec<&str>` | Divide comandos respeitando aspas e pipes. |
| `ClaudeProvider::encode_project_path` | `&str` -> `String` | Replica slug de diretorio de projeto Claude Code. |
| `classify_command` | `&str` -> `Classification` | Classifica comando contra regras RTK. |
| `rewrite_command` | cmd + exclusoes + prefixos -> `Option<String>` | Reescreve comando shell quando seguro e suportado. |
| `strip_disabled_prefix` | `&str` -> `(&str, &str)` | Separa prefixo env/sudo/env do comando real. |
| `format_text` / `format_json` | `&DiscoverReport` -> `String` | Renderiza relatorio humano ou JSON. |

## Modulo `learn`

### Detector

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `ErrorType` | enum | `src/learn/detector.rs` | `UnknownFlag`, `CommandNotFound`, `WrongSyntax`, `WrongPath`, `MissingArg`, `PermissionDenied`, `Other(String)`. |
| `CorrectionPair` | struct | `src/learn/detector.rs` | `wrong_command`, `right_command`, `error_output`, `error_type`, `confidence`. |
| `CorrectionRule` | struct | `src/learn/detector.rs` | `wrong_pattern`, `right_pattern`, `error_type`, `occurrences`, `base_command`, `example_error`. |
| `CommandExecution` | struct | `src/learn/detector.rs` | `command`, `is_error`, `output`. |

### Constantes e regexes

| Nome | Local | Valor/Papel |
|---|---|---|
| `CORRECTION_WINDOW` | `src/learn/detector.rs` | Busca correcao nos proximos 3 comandos. |
| `MIN_CONFIDENCE` | `src/learn/detector.rs` | Limiar interno 0.6 para aceitar par detectado. |
| `UNKNOWN_FLAG_RE` | `src/learn/detector.rs` | Detecta flag/opcao desconhecida ou invalida. |
| `CMD_NOT_FOUND_RE` | `src/learn/detector.rs` | Detecta command not found / comando nao reconhecido. |
| `WRONG_PATH_RE` | `src/learn/detector.rs` | Detecta arquivo/caminho inexistente. |
| `MISSING_ARG_RE` | `src/learn/detector.rs` | Detecta argumento/valor obrigatorio ausente. |
| `PERMISSION_DENIED_RE` | `src/learn/detector.rs` | Detecta permissao/acesso negado. |
| `USER_REJECTION_RE` | `src/learn/detector.rs` | Filtra cancelamento/rejeicao humana, nao erro de CLI. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `learn::run` | filtros/formato/limiares -> `Result<()>` | Orquestra deteccao e saida de regras. |
| `is_command_error` | `is_error: bool`, `output: &str` -> `bool` | Decide se tool_result representa erro CLI real. |
| `classify_error` | `&str` -> `ErrorType` | Classifica texto de erro. |
| `extract_base_command` | `&str` -> `String` | Normaliza base command por primeiros 1-2 tokens. |
| `command_similarity` | `&str`, `&str` -> `f64` | Similaridade Jaccard com 0.5 base para mesmo comando. |
| `find_corrections` | `&[CommandExecution]` -> `Vec<CorrectionPair>` | Detecta pares erro-correcao na janela. |
| `deduplicate_corrections` | `Vec<CorrectionPair>` -> `Vec<CorrectionRule>` | Agrupa pares por base/erro/diff token. |
| `format_console_report` | rules + contadores -> `String` | Renderiza relatorio textual. |
| `write_rules_file` | rules + path -> `Result<()>` | Gera Markdown de regras CLI. |

## Modulo `parser`

### Resultado de parse

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `ParseResult<T>` | enum generico | `src/parser/mod.rs` | `Full(T)`, `Degraded(T, Vec<String>)`, `Passthrough(String)`. |
| `OutputParser` | trait | `src/parser/mod.rs` | Associated type `Output`; contrato `parse(input: &str) -> ParseResult<Self::Output>`. |

### Tipos canonicos

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `TestResult` | struct | `src/parser/types.rs` | `total`, `passed`, `failed`, `skipped`, `duration_ms`, `failures`. |
| `TestFailure` | struct | `src/parser/types.rs` | `test_name`, `file_path`, `error_message`, `stack_trace`. |
| `DependencyState` | struct | `src/parser/types.rs` | `total_packages`, `outdated_count`, `dependencies`. |
| `Dependency` | struct | `src/parser/types.rs` | `name`, `current_version`, `latest_version`, `wanted_version`, `dev_dependency`. |

### Formatacao

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `FormatMode` | enum | `src/parser/formatter.rs` | `Compact`, `Verbose`, `Ultra`. |
| `TokenFormatter` | trait | `src/parser/formatter.rs` | `format_compact`, `format_verbose`, `format_ultra`, `format`. |

### Constantes e limites

| Nome | Local | Valor/Papel |
|---|---|---|
| `MAX_DEPS_LISTING` | `src/parser/formatter.rs` | Alias local de `CAP_INVENTORY` para limitar listagens de dependencias. |
| `passthrough_max_chars` | `src/core/config.rs` | Limite configuravel usado por `truncate_passthrough`; default observado: 2000. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `ParseResult::tier` | `&self` -> `u8` | Retorna 1, 2 ou 3 conforme degradacao. |
| `ParseResult::is_ok` | `&self` -> `bool` | Verdadeiro para `Full` e `Degraded`. |
| `ParseResult::map` | transforma `T` em `U` | Preserva tier e warnings ao mapear dado estruturado. |
| `ParseResult::warnings` | `&self` -> `Vec<String>` | Retorna warnings apenas no tier degradado. |
| `OutputParser::parse_with_tier` | input + max_tier -> `ParseResult<Output>` | Forca passthrough quando o parser excede o tier maximo. |
| `truncate_passthrough` | `&str` -> `String` | Trunca pelo limite global de passthrough. |
| `truncate_output` | output + max chars -> `String` | Trunca por caracteres e anexa marcador RTK. |
| `extract_json_object` | `&str` -> `Option<&str>` | Extrai objeto JSON completo de output com prefixos. |
| `FormatMode::from_verbosity` | `u8` -> `FormatMode` | Mapeia 0 compacto, 1 verbose, 2+ ultra. |

## Modulo `filters`

### Arquivo TOML e definicoes

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `TomlFilterFile` | struct deser | `src/core/toml_filter.rs` | `schema_version`, `filters`, `tests`. |
| `TomlFilterDef` | struct deser | `src/core/toml_filter.rs` | `description`, `match_command`, `strip_ansi`, `replace`, `match_output`, `strip_lines_matching`, `keep_lines_matching`, `truncate_lines_at`, `head_lines`, `tail_lines`, `max_lines`, `on_empty`, `filter_stderr`. |
| `MatchOutputRule` | struct deser | `src/core/toml_filter.rs` | `pattern`, `message`, `unless`. |
| `ReplaceRule` | struct deser | `src/core/toml_filter.rs` | `pattern`, `replacement`. |
| `TomlFilterTestDef` | struct deser | `src/core/toml_filter.rs` | `name`, `input`, `expected`. |

### Tipos compilados

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `CompiledFilter` | struct | `src/core/toml_filter.rs` | `name`, `description`, `match_regex`, `strip_ansi`, `replace`, `match_output`, `line_filter`, limites, `on_empty`, `filter_stderr`. |
| `CompiledMatchOutputRule` | struct | `src/core/toml_filter.rs` | `pattern`, `message`, `unless`. |
| `CompiledReplaceRule` | struct | `src/core/toml_filter.rs` | `pattern`, `replacement`. |
| `LineFilter` | enum | `src/core/toml_filter.rs` | `None`, `Strip(RegexSet)`, `Keep(RegexSet)`. |
| `TomlFilterRegistry` | struct | `src/core/toml_filter.rs` | `filters: Vec<CompiledFilter>`. |
| `Lossiness` | enum | `src/core/toml_filter.rs` | `None`, `Tail { tee_payload, tail_offset }`, `Whole`. |

### Testes e verificacao

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `TestOutcome` | struct | `src/core/toml_filter.rs` | `filter_name`, `test_name`, `passed`, `actual`, `expected`. |
| `VerifyResults` | struct | `src/core/toml_filter.rs` | `outcomes`, `filters_without_tests`. |

### Catalogo built-in observado

| Metricas | Valor | Fonte |
|---|---:|---|
| Arquivos TOML em `src/filters/` | 63 | `find src/filters -name '*.toml'` |
| Blocos `[[tests.*]]` inline | 154 | `rg '^\\[\\[tests\\.' src/filters/*.toml` |
| Filtros ativos em `.rtk/filters.toml` | 0 | arquivo contem apenas template comentado |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `build.rs main` | cargo build script | Concatena `src/filters/*.toml`, valida TOML e nomes duplicados, grava `builtin_filters.toml`. |
| `TomlFilterRegistry::load` | () -> registry | Carrega filtros trusted project/global e built-ins embutidos. |
| `parse_and_compile` | content + source -> filters | Valida schema e compila filtros individuais. |
| `compile_filter` | name + def -> `CompiledFilter` | Compila regexes e transforma TOML em runtime model. |
| `apply_filter_with_info` | filter + stdout -> `(String, Lossiness)` | Executa pipeline de oito estagios e informa perda de informacao. |
| `find_filter_in` | command + filters -> Option | Retorna primeiro filtro cujo `match_command` casa com o comando. |

## Modulo `openclaw`

### Manifestos

| Entidade | Tipo | Local | Campos principais |
|---|---|---|---|
| `package.json` | manifest npm | `openclaw/package.json` | `name=@rtk-ai/rtk-rewrite`, `version=1.0.0`, `main=index.ts`, `license=Apache-2.0`, `files`. |
| `openclaw.plugin.json` | manifest plugin | `openclaw/openclaw.plugin.json` | `id=rtk-rewrite`, `name=RTK Token Optimizer`, `version=1.0.0`, `configSchema`, `uiHints`. |

### Configuracao

| Campo | Tipo | Default | Papel |
|---|---|---|---|
| `enabled` | boolean | `true` | Habilita/desabilita rewrite automatico. |
| `verbose` | boolean | `false` | Loga decisoes de rewrite/aprovacao no console. |

### Tipos e contratos

| Entidade | Tipo | Local | Campos/variantes principais |
|---|---|---|---|
| `RewriteVerdict` | union type | `openclaw/index.ts` | `"ask"`, `"deny"`. |
| Retorno `tryRewrite` | tuple | `openclaw/index.ts` | `[string | null, RewriteVerdict?]`. |
| `requireApproval` | objeto | `openclaw/index.ts` | `title`, `description`, `severity`, `timeoutBehavior`, `allowedDecisions`, `onResolution`. |

### Contratos de funcao principais

| Funcao | Assinatura resumida | Papel |
|---|---|---|
| `checkRtk` | () -> `boolean` | Verifica e cacheia disponibilidade de `rtk` no PATH. |
| `tryRewrite` | `command: string` -> `[string | null, RewriteVerdict?]` | Executa `rtk rewrite` e interpreta exit codes. |
| `register` | `api: any` -> void | Registra hook `before_tool_call` quando plugin esta habilitado e RTK existe. |

### Protocolo `rtk rewrite`

| Exit code | stdout | Efeito no plugin |
|---:|---|---|
| 0 | comando reescrito | Auto-aplica rewrite se stdout difere do comando original. |
| 1 | irrelevante | Passthrough sem alteracao. |
| 2 | irrelevante | Bloqueia a chamada `exec`. |
| 3 | comando reescrito | Requer aprovacao humana antes de aplicar. |
