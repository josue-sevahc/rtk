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
