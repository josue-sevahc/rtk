# Analise Tecnica do Codigo — Archaeologist

Projeto: `rtk`
Nivel de documentacao: `completo`
Organizacao das specs: `hybrid` — modulo tecnico na raiz, casos de uso/fluxos transversais relacionados aninhados nas fases posteriores.

## Escala de confianca

- 🟢 **CONFIRMADO** — extraido diretamente do codigo
- 🟡 **INFERIDO** — baseado em padroes observados
- 🔴 **LACUNA** — requer validacao humana

## Modulo `main`

### Proposito

🟢 **CONFIRMADO** — `src/main.rs` e o entrypoint do binario `rtk`. Ele define a CLI com `clap`, centraliza enums de subcomandos, aplica verificacoes globais, despacha cada comando para modulos especializados e implementa fallback para comandos nao reconhecidos.

### Arquivos analisados

- `src/main.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Declarar os modulos Rust principais: `analytics`, `cmds`, `core`, `discover`, `hooks`, `learn`, `parser`.
- 🟢 **CONFIRMADO** — Definir `AgentTarget`, `Cli`, `Commands` e enums de subcomandos especializados (`GitCommands`, `PnpmCommands`, `DockerCommands`, `CargoCommands`, etc.).
- 🟢 **CONFIRMADO** — Configurar flags globais: `verbose`, `ultra_compact`, `skip_env`.
- 🟢 **CONFIRMADO** — Restaurar `SIGPIPE` no Unix antes da execucao principal.
- 🟢 **CONFIRMADO** — Rodar telemetria diaria nao bloqueante, aviso de hook desatualizado e verificacao de integridade para comandos operacionais.
- 🟢 **CONFIRMADO** — Despachar comandos RTK para modulos especializados.
- 🟢 **CONFIRMADO** — Executar fallback quando `clap` falha para comandos que podem ser ferramentas externas.
- 🟢 **CONFIRMADO** — Implementar `run` e `proxy` como caminhos crus/controlados de execucao.

### Fluxo de controle

1. 🟢 **CONFIRMADO** — `main()` ajusta `SIGPIPE` no Unix para evitar abort em pipe fechado, chama `run_cli()`, imprime erro formatado se necessario e finaliza o processo com o codigo retornado.
2. 🟢 **CONFIRMADO** — `run_cli()` dispara `core::telemetry::maybe_ping()` antes do parse da CLI.
3. 🟢 **CONFIRMADO** — `Cli::try_parse()` separa dois caminhos:
   - sucesso: segue para avisos, integridade e dispatch;
   - erro de help/version: deixa `clap` encerrar;
   - erro comum: chama `run_fallback(e)`.
4. 🟢 **CONFIRMADO** — `run_cli()` chama `hooks::hook_check::maybe_warn()` para quase todos os comandos, exceto `gain`.
5. 🟢 **CONFIRMADO** — Se `is_operational_command(&cli.command)` retorna `true`, executa `hooks::integrity::runtime_check()`.
6. 🟢 **CONFIRMADO** — Um grande `match cli.command` roteia cada subcomando para o modulo apropriado e normaliza o codigo de saida.

### Algoritmos e regras relevantes

#### Fallback para comandos nao reconhecidos

🟢 **CONFIRMADO** — `run_fallback(parse_error)`:

- coleta `std::env::args().skip(1)`;
- se nao houver argumentos, deixa o erro do `clap` encerrar;
- se o primeiro token estiver em `core::constants::RTK_META_COMMANDS`, tambem deixa o erro do `clap` encerrar, impedindo que meta-comandos com flags erradas virem execucao externa;
- monta `raw_command` e remove ANSI da mensagem de erro para tracking;
- inicia `TimedExecution`;
- normaliza o primeiro token pelo basename para busca de filtros TOML;
- se `RTK_NO_TOML` nao estiver ativo e um filtro TOML casar, executa o comando capturando stdout e, opcionalmente, stderr;
- aplica `core::toml_filter::apply_filter_with_info`;
- se houver perda de informacao, tenta criar hint de recuperacao via `core::tee`;
- registra tracking e parse failure silencioso;
- se nenhum filtro TOML casar, executa o comando real com `Stdio::inherit()` e registra passthrough.

Regra de seguranca:

- 🟢 **CONFIRMADO** — meta-comandos falham fechados no parse; ferramentas externas podem cair em passthrough ou TOML filter.

#### Dispatch inteligente de comandos

🟢 **CONFIRMADO** — `run_cli()` encapsula varias adaptacoes antes de chamar modulos:

- `git`: reconstrói argumentos globais (`-C`, `-c`, `--git-dir`, `--work-tree`, `--no-pager`, `--no-optional-locks`, `--bare`, `--literal-pathspecs`) e despacha subcomandos para `cmds::git::git`.
- `glab`: adiciona `-R`/`-g` no final para nao interferir no sub-subcomando.
- `pnpm`: aceita filtros globais, mescla `--filter=<valor>` aos args e avisa quando filtros globais sao usados com `typecheck`.
- `kubectl`/`oc`: constroi argumentos de namespace ou logs via helpers compartilhados.
- `init`: roteia modos de instalacao/desinstalacao por agente e combinacao de flags.
- `npx`: detecta ferramentas conhecidas (`tsc`, `eslint`, `prisma`, `next`, `prettier`, `playwright`) e delega a filtros especializados; desconhecidos seguem por `npm_cmd::exec`.
- `hook check`: usa `discover::registry::rewrite_command` para prever rewrite.
- `run`: executa `sh -c`/`cmd /C` sem filtering/tracking especializado.
- `proxy`: executa comando externo, espelha stdout/stderr em tempo real, captura ate 1 MiB de cada stream para tracking e mata processo filho em sinais Unix.

#### Verificacao de integridade operacional

🟢 **CONFIRMADO** — `is_operational_command()` usa whitelist explicita. O comentario declara que novos comandos falham abertos quanto a integridade ate serem adicionados, evitando falsa seguranca.

#### Tratamento de sinais em proxy

🟢 **CONFIRMADO** — `Commands::Proxy` define `PROXY_CHILD_PID` estatico e registra handler Unix para `SIGINT`/`SIGTERM`. Ao receber sinal, mata o filho, espera sua finalizacao, restaura handler default e reemite o sinal.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `AgentTarget`
- `Cli`
- `Commands`
- `HookCommands`
- `GitCommands`
- `PnpmCommands`
- `DockerCommands`
- `ComposeCommands`
- `KubectlCommands`
- `OcCommands`
- `PrismaCommands`
- `PrismaMigrateCommands`
- `CargoCommands`
- `DotnetCommands`
- `GoCommands`
- `GtCommands`

### Dependencias internas

🟢 **CONFIRMADO** — `main` depende de todos os modulos top-level:

- `cmds::*` para roteamento de wrappers/filtros;
- `core::*` para configuracao, tracking, TOML filters, tee, utils, telemetry e comandos de sistema;
- `hooks::*` para init, integridade, rewrite, trust, auditoria e hook processors;
- `analytics::*` para `gain`, `cc-economics`, `session`;
- `discover::*` para `discover`, lexer e rewrite registry;
- `learn::*` para regras de correcao;
- `parser` e usado indiretamente pelos comandos especializados.

### Tratamento de erros

- 🟢 **CONFIRMADO** — `anyhow::Result<i32>` e usado em `run_cli()` e helpers.
- 🟢 **CONFIRMADO** — `main()` imprime `rtk: {:#}` e retorna `1` em erro.
- 🟢 **CONFIRMADO** — varios comandos convertem erros internos em codigo de saida, preservando compatibilidade CLI.
- 🟢 **CONFIRMADO** — `Read` acumula falhas por arquivo e imita mensagem `cat: file: erro`.
- 🟢 **CONFIRMADO** — fallback retorna `127` em comando nao encontrado.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `src/main.rs` contem testes unitarios para parsing de CLI, preservacao de flags, classificacao de subcomandos, fallback/meta commands, `shell_split`, rewrite, pnpm filters, SIGPIPE e formas de init/hook.

### Lacunas

- 🔴 **LACUNA** — O Archaeologist ainda nao validou runtime dos comandos; esta fase e estatica.
- 🔴 **LACUNA** — O volume de subcomandos dificulta garantir cobertura completa sem correlacionar cada dispatch com testes correspondentes.

## Modulo `cmds`

### Proposito

🟢 **CONFIRMADO** — `src/cmds/` contem os filtros e wrappers por comando/ecossistema. Cada submodulo executa uma ferramenta externa, captura ou transmite stdout/stderr, reduz o volume da saida e registra economia via infraestrutura de `core`.

### Arquivos analisados

- `src/cmds/README.md`
- `src/cmds/mod.rs`
- `src/cmds/cloud/*`
- `src/cmds/dotnet/*`
- `src/cmds/git/*`
- `src/cmds/go/*`
- `src/cmds/js/*`
- `src/cmds/jvm/*`
- `src/cmds/php/*`
- `src/cmds/python/*`
- `src/cmds/ruby/*`
- `src/cmds/rust/*`
- `src/cmds/system/*`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Executar comandos externos via `resolved_command`, `Command` e helpers de `core::runner`/`core::stream`.
- 🟢 **CONFIRMADO** — Filtrar saidas com funcoes puras (`&str -> String`) quando possivel.
- 🟢 **CONFIRMADO** — Usar parsing estruturado JSON/XML/NDJSON quando a ferramenta permite.
- 🟢 **CONFIRMADO** — Usar state machines/stream filters para saidas longas ou em blocos.
- 🟢 **CONFIRMADO** — Preservar exit codes reais e oferecer tee hints quando a saida compactada perde detalhes importantes.
- 🟢 **CONFIRMADO** — Delegar responsabilidade compartilhada para `core`: runner, tracking, truncamento, configuracao, TOML DSL e utilitarios.

### Submodulos por ecossistema

| Submodulo | Arquivos Rust | Papel |
|---|---:|---|
| `system` | 16 | Utilitarios genericos (`ls`, `tree`, `read`, `grep/rg`, `find`, `wc`, `env`, `json`, `log`, `deps`, `summary`, `format`, `pipe`, `smart`). |
| `php` | 11 | PHP, PHPUnit, Pest, ParaTest, PHPStan, Pint, ECS e Artisan. |
| `js` | 10 | npm/pnpm/npx, Vitest/Jest, TypeScript, ESLint, Next.js, Prettier, Playwright, Prisma. |
| `cloud` | 6 | AWS, Docker, Kubernetes/OpenShift, curl, wget, psql. |
| `git` | 6 | git, gh, glab, gt e diff local. |
| `dotnet` | 5 | dotnet build/test/restore/format, binlog, TRX e reports. |
| `python` | 6 | pytest, ruff, mypy, pip, uv. |
| `ruby` | 4 | rake/rails test, rspec, rubocop e utilitarios. |
| `rust` | 3 | cargo e runner generico `err`/`test`. |
| `go` | 3 | go test/build/vet e golangci-lint. |
| `jvm` | 3 | Maven e Gradle/Android Gradle wrapper. |

### Padroes de fluxo de controle

#### Runner padrao

🟢 **CONFIRMADO** — O fluxo declarado em `src/cmds/README.md` e seguido por muitos modulos:

1. construir `Command`;
2. escolher modo (`run_filtered`, `run_streamed`, `run_passthrough` ou execucao manual);
3. capturar stdout/stderr ou transmitir linha a linha;
4. aplicar filtro;
5. imprimir saida compacta;
6. registrar tracking raw vs filtered;
7. retornar exit code real.

#### Captura estruturada

🟢 **CONFIRMADO** — Usada onde ferramentas oferecem JSON/XML:

- AWS injeta/forca `--output json` para operacoes estruturadas.
- `gh`/`glab` usam JSON para listas/views quando possivel.
- `go test` usa NDJSON via `-json`.
- `golangci-lint` usa JSON.
- `dotnet` usa binlog e TRX.
- `phpstan`, `rubocop`, `pint`, `rspec`, `vitest`, `playwright` usam JSON quando possivel.
- `json_cmd` compacta JSON genericamente por profundidade.

#### State machines e streaming

🟢 **CONFIRMADO** — Usadas quando a ferramenta nao tem formato estruturado confiavel:

- `mvn_cmd.rs`: detecta fases Maven, blocos Surefire, reactor summary e boilerplate.
- `gradlew_cmd.rs`: streaming line filter para tarefas Gradle.
- `cargo_cmd.rs`: `BlockHandler` para build/test/clippy/check e parsers de JSON diagnostics.
- `runner.rs`: `ErrorStreamFilter` para `rtk err`/`rtk test`.
- `pytest_cmd.rs`, `rake_cmd.rs`, `phpunit_cmd.rs`: parsers textuais focados em falhas.
- `search.rs`: parsing compacto de saida `grep`/`rg`, com bypass para flags que alteram formato.

### Algoritmos por area

#### `git`

- 🟢 **CONFIRMADO** — `git.rs` implementa filtros para diff, show, log, status, add, commit, checkout, push, pull, branch, fetch, stash e worktree.
- 🟢 **CONFIRMADO** — `compact_diff()` limita diffs e preserva contexto essencial.
- 🟢 **CONFIRMADO** — `format_status_output()` interpreta porcelain output e estados especiais como rebase/merge/cherry-pick.
- 🟢 **CONFIRMADO** — `gh_cmd.rs` e `glab_cmd.rs` formatam PRs/issues/runs/releases/status/diffs; ambos tratam passthrough quando o usuario pede formatos especiais.
- 🟢 **CONFIRMADO** — `gt_cmd.rs` encapsula Graphite com compactacao e fallback passthrough.

#### `rust`

- 🟢 **CONFIRMADO** — `cargo_cmd.rs` usa `CargoCommand` para build, test, clippy, check, install, nextest e passthrough.
- 🟢 **CONFIRMADO** — `restore_double_dash()` e mencionado no README como correcao para preservar flags de teste apos `--`.
- 🟢 **CONFIRMADO** — Parsers agrupam erros/warnings, falhas de teste e regras de clippy; quando ha detalhes demais, tee hints preservam recuperabilidade.
- 🟢 **CONFIRMADO** — `runner.rs` oferece modo generico para comandos arbitrarios: `err` conserva erros/warnings; `test` conserva falhas.

#### `cloud`

- 🟢 **CONFIRMADO** — `aws_cmd.rs` contem 25 filtros especializados, cobrindo STS, S3, EC2, ECS, RDS, CloudFormation, CloudWatch Logs, Lambda, IAM, DynamoDB, EKS, SQS, Secrets Manager.
- 🟢 **CONFIRMADO** — `container.rs` formata Docker, Compose, Kubernetes e OpenShift; usa JSON em pods/services quando possivel.
- 🟢 **CONFIRMADO** — `curl_cmd.rs` detecta binario, trunca resposta e salva recuperacao.
- 🟢 **CONFIRMADO** — `psql_cmd.rs` compacta tabelas e formato expandido.
- 🟢 **CONFIRMADO** — `wget_cmd.rs` resume download, arquivo, tamanho e erros.

#### `.NET`

- 🟢 **CONFIRMADO** — `dotnet_cmd.rs` injeta binlog/report/TRX quando necessario, normaliza linguagem para `en-US` e mescla evidencias de binlog, TRX e stdout.
- 🟢 **CONFIRMADO** — `binlog.rs` implementa parsing binario/textual de build, test e restore, com scrubbing de variaveis sensiveis.
- 🟢 **CONFIRMADO** — `dotnet_trx.rs` extrai falhas de teste de arquivos TRX.
- 🟢 **CONFIRMADO** — `dotnet_format_report.rs` resume arquivos alterados por `dotnet format`.

#### `jvm`

- 🟢 **CONFIRMADO** — `mvn_cmd.rs` detecta fase (`Test`, `Compile`, `Package`, `Passthrough`) e aplica filtro correspondente.
- 🟢 **CONFIRMADO** — Maven possui guarda de rodape em ingles para evitar compactar locales desconhecidos incorretamente.
- 🟢 **CONFIRMADO** — Surefire e tratado por blocos, com reentrada para multiplas falhas por classe e limite de classes/falhas.
- 🟢 **CONFIRMADO** — `gradlew_cmd.rs` detecta build/test/connectedTest/lint/dependencies e escolhe filtro streaming ou buffered.

#### `js`

- 🟢 **CONFIRMADO** — `lint_cmd.rs` tambem funciona como roteador cross-ecosystem para Python (`mypy`/`ruff`) quando detecta projeto Python.
- 🟢 **CONFIRMADO** — `vitest_cmd.rs` e `playwright_cmd.rs` usam a infraestrutura `parser/` para resultados de teste.
- 🟢 **CONFIRMADO** — `pnpm_cmd.rs` compacta list/outdated/install e oferece passthrough.
- 🟢 **CONFIRMADO** — `prisma_cmd.rs` filtra generate/migrate/db push e extrai nomes de tabela/indice.
- 🟢 **CONFIRMADO** — `npm_cmd.rs`, `next_cmd.rs`, `tsc_cmd.rs`, `prettier_cmd.rs` encapsulam saidas comuns de toolchain JS.

#### `python`

- 🟢 **CONFIRMADO** — `pytest_cmd.rs` usa state machine para falhas, xfail/xpass e resumo.
- 🟢 **CONFIRMADO** — `ruff_cmd.rs` usa JSON para check e texto para format.
- 🟢 **CONFIRMADO** — `mypy_cmd.rs` agrupa erros por arquivo/codigo e preserva notas.
- 🟢 **CONFIRMADO** — `pip_cmd.rs` auto-detecta `uv` e compacta `list`/`outdated`.
- 🟢 **CONFIRMADO** — `uv_cmd.rs` preserva semantica `uv run` e compacta tracebacks/blocos de erro.

#### `php`

- 🟢 **CONFIRMADO** — `php_cmd.rs` detecta lint e Artisan.
- 🟢 **CONFIRMADO** — `test_output.rs` compartilha filtro de PHPUnit/Pest/ParaTest.
- 🟢 **CONFIRMADO** — `phpstan_cmd.rs` escolhe JSON ou texto conforme formato.
- 🟢 **CONFIRMADO** — `utils.rs` resolve `vendor/bin/*` antes de binarios globais e detecta runner de testes.

#### `ruby`

- 🟢 **CONFIRMADO** — `rake_cmd.rs` filtra Minitest por falhas.
- 🟢 **CONFIRMADO** — `rspec_cmd.rs` usa JSON com fallback textual.
- 🟢 **CONFIRMADO** — `rubocop_cmd.rs` usa JSON e agrupa offenses por severidade/cop.
- 🟢 **CONFIRMADO** — README declara auto-deteccao de `bundle exec` via `ruby_exec()`.

#### `system`

- 🟢 **CONFIRMADO** — `search.rs` diferencia `grep` e `rg`, preserva flags de formato via passthrough e compacta matches por arquivo.
- 🟢 **CONFIRMADO** — `pipe_cmd.rs` resolve filtros por nome e aplica filtros existentes a stdin, com protecao contra panics.
- 🟢 **CONFIRMADO** — `read.rs` aplica `core::filter` por nivel e suporta janelas de linhas.
- 🟢 **CONFIRMADO** — `find_cmd.rs` aceita sintaxe nativa e sintaxe RTK, rejeitando flags perigosas/complexas.
- 🟢 **CONFIRMADO** — `format_cmd.rs` detecta formatter e despacha para Prettier/Ruff/Black.
- 🟢 **CONFIRMADO** — `summary.rs` classifica saida como testes, build, logs, lista, JSON ou generica.

### Tratamento de erros e recuperacao

- 🟢 **CONFIRMADO** — Muitos filtros usam `RunOptions::stdout_only().tee(...)`, `tee_and_hint`, `force_tee_hint` ou `force_tee_tail_hint` para permitir recuperacao do raw quando a compactacao omite detalhes.
- 🟢 **CONFIRMADO** — Modulos com JSON geralmente caem para raw/passthrough quando parsing falha.
- 🟢 **CONFIRMADO** — Exit code real e preservado via wrappers de `core::runner` ou `exit_code_from_*`.
- 🟢 **CONFIRMADO** — Alguns filtros tem guardas de locale/formato para evitar resumo incorreto, especialmente Maven.

### Complexidade

🟢 **CONFIRMADO** — Complexidade alta. O modulo mistura dispatch por ecossistema, parsers estruturados, filtros por estado, streaming, captura manual, heuristicas de reducao, tee recovery, limites de truncamento e compatibilidade com ferramentas externas.

### Lacunas

- 🔴 **LACUNA** — A analise estatica nao executou as ferramentas externas, entao nao valida compatibilidade com versoes instaladas localmente.
- 🔴 **LACUNA** — Alguns filtros sao extensos o bastante para merecer specs unitarias mais profundas no Writer, sobretudo `git`, `cargo`, `aws`, `dotnet`, `mvn`, `search`.

## Modulo `core`

### Proposito

🟢 **CONFIRMADO** — `src/core/` contem a infraestrutura compartilhada e agnostica de dominio do RTK. O README declara explicitamente que este modulo nao deve conhecer comandos especificos, hooks ou agentes, e que e consumido pelos demais componentes como folha do grafo de dependencias.

### Arquivos analisados

- `src/core/README.md`
- `src/core/mod.rs`
- `src/core/args_utils.rs`
- `src/core/config.rs`
- `src/core/constants.rs`
- `src/core/display_helpers.rs`
- `src/core/filter.rs`
- `src/core/guard.rs`
- `src/core/runner.rs`
- `src/core/stream.rs`
- `src/core/tee.rs`
- `src/core/telemetry.rs`
- `src/core/telemetry_cmd.rs`
- `src/core/toml_filter.rs`
- `src/core/tracking.rs`
- `src/core/truncate.rs`
- `src/core/utils.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Expor blocos compartilhados via `mod.rs`: argumentos, configuracao, constantes, display, filtros de codigo, guard, runner, streaming, tee, telemetria, filtros TOML, tracking, truncamento e utilitarios.
- 🟢 **CONFIRMADO** — Centralizar configuracao em `~/.config/rtk/config.toml`, com defaults para tracking, display, filtros, tee, telemetria, hooks e limites.
- 🟢 **CONFIRMADO** — Padronizar execucao de comandos em `runner.rs`, com modos filtrado, filtrado com exit code, streaming e passthrough.
- 🟢 **CONFIRMADO** — Capturar stdout/stderr de processos filhos em `stream.rs`, preservar exit code e limitar raw capturado a 10 MiB por stream.
- 🟢 **CONFIRMADO** — Persistir historico de economia em SQLite e expor agregacoes diarias, semanais, mensais, por comando, por projeto e para telemetria.
- 🟢 **CONFIRMADO** — Implementar DSL de filtros TOML com lookup trust-gated, filtros built-in e testes inline via `rtk verify`.
- 🟢 **CONFIRMADO** — Salvar raw output recuperavel em tee files quando a compactacao perde detalhes.
- 🟢 **CONFIRMADO** — Garantir "never worse": saida filtrada nao deve emitir mais tokens estimados que a saida raw.
- 🟢 **CONFIRMADO** — Controlar telemetria opt-in, salt/hash de dispositivo, ping diario e comandos `telemetry status|enable|disable|forget`.

### Fluxo de controle

#### Runner compartilhado

1. 🟢 **CONFIRMADO** — `run()` inicia `TimedExecution` e monta `cmd_label`.
2. 🟢 **CONFIRMADO** — `RunMode::Filtered` e `RunMode::FilteredWithExit` chamam `run_captured_filter()`, que executa `stream::run_streaming()` em modo `CaptureOnly`.
3. 🟢 **CONFIRMADO** — Em falha com `skip_filter_on_failure`, stdout/stderr raw sao reemitidos e o tracking registra raw contra raw.
4. 🟢 **CONFIRMADO** — Em sucesso ou falha filtravel, o texto escolhido (`stdout` ou `stdout+stderr`) passa pelo filtro.
5. 🟢 **CONFIRMADO** — Se houver tee label, `print_with_hint()` escreve tee hint quando aplicavel e `emit_guarded()` aplica `guard::never_worse()`.
6. 🟢 **CONFIRMADO** — O timer registra `original_cmd`, `rtk_cmd`, raw usado para tracking e texto efetivamente mostrado.
7. 🟢 **CONFIRMADO** — `RunMode::Streamed` processa linha/bloco em tempo real e registra raw vs filtered; `RunMode::Passthrough` herda stdio e registra passthrough.

#### Streaming e captura de processo

🟢 **CONFIRMADO** — `run_streaming()` tem dois caminhos:

- passthrough: herda stdout/stderr/stdin conforme o modo, executa `cmd.status()` e retorna raw vazio;
- captura/streaming: cria pipes, le stdout e stderr em threads, acumula raw ate `RAW_CAP`, aplica filtro streaming quando fornecido, chama `on_exit()` e retorna `StreamResult`.

Regra de robustez:

- 🟢 **CONFIRMADO** — `ChildGuard` aguarda o processo filho no `Drop`, reduzindo risco de processos zumbis.
- 🟢 **CONFIRMADO** — `status_to_exit_code()` converte termino por sinal Unix para `128 + signal`.

#### Pipeline TOML

🟢 **CONFIRMADO** — `apply_filter_with_info()` aplica a DSL em 8 estagios:

1. `strip_ansi`;
2. substituicoes `replace` por linha;
3. `match_output` com `unless`;
4. `strip_lines_matching` ou `keep_lines_matching`;
5. `truncate_lines_at`;
6. `head_lines`/`tail_lines`;
7. `max_lines`;
8. `on_empty`.

🟢 **CONFIRMADO** — A funcao retorna tambem `Lossiness`, que classifica perda como `None`, `Tail` recuperavel via `tail -n +offset`, ou `Whole`.

#### Tracking

🟢 **CONFIRMADO** — `Tracker::new()` cria/abre SQLite, aplica migracoes idempotentes (`exec_time_ms`, `project_path`), ativa WAL/busy timeout de forma nao fatal e cria tabelas `commands` e `parse_failures`.

🟢 **CONFIRMADO** — `Tracker::record()` estima economia por `input_tokens - output_tokens`, calcula percentual, salva o cwd canonico em `project_path` e limpa historico acima de `DEFAULT_HISTORY_DAYS`.

🟢 **CONFIRMADO** — Consultas filtradas por projeto usam match exato ou `GLOB` com separador de path, evitando os curingas de `LIKE`.

### Algoritmos e regras relevantes

#### Never-worse guard

🟢 **CONFIRMADO** — `guard::never_worse(raw, filtered)` usa `tracking::estimate_tokens()` e retorna `raw` se a saida filtrada tiver mais tokens estimados que a original. Empates preservam a saida filtrada.

#### Recuperacao por tee

🟢 **CONFIRMADO** — `tee_raw()` respeita `RTK_TEE=0`, `TeeConfig.enabled`, `TeeMode` (`failures`, `always`, `never`), tamanho minimo de 500 bytes, diretorio configuravel por env/config/default e rotacao por quantidade de arquivos.

🟢 **CONFIRMADO** — `force_tee_hint()` e `force_tee_tail_hint()` gravam recuperacao mesmo em sucesso quando a saida foi truncada e precisa de caminho operacional para detalhes ocultos.

#### Filtros de codigo para `read`

🟢 **CONFIRMADO** — `filter.rs` define `FilterLevel` (`none`, `minimal`, `aggressive`), detecta linguagem por extensao e aplica estrategias de reducao. Formatos de dados (`json`, `yaml`, `toml`, `xml`, `csv`, `md`, etc.) nao passam por remocao de comentarios como codigo.

#### Argumentos `--`

🟢 **CONFIRMADO** — `args_utils::restore_double_dash_with_raw()` reconstrói `--` consumidos pelo `clap` comparando contagem em args raw e parseados, e retorna a regiao de argumentos do usuario para preservar posicoes originais.

#### Telemetria

🟢 **CONFIRMADO** — `maybe_ping()` so faz trabalho quando ha endpoint compilado, env/config nao desabilitam, consentimento e `Some(true)`, e o marcador local nao foi tocado nas ultimas 23 horas. O envio ocorre em thread separada e erros sao ignorados.

🟢 **CONFIRMADO** — `telemetry forget` desativa consentimento, remove salt/marker, tenta apagar o banco local de tracking e envia pedido de erasure se houver endpoint.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `Config`, `TrackingConfig`, `DisplayConfig`, `FilterConfig`, `TelemetryConfig`, `HooksConfig`, `LimitsConfig`
- `RunOptions`, `RunMode`, `StreamResult`, `CaptureResult`
- `StreamFilter`, `BlockHandler`, `LineHandler`, `StdinFilter`
- `Tracker`, `CommandRecord`, `GainSummary`, `DayStats`, `WeekStats`, `MonthStats`
- `TomlFilterDef`, `CompiledFilter`, `Lossiness`, `VerifyResults`
- `TeeConfig`, `TeeMode`
- `FilterLevel`, `Language`, `FilterStrategy`
- `TelemetrySubcommand`

### Dependencias internas

🟢 **CONFIRMADO** — `core` e consumido por `main`, `cmds`, `analytics`, `hooks`, `discover` e `learn`, mas seu README define que nao deve depender de modulos command-specific ou hook-specific. Na pratica, alguns arquivos usam funcoes de `hooks` para trust/telemetria/init, o que sera melhor reconciliado na fase Architect/Reviewer.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Config e tracking retornam `anyhow::Result`, mas varios pontos de telemetria/tee/tracking silencioso ignoram erro de proposito para nao bloquear o comando do usuario.
- 🟢 **CONFIRMADO** — `run_streaming()` trata `BrokenPipe` como encerramento tolerado ao escrever saida filtrada.
- 🟢 **CONFIRMADO** — `toml_filter` nao panica em filtros invalidos de usuario; emite warnings e ignora filtros problematicos.
- 🟢 **CONFIRMADO** — `resolved_command()` tenta `which::which()` e cai para `Command::new(name)` se a resolucao falhar.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `core` contem testes unitarios para configuracao, restore de `--`, truncamento, never-worse, filtros de codigo, stream/capture, tee, TOML DSL, tracking e utilitarios.

### Complexidade

🟢 **CONFIRMADO** — Complexidade alta. `core` concentra contratos transversais de execucao, captura concorrente de IO, persistencia SQLite, migracoes, DSL declarativa, telemetria opt-in, guardas de economia e recuperabilidade por tee.

### Lacunas

- 🔴 **LACUNA** — A analise estatica nao validou comportamento em Windows, embora existam caminhos especificos de PATHEXT e process signal handling condicional.
- 🔴 **LACUNA** — O README afirma que `core` nao deve importar hooks, mas `toml_filter.rs` e `telemetry.rs` fazem chamadas para `hooks`; isto exige interpretacao arquitetural posterior.
- 🔴 **LACUNA** — Nao foi executado teste de concorrencia real sobre SQLite/WAL e multiplas instancias.

## Modulo `hooks`

### Proposito

🟢 **CONFIRMADO** — `src/hooks/` e a camada de integracao com agentes LLM. Ela instala, remove, verifica, audita e executa hooks que interceptam comandos shell e os reescrevem para equivalentes RTK. O README define que o modulo nao possui a logica de rewrite em si: `rewrite_cmd.rs` e `hook_cmd.rs` delegam para `discover::registry`.

### Arquivos analisados

- `src/hooks/README.md`
- `src/hooks/mod.rs`
- `src/hooks/constants.rs`
- `src/hooks/init.rs`
- `src/hooks/hook_cmd.rs`
- `src/hooks/rewrite_cmd.rs`
- `src/hooks/permissions.rs`
- `src/hooks/integrity.rs`
- `src/hooks/hook_check.rs`
- `src/hooks/trust.rs`
- `src/hooks/verify_cmd.rs`
- `src/hooks/hook_audit_cmd.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Orquestrar `rtk init` para Claude, Cursor, Windsurf, Cline, Codex, Gemini, Copilot, Pi, Droid, Hermes e OpenCode.
- 🟢 **CONFIRMADO** — Aplicar patch idempotente em arquivos de configuracao de agentes, preservando conteudo existente e usando escrita atomica.
- 🟢 **CONFIRMADO** — Processar payloads de hooks nativos para Claude, Cursor, Gemini, Copilot e Droid.
- 🟢 **CONFIRMADO** — Consultar permissoes do host e aplicar precedencia `Deny > Ask > Allow > Default`.
- 🟢 **CONFIRMADO** — Expor `rtk rewrite` como ponte CLI com protocolo por exit code.
- 🟢 **CONFIRMADO** — Verificar integridade de hooks legados por hash SHA-256.
- 🟢 **CONFIRMADO** — Gerenciar trust de filtros TOML locais/globais antes que `core::toml_filter` os carregue.
- 🟢 **CONFIRMADO** — Auditar atividade de hooks quando `RTK_HOOK_AUDIT=1`.
- 🟢 **CONFIRMADO** — Detectar hooks ausentes/desatualizados e emitir aviso rate-limited.

### Fluxo de controle

#### `rtk init`

1. 🟢 **CONFIRMADO** — `init::run()` valida combinacoes de flags e escolhe o modo de instalacao.
2. 🟢 **CONFIRMADO** — Modos globais obrigatorios rejeitam execucao local quando a integracao nao suporta escopo de projeto.
3. 🟢 **CONFIRMADO** — `write_if_changed()` so escreve quando o conteudo difere; em `dry_run`, imprime a acao planejada.
4. 🟢 **CONFIRMADO** — `atomic_write()` grava em tempfile no mesmo diretorio e faz persist/rename.
5. 🟢 **CONFIRMADO** — `patch_settings_json_command()` le/cria `settings.json`, checa idempotencia, respeita `PatchMode`, faz backup `.bak`, insere hook e grava JSON formatado.
6. 🟢 **CONFIRMADO** — No modo global Claude, `run_default_mode()` migra hook script antigo, escreve `RTK.md`, atualiza `CLAUDE.md`, registra `rtk hook claude` em settings e cria template global de filtros.
7. 🟢 **CONFIRMADO** — Modos especializados instalam artefatos proprios: Codex atualiza `AGENTS.md`, Gemini instala script e `GEMINI.md`, Droid atualiza hooks/settings, Copilot cria hook config e instrucoes.

#### Reescrita via `rtk rewrite`

🟢 **CONFIRMADO** — `rewrite_cmd::run(cmd)` carrega exclusoes/prefixos transparentes do config, chama `evaluate()` e usa exit code como contrato:

- `0`: imprime rewrite e permite auto-allow;
- `1`: sem equivalente RTK, passthrough;
- `2`: regra deny, host deve lidar com bloqueio;
- `3`: imprime rewrite e força ask.

🟢 **CONFIRMADO** — `evaluate()` aplica deny antes de tudo, recusa constructos nao atestaveis, delega rewrite a `discover::registry::rewrite_command()` e transforma `Default` em `Ask`.

#### Processadores nativos de hook

🟢 **CONFIRMADO** — `hook_cmd.rs` limita stdin a 1 MiB e usa `writeln!` controlado para nao corromper protocolos JSON.

🟢 **CONFIRMADO** — `decide_hook_action()` converte permissoes do host em `HookDecision`: `AllowRewrite`, `AskRewrite`, `Defer` ou `Deny`.

🟢 **CONFIRMADO** — Claude/Copilot VS Code emitem `hookSpecificOutput` com `updatedInput` e `permissionDecision` quando permitido; `AskRewrite` preserva prompt do host.

🟢 **CONFIRMADO** — Gemini retorna JSON com `decision` (`allow`, `deny`, `ask_user`) e injeta `tool_input.command` quando ha rewrite.

🟢 **CONFIRMADO** — Cursor retorna `{}` em vazio/erro/no-op, ou JSON com `continue`, `permission` e `updated_input`.

🟢 **CONFIRMADO** — Droid reescreve via `updatedInput` sem `permissionDecision`, para manter a decisao com o fluxo nativo do Droid.

### Algoritmos e regras relevantes

#### Modelo de permissao

🟢 **CONFIRMADO** — `permissions::check_command_with_rules()` divide comandos compostos, aplica deny por segmento antes de qualquer allow, transforma constructos nao atestaveis em `Ask`, e so retorna `Allow` quando todos os segmentos nao vazios casam com allow rules.

🟢 **CONFIRMADO** — Regras Claude sao lidas de settings de projeto/global, incluindo `.local`; apenas escopos `Bash(...)` entram no matching.

🟢 **CONFIRMADO** — Hosts diferentes usam fontes diferentes: Cursor usa config global, Gemini respeita trust de workspace, Droid usa listas deny/block de escopos user e projeto.

#### Integridade de hooks

🟢 **CONFIRMADO** — `integrity::store_hash()` calcula SHA-256 do hook, grava sidecar `.rtk-hook.sha256` em formato `sha256sum`, e torna o arquivo read-only em Unix.

🟢 **CONFIRMADO** — `verify_hook_at()` classifica `Verified`, `Tampered`, `NoBaseline`, `NotInstalled` ou `OrphanedHash`.

🟢 **CONFIRMADO** — `runtime_check()` bloqueia execucao quando detecta `Tampered`, ignora hooks nativos sem script e apenas avisa em hash orfao.

#### Trust de filtros TOML

🟢 **CONFIRMADO** — `trust::check_trust_with_content()` so retorna conteudo para filtros `Trusted` ou `EnvOverride`; em erro, ausencia, hash mudado ou UTF-8 invalido, o filtro nao e carregado.

🟢 **CONFIRMADO** — `RTK_TRUST_PROJECT_FILTERS=1` so funciona em ambiente CI detectado.

🟢 **CONFIRMADO** — `run_trust()` mostra filtros ativos, resumo de risco (`replace`, `match_output`, catch-all), exige confirmacao ou `--yes`, e grava hash no trust store.

#### Auditoria e avisos

🟢 **CONFIRMADO** — `audit_log()` grava `timestamp | action | original | rewritten`, sanitizando quebras de linha e pipes, apenas quando `RTK_HOOK_AUDIT=1`.

🟢 **CONFIRMADO** — `hook_check::maybe_warn()` compara versao ou registro nativo e avisa no maximo uma vez por dia.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `PatchMode`, `FilterTrust`, `PatchResult`, `InitContext`
- `HookFormat`, `HookDecision`, `PayloadAction`
- `PermissionVerdict`, `Host`
- `IntegrityStatus`
- `HookStatus`
- `TrustStore`, `TrustEntry`, `TrustStatus`
- `AuditEntry`
- `DroidLayout`, `DroidHookFile`

### Dependencias internas

🟢 **CONFIRMADO** — `hooks` depende de `core` para config, constants, stream capture e TOML helpers; de `discover` para lexer e registry de rewrite; de `serde_json` para payloads/configuracoes; de `sha2` para hash; de `tempfile` para escrita atomica.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Processadores de hooks geralmente retornam `Ok(())` em input vazio, JSON invalido, no-op ou denial, preservando o contrato de nao bloquear o host.
- 🟢 **CONFIRMADO** — `rtk rewrite` usa `process::exit()` para comunicar estado ao shell hook.
- 🟢 **CONFIRMADO** — Init/uninstall falham com `anyhow` quando nao conseguem parsear ou gravar arquivos principais, mas varias migracoes stale sao best-effort.
- 🟢 **CONFIRMADO** — Trust falha fechado para filtros nao confiaveis.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `hooks` contem testes para matching de comandos de hook, exit-code protocol, permissoes, payloads de hosts, integridade, trust, patch idempotente, paths de agentes e fluxos de instalacao.

### Complexidade

🟢 **CONFIRMADO** — Complexidade alta. O modulo combina protocolos de varios agentes, patching idempotente de arquivos externos, politica de permissoes, seguranca de trust/integridade, migracoes legadas e comportamentos divergentes por host.

### Lacunas

- 🔴 **LACUNA** — A analise estatica nao executou instalacoes reais em ambientes Claude/Cursor/Gemini/Droid/Copilot.
- 🔴 **LACUNA** — O arquivo `init.rs` e muito extenso; o Writer deve quebrar specs por modo de agente para evitar perda de detalhes.
- 🔴 **LACUNA** — A compatibilidade exata com versoes futuras dos hosts depende de contratos externos que podem mudar.

## Modulo `analytics`

### Proposito

🟢 **CONFIRMADO** — `src/analytics/` concentra dashboards e relatorios read-only sobre economia de tokens, custo estimado e adocao de RTK em sessoes Claude Code. O README delimita explicitamente que o modulo nao grava no banco de tracking; gravacao pertence a `core`/`cmds`.

### Arquivos analisados

- `src/analytics/README.md`
- `src/analytics/mod.rs`
- `src/analytics/gain.rs`
- `src/analytics/ccusage.rs`
- `src/analytics/cc_economics.rs`
- `src/analytics/session_cmd.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Implementar `rtk gain`, com resumo de economia, historico diario/semanal/mensal, export JSON/CSV, escopo global ou por projeto, visualizacao em terminal e reset confirmado.
- 🟢 **CONFIRMADO** — Implementar `rtk cc-economics`, correlacionando uso/custo do Claude Code via `ccusage` com tokens economizados pelo RTK.
- 🟢 **CONFIRMADO** — Implementar `rtk session`, analisando sessoes Claude Code recentes e medindo adocao de comandos cobertos pelo RTK.
- 🟢 **CONFIRMADO** — Parser isolado para `ccusage`, com fallback para `npx ccusage` e degradacao graciosa quando a ferramenta nao existe ou falha.
- 🟢 **CONFIRMADO** — Exibir alertas operacionais de analytics: hook ausente/desatualizado, filtros customizados nao confiaveis e uso excessivo de `RTK_DISABLED=`.

### Fluxo de controle

#### `rtk gain`

1. 🟢 **CONFIRMADO** — `gain::run()` inicializa `Tracker` e resolve escopo opcional do projeto via `current_dir().canonicalize()`.
2. 🟢 **CONFIRMADO** — Se `--reset` for usado, exige confirmacao interativa salvo `--yes`; em stdin nao interativo, recusa por padrao.
3. 🟢 **CONFIRMADO** — Se `--failures` for usado, consulta `get_parse_failure_summary()` e mostra top comandos, taxa de recuperacao e falhas recentes.
4. 🟢 **CONFIRMADO** — Para `--format json|csv`, exporta resumo e periodos selecionados usando os metodos filtrados do `Tracker`.
5. 🟢 **CONFIRMADO** — Na visao padrao, busca `get_summary_filtered()`, imprime KPIs, tabela por comando, medidor de eficiencia, historico recente e analise de quota quando solicitado.
6. 🟢 **CONFIRMADO** — Views diaria/semanal/mensal usam `print_period_table()` e metodos filtrados (`get_all_days_filtered`, `get_by_week_filtered`, `get_by_month_filtered`).

#### `rtk cc-economics`

1. 🟢 **CONFIRMADO** — `cc_economics::run()` cria `Tracker` e escolhe texto, JSON ou CSV.
2. 🟢 **CONFIRMADO** — Cada periodo combina dados de `ccusage::fetch()` com estatisticas RTK de tracking.
3. 🟢 **CONFIRMADO** — O merge usa mapa por chave de periodo e aceita presenca independente de dados Claude Code ou RTK.
4. 🟢 **CONFIRMADO** — Para semanais, converte o `week_start` legado do RTK (sabado) para a segunda-feira ISO usada pelo ccusage.
5. 🟢 **CONFIRMADO** — Calcula metrica primaria por custo por token de input ponderado e, em modo verbose, tambem mostra metricas legacy blended/active.

#### `ccusage` parser

🟢 **CONFIRMADO** — `ccusage::fetch(granularity)` constroi o comando `ccusage daily|weekly|monthly --json --since 20250101`, preferindo binario no PATH e caindo para `npx --yes ccusage` quando possivel. Falhas de existencia, execucao ou exit code viram `Ok(None)` com warning; JSON invalido vira erro contextualizado.

🟢 **CONFIRMADO** — O parser aceita tanto chaves antigas (`date`, `week`, `month`) quanto a chave atual `period`, via `serde(alias = "period")`.

#### `rtk session`

🟢 **CONFIRMADO** — `session_cmd::run()` descobre sessoes Claude Code dos ultimos 30 dias, remove JSONL de subagents, ordena por mtime, limita aos 10 arquivos mais recentes, extrai Bash commands e monta tabela de adocao.

🟢 **CONFIRMADO** — `count_rtk_commands()` divide comandos encadeados com `discover::registry::split_command_chain()`, conta invocacoes explicitas `rtk ...` e comandos que `classify_command()` marcaria como suportados pelo hook.

### Algoritmos e regras relevantes

#### Economia e custo ponderado

🟢 **CONFIRMADO** — `cc_economics` usa pesos constantes: output = 5x input, cache write = 1.25x input, cache read = 0.1x input. O custo por token de input ponderado e calculado como `total_cost / weighted_units`; a economia estimada e `rtk_saved_tokens * weighted_input_cpt`.

🟡 **INFERIDO** — A data `--since 20250101` esta hardcoded apesar do comentario mencionar "last 90 days"; em 2026-07-14 isso cobre mais que 90 dias e pode ser intencional para historico amplo.

#### Classificacao de adocao

🟢 **CONFIRMADO** — A adocao por sessao e `rtk_cmds / total_cmds * 100`, com protecao contra divisao por zero.

🟢 **CONFIRMADO** — Output de comandos em sessoes e somado a partir de `ExtractedCommand.output_len`, apenas quando presente.

#### Alertas preventivos

🟢 **CONFIRMADO** — `check_rtk_disabled_bypass()` faz scan best-effort de ate 200 sessoes dos ultimos 7 dias e alerta quando mais de 10% dos Bash commands usam prefixo `RTK_DISABLED=`.

🟢 **CONFIRMADO** — `gain` consulta `hook_check::status()` e `hooks::trust::untrusted_active_filter_count()` para avisos que afetam economia silenciosamente.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `SessionSummary`
- `CcusageMetrics`, `CcusagePeriod`, `Granularity`
- `DailyResponse`, `DailyEntry`, `WeeklyResponse`, `WeeklyEntry`, `MonthlyResponse`, `MonthlyEntry`
- `PeriodEconomics`, `Totals`
- `ExportData`, `ExportSummary`

### Dependencias internas

🟢 **CONFIRMADO** — `analytics` depende de `core::tracking` para consultas SQLite, `core::display_helpers`/`core::utils` para formatacao, `discover::provider` e `discover::registry` para analise de sessoes, `hooks::hook_check`/`hooks::trust` para avisos e `core::stream`/`core::utils` para execucao do ccusage.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Falha ao inicializar `Tracker` ou carregar dados locais retorna `anyhow::Result` com contexto.
- 🟢 **CONFIRMADO** — `ccusage` indisponivel degrada para `Ok(None)`, preservando relatorios baseados somente em tracking RTK.
- 🟢 **CONFIRMADO** — `session` ignora arquivos de sessao cujo parse de comandos falha, e encerra com mensagem amigavel quando nao ha sessoes ou Bash commands.
- 🟢 **CONFIRMADO** — `gain --reset` e destrutivo, mas exige confirmacao e nao prossegue em stdin nao interativo sem `--yes`.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `analytics` contem testes para barra de progresso, contagem de comandos RTK, split de comandos encadeados, parsing JSONL de sessoes, parser ccusage com `date/week/month` e `period`, defaults de cache, conversao de semana e calculos economics.

### Complexidade

🟢 **CONFIRMADO** — Complexidade media-alta. O modulo nao tem mutacao primaria de dominio, mas cruza fontes heterogeneas (SQLite local, JSONL de sessoes Claude, subprocesso externo ccusage), aplica heuristicas financeiras e precisa manter output humano/exportavel consistente.

### Lacunas

- 🔴 **LACUNA** — A analise estatica nao executou `ccusage` real nem validou disponibilidade via `npx` em rede/ambiente do usuario.
- 🔴 **LACUNA** — A formula de precificacao depende de ratios externos de API citados em comentario; deve ser revalidada periodicamente.
- 🔴 **LACUNA** — O limite hardcoded `--since 20250101` diverge do comentario "last 90 days" e precisa decisao de produto.

## Modulo `discover`

### Proposito

🟢 **CONFIRMADO** — `src/discover/` tem duas missoes acopladas: classificar/reescrever comandos shell para equivalentes RTK no hot path dos hooks, e analisar historico de sessoes Claude Code para encontrar oportunidades perdidas de economia. O README declara que a logica de classificacao e compartilhada entre rewrite ao vivo e `rtk discover`.

### Arquivos analisados

- `src/discover/README.md`
- `src/discover/mod.rs`
- `src/discover/lexer.rs`
- `src/discover/provider.rs`
- `src/discover/registry.rs`
- `src/discover/report.rs`
- `src/discover/rules.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Implementar `rtk discover`: localizar JSONL de sessoes Claude Code, extrair comandos Bash, dividir comandos compostos, classificar comandos suportados/ignorados/nao suportados e estimar economia.
- 🟢 **CONFIRMADO** — Implementar o registro de rewrite usado por hooks: `rewrite_command()` normaliza, divide, reescreve segmentos e retorna `None` quando nada mudou.
- 🟢 **CONFIRMADO** — Fornecer lexer shell-aware para aspas, escapes, operadores, pipes, redirects, shellisms e offsets.
- 🟢 **CONFIRMADO** — Expor primitivas de seguranca para permissoes: `contains_unattestable_construct()` e `split_for_permissions()`.
- 🟢 **CONFIRMADO** — Manter `rules.rs` com 86 regras de rewrite cobrindo Git, JS/TS, Python, Go, Ruby, PHP, Infra, Build, Files, Network, System e package managers.
- 🟢 **CONFIRMADO** — Detectar integracoes de agentes relevantes no relatorio (`Cursor`, `Hermes`, `Copilot`) e orientar que esses fluxos sao medidos por `rtk gain`, nao por scan de JSONL Claude.

### Fluxo de controle

#### `rtk discover`

1. 🟢 **CONFIRMADO** — `discover::run(project, all, since_days, limit, format, verbose)` cria `ClaudeProvider`.
2. 🟢 **CONFIRMADO** — Se `--all` nao estiver ativo, usa projeto informado ou codifica o `cwd` no formato de diretorio do Claude Code.
3. 🟢 **CONFIRMADO** — Descobre arquivos `.jsonl` sob `~/.claude/projects`, filtrando por projeto e mtime quando aplicavel.
4. 🟢 **CONFIRMADO** — Para cada comando extraido, usa `split_command_chain()` para separar `&&`, `||`, `;` e parar no pipe quando necessario.
5. 🟢 **CONFIRMADO** — Antes de classificar, detecta prefixo `RTK_DISABLED=` e so contabiliza bypass quando o comando subjacente seria suportado.
6. 🟢 **CONFIRMADO** — Comandos `Supported` entram em buckets por `rtk_equivalent`, acumulando count, exemplo mais frequente, tokens brutos e tokens economizaveis.
7. 🟢 **CONFIRMADO** — Comandos `Unsupported` entram em buckets por base command; `Ignored` incrementa `already_rtk` apenas quando comeca com `rtk `.
8. 🟢 **CONFIRMADO** — O relatorio ordena suportados por economia estimada, unsupported por frequencia, exemplos `RTK_DISABLED` por frequencia e renderiza texto ou JSON.

#### Provider Claude

🟢 **CONFIRMADO** — `ClaudeProvider::discover_sessions()` resolve o diretorio Claude via `hooks::init::resolve_claude_dir()`, procura recursivamente `.jsonl`, nao segue symlinks e aplica filtro de projeto por substring no nome de pasta codificado.

🟢 **CONFIRMADO** — `extract_commands()` faz uma passagem por linhas JSONL, prefiltra por `"Bash"` ou `"tool_result"`, coleta `tool_use` Bash com id/command/sequence e casa com `tool_result` para obter tamanho do output, preview de ate 1000 chars e flag `is_error`.

#### Classificacao

🟢 **CONFIRMADO** — `classify_command()` ignora comandos vazios, exatos e prefixos conhecidos, remove prefixos env/sudo/env, normaliza path absoluto (`/usr/bin/grep`), remove opcoes globais Git, normaliza ferramentas PHP Composer e opcoes globais `golangci-lint`.

🟢 **CONFIRMADO** — `RegexSet` seleciona os matches em `RULES` e usa o ultimo match como mais especifico. Depois captura subcomando para sobrescrever percentual de economia e status (`Existing`, `Passthrough`, `NotSupported`).

🟢 **CONFIRMADO** — `cat/head/tail` com redirect de escrita deixam de ser `Supported`; `cat` com flags alem de `-n` nao e reescrito.

#### Rewrite

🟢 **CONFIRMADO** — `rewrite_command()` colapsa continuacoes Bash `\` + newline, rejeita heredoc e aritmetica `$((...))`, compila exclusoes, normaliza prefixos transparentes e passa comandos compostos para `rewrite_compound()`.

🟢 **CONFIRMADO** — Comando simples que ja comeca com `rtk` retorna `Some(trimmed)`. Em comando composto que inicia com `rtk`, continua processando os segmentos seguintes.

🟢 **CONFIRMADO** — `rewrite_compound()` reescreve cada segmento separado por `&&`, `||`, `;` e background `&`. Em pipes, reescreve apenas o lado esquerdo; `find` e `fd` antes de pipe sao preservados crus por incompatibilidade com consumidores como `xargs`.

🟢 **CONFIRMADO** — `rewrite_segment_inner()` aplica recursivamente prefixos env, prefixos built-in (`noglob`, `command`, `builtin`, `exec`, `nocorrect`) e prefixos transparentes configurados, com `MAX_PREFIX_DEPTH = 10`.

🟢 **CONFIRMADO** — Redirects finais sao removidos antes do match e reaplicados depois; `head`/`tail` de linha simples viram `rtk read --max-lines/--tail-lines`; `gh` com `--json`, `--jq` ou `--template` e pulado para nao corromper output estruturado.

🟢 **CONFIRMADO** — Comandos nao cobertos por `RULES` podem ser reescritos via filtros TOML customizados, desde que TOML nao esteja desabilitado, o comando nao seja reservado RTK e `command_matches_filter()` retorne true.

### Algoritmos e regras relevantes

#### Lexer shell-aware

🟢 **CONFIRMADO** — `tokenize()` preserva aspas, escapes e offsets byte-based, classificando tokens em `Arg`, `Operator`, `Pipe`, `Redirect` e `Shellism`.

🟢 **CONFIRMADO** — Variaveis simples `$VAR` sao tratadas como `Arg`, enquanto substituicoes, brace expansion, glob, backticks, subshells e background sao `Shellism`.

🟢 **CONFIRMADO** — `contains_unattestable_construct()` marca como inseguro comando/process substitution e redirects com alvo de arquivo; fd-dup/close (`2>&1`, `>&2`, `2>&-`) e `/dev/null` sao permitidos.

#### Estimativa de economia

🟢 **CONFIRMADO** — Quando `tool_result` tem tamanho real, `rtk discover` estima tokens por `len / 4`; se nao houver output, usa medias por categoria/subcomando em `category_avg_tokens()`.

🟢 **CONFIRMADO** — Percentual efetivo por bucket e media ponderada: `total_output_tokens / total_raw_output_tokens * 100`.

#### Regras de exclusao e bypass

🟢 **CONFIRMADO** — `exclude_commands` aceita regex ancorada ou prefixo literal; padroes vazios/triviais sao ignorados com warning, regex invalida cai para prefixo.

🟢 **CONFIRMADO** — `RTK_DISABLED=` no prefixo env bloqueia rewrite e emite warning, preservando comando cru.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `SupportedBucket`, `UnsupportedBucket`
- `ParsedToken`, `TokenKind`
- `ExtractedCommand`, `SessionProvider`, `ClaudeProvider`
- `Classification`
- `RtkRule`
- `RtkStatus`, `SupportedEntry`, `UnsupportedEntry`, `DiscoverReport`, `AgentIntegrationStatus`

### Dependencias internas

🟢 **CONFIRMADO** — `discover` depende de `hooks::init` para localizar Claude dir, de `hooks::constants` para detectar integracoes, de `core::utils` para composer bin dirs e de `core::toml_filter` para rewrite por filtros TOML. `hooks` e `analytics` tambem consomem `discover`, criando acoplamento bidirecional funcional entre hook/rewrite/analytics.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Diretorio Claude projects ausente retorna lista vazia; arquivo projects existente mas nao diretorio gera erro contextualizado.
- 🟢 **CONFIRMADO** — Linhas JSONL malformadas sao ignoradas; linhas sem Bash/tool_result sao prefiltradas.
- 🟢 **CONFIRMADO** — Falha ao extrair uma sessao incrementa `parse_errors` e nao aborta todo `rtk discover`.
- 🟢 **CONFIRMADO** — Regras de rewrite inseguras retornam `None`, deixando o hook passar o comando cru.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `discover` tem cobertura ampla: lexer, redirects, shellisms, split de operadores/permissoes, provider JSONL, codificacao de projeto Claude, formatacao de relatorio, integracoes detectadas, classificacao por regra, rewrite composto, heredoc, command substitution, exclusoes, `gh --json`, paths absolutos, opcoes globais Git/golangci, prefixos transparentes e normalizacao PHP/Composer.

### Complexidade

🟢 **CONFIRMADO** — Complexidade alta. O modulo combina parser shell parcial, motor de regras regex, seguranca de rewrite, heuristicas de economia, leitura de historico Claude e contratos consumidos por hooks/permissoes/analytics.

### Lacunas

- 🔴 **LACUNA** — O lexer e deliberadamente parcial: cobre os casos relevantes ao RTK, mas nao e um parser Bash completo.
- 🔴 **LACUNA** — Percentuais de economia em `rules.rs` sao heuristicas estaticas e devem ser validados contra dados reais.
- 🔴 **LACUNA** — A analise estatica nao executou um scan real de `~/.claude/projects` nem validou comportamento com sessoes grandes/corrompidas em massa.

## Modulo `learn`

### Proposito

🟢 **CONFIRMADO** — `src/learn/` analisa historico Claude Code para detectar erros de CLI recorrentes que foram corrigidos pelo agente em comandos subsequentes. Ele alimenta o comando `rtk learn` e pode gerar regras Markdown em `.claude/rules/cli-corrections.md`.

### Arquivos analisados

- `src/learn/README.md`
- `src/learn/mod.rs`
- `src/learn/detector.rs`
- `src/learn/report.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Reutilizar `discover::provider::ClaudeProvider` para descobrir sessoes e extrair comandos Bash com output.
- 🟢 **CONFIRMADO** — Identificar comandos com erro real, excluindo rejeicoes/cancelamentos do usuario.
- 🟢 **CONFIRMADO** — Classificar erros em `UnknownFlag`, `CommandNotFound`, `WrongSyntax`, `WrongPath`, `MissingArg`, `PermissionDenied` ou `Other`.
- 🟢 **CONFIRMADO** — Procurar uma correcao nos proximos 3 comandos com mesma base e similaridade suficiente.
- 🟢 **CONFIRMADO** — Deduplicar pares em regras por `(base_command, error_type, diff_token)`, contando ocorrencias.
- 🟢 **CONFIRMADO** — Renderizar relatorio em texto, JSON ou arquivo Markdown de regras quando `--write-rules` e usado.

### Fluxo de controle

#### `rtk learn`

1. 🟢 **CONFIRMADO** — `learn::run()` recebe filtros de projeto, periodo, formato, escrita de regras e limiares de confianca/ocorrencias.
2. 🟢 **CONFIRMADO** — Resolve filtro de projeto com a mesma logica do `discover`: `--all`, projeto explicito ou slug do `current_dir()`.
3. 🟢 **CONFIRMADO** — Se nao houver sessoes Claude Code no periodo, imprime mensagem e encerra com sucesso.
4. 🟢 **CONFIRMADO** — Para cada sessao, extrai comandos; sessoes malformadas sao ignoradas.
5. 🟢 **CONFIRMADO** — Apenas comandos com `output_content` entram em `CommandExecution`.
6. 🟢 **CONFIRMADO** — `find_corrections()` detecta pares erro-correcao; depois aplica filtro `min_confidence`, deduplicacao e filtro `min_occurrences`.
7. 🟢 **CONFIRMADO** — Em `format=json`, imprime `sessions_scanned`, `total_corrections` e rules serializadas.
8. 🟢 **CONFIRMADO** — Em formato texto, chama `format_console_report()` e, se solicitado, grava `.claude/rules/cli-corrections.md`.

#### Detector

🟢 **CONFIRMADO** — `is_command_error(is_error, output)` exige `is_error=true`, rejeita padroes de cancelamento/rejeicao do usuario e so aceita conteudo com indicadores como `error`, `failed`, `unknown`, `invalid`, `not found`, `permission denied` ou `cannot`.

🟢 **CONFIRMADO** — `classify_error()` aplica regexes lazy_static para unknown flag, command not found, missing arg, permission denied e wrong path; fallback e `Other("General Error")`.

🟢 **CONFIRMADO** — `find_corrections()` varre cada comando com erro, pula erros de ciclo TDD/compilacao/testes, olha ate 3 comandos a frente, exige similaridade >= 0.5, descarta diferenca apenas de path e repeticao identica, soma boost de 0.2 quando candidato nao e erro e exige confianca >= 0.6.

🟢 **CONFIRMADO** — `command_similarity()` compara base command; bases diferentes retornam 0.0. Bases iguais recebem 0.5 de base e ate 0.5 por Jaccard similarity dos argumentos.

### Algoritmos e regras relevantes

#### Deteccao de correcao

🟢 **CONFIRMADO** — A janela fixa `CORRECTION_WINDOW = 3` define o alcance de busca da correcao apos um erro.

🟢 **CONFIRMADO** — `MIN_CONFIDENCE = 0.6` e hardcoded no detector, enquanto o CLI tambem recebe `min_confidence` para filtrar os pares detectados.

🟢 **CONFIRMADO** — `extract_base_command()` usa os primeiros 1-2 tokens apos remover alguns env prefixes comuns (`RUST_BACKTRACE=1`, `NODE_ENV=production`, `DEBUG=*`).

🟡 **INFERIDO** — A ordenacao cronologica global entre multiplas sessoes nao e estritamente garantida; o comentario afirma que a ordem de extracao dentro de cada sessao ja e preservada, mas nao ha sort entre arquivos.

#### Escrita de regras

🟢 **CONFIRMADO** — `write_rules_file()` cria diretorios pais, agrupa regras por `base_command`, ordena os grupos alfabeticamente e escreve bullets `Use right not wrong`, incluindo `(seen Nx)` quando recorrente.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `ErrorType`
- `CorrectionPair`
- `CorrectionRule`
- `CommandExecution`

### Dependencias internas

🟢 **CONFIRMADO** — `learn` depende de `discover::provider::{ClaudeProvider, SessionProvider}` para sessoes e de `regex`/`lazy_static` para classificacao de erros. E roteado por `main` como comando analitico/meta.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Sessao malformada e ignorada sem abortar o comando.
- 🟢 **CONFIRMADO** — Ausencia de sessoes ou ausencia de correcoes gera mensagem amigavel e `Ok(())`.
- 🟢 **CONFIRMADO** — Escrita de arquivo de regras propaga erro via `anyhow::Result`.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `learn` contem testes para filtragem de erro real, rejeicao de usuario, classificacao de tipos de erro, formatacao de relatorio e escrita Markdown de regras.

### Complexidade

🟢 **CONFIRMADO** — Complexidade media. O modulo e menor que `discover`, mas tem heuristicas sensiveis para nao confundir exploracao normal, TDD e cancelamentos humanos com correcoes de CLI.

### Lacunas

- 🔴 **LACUNA** — A analise estatica nao validou a qualidade das heuristicas em historico real.
- 🔴 **LACUNA** — `MIN_CONFIDENCE` interno e filtro CLI podem causar dupla filtragem com semantica pouco obvia.
- 🔴 **LACUNA** — A escrita de `.claude/rules/cli-corrections.md` sobrescreve o arquivo alvo quando usada pelo produto; isso deve ser avaliado no contexto da regra Reversa, embora esta analise nao execute `--write-rules`.

## Modulo `parser`

### Proposito

🟢 **CONFIRMADO** — `src/parser/` fornece a infraestrutura canonica de parsing e formatacao token-efficient para saidas de ferramentas, com degradacao explicita em tres niveis para evitar dados falsos silenciosos.

### Arquivos analisados

- `src/parser/README.md`
- `src/parser/mod.rs`
- `src/parser/formatter.rs`
- `src/parser/types.rs`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Definir `ParseResult<T>` com tiers `Full`, `Degraded` e `Passthrough`.
- 🟢 **CONFIRMADO** — Definir o trait `OutputParser`, que padroniza parsers de ferramentas em `parse(input) -> ParseResult<Self::Output>`.
- 🟢 **CONFIRMADO** — Truncar passthrough por limite configurado em `core::config::limits().passthrough_max_chars`.
- 🟢 **CONFIRMADO** — Extrair objeto JSON completo de saidas com prefixos nao JSON, preservando strings, escapes e braces aninhados.
- 🟢 **CONFIRMADO** — Definir tipos canonicos para resultados de teste e estado de dependencias.
- 🟢 **CONFIRMADO** — Formatar `TestResult` e `DependencyState` em modos `Compact`, `Verbose` e `Ultra`.

### Fluxo de controle

#### Parsing de ferramenta

1. 🟢 **CONFIRMADO** — Implementadores de `OutputParser` recebem output bruto e tentam produzir dado estruturado.
2. 🟢 **CONFIRMADO** — O contrato documentado prioriza JSON completo como Tier 1, fallback parcial como Tier 2 e passthrough truncado como Tier 3.
3. 🟢 **CONFIRMADO** — `parse_with_tier(input, max_tier)` chama `parse()` e, se o tier real exceder o limite permitido, forca `ParseResult::Passthrough(truncate_passthrough(input))`.
4. 🟢 **CONFIRMADO** — Consumidores podem usar `tier()`, `is_ok()`, `warnings()` e `map()` para preservar a semantica de degradacao ao transformar o dado.

#### Truncamento e warnings

🟢 **CONFIRMADO** — `truncate_output()` conta por `char`, nao por byte, evitando cortar UTF-8 no meio. Quando excede o limite, anexa marcador `[RTK:PASSTHROUGH] Output truncated (...)`.

🟢 **CONFIRMADO** — `emit_degradation_warning()` e `emit_passthrough_warning()` escrevem marcadores padronizados em stderr.

#### Extracao JSON tolerante a prefixos

🟢 **CONFIRMADO** — `extract_json_object()` procura primeiro `"numTotalTests"` como marcador Vitest e retrocede ate `{`. Sem esse marcador, procura uma linha cujo `trim()` comece com `{`.

🟢 **CONFIRMADO** — Depois do ponto inicial, a funcao balanceia braces com offsets de byte, controlando `in_string` e `escape_next`, e retorna o slice ate o fechamento do objeto completo.

### Algoritmos e regras relevantes

#### Formatacao de testes

🟢 **CONFIRMADO** — `TestResult::format_compact()` sempre mostra pass/fail, inclui skipped quando maior que zero, lista ate 5 falhas e preserva todas as linhas da mensagem de erro de cada falha listada.

🟢 **CONFIRMADO** — `format_verbose()` lista todas as falhas com arquivo e ate 3 linhas de stack trace.

🟢 **CONFIRMADO** — `format_ultra()` usa representacao simbolica curta `[ok]N [x]N [skip]N (Nms)`.

#### Formatacao de dependencias

🟢 **CONFIRMADO** — `DependencyState::format_compact()` diferencia listagem simples de pacotes de consulta de outdated. Quando todas as dependencias nao tem `latest_version`, ele lista pacotes em vez de afirmar "All packages up-to-date".

🟢 **CONFIRMADO** — Listagens simples sao limitadas por `CAP_INVENTORY`; outdated compact mostra ate 10 dependencias com `current -> latest`.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/enums principais:

- `ParseResult<T>`
- `OutputParser`
- `FormatMode`
- `TokenFormatter`
- `TestResult`
- `TestFailure`
- `DependencyState`
- `Dependency`

### Dependencias internas

🟢 **CONFIRMADO** — `parser` depende de `core::config` para limite de passthrough e de `core::truncate::CAP_INVENTORY` para limite de listagem de dependencias. Ele e consumido por comandos/filtros que precisam normalizar output de ferramentas em formatos compactos.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Falha de parse pode ser expressa como `Passthrough`, preservando output bruto truncado com marcador explicito.
- 🟢 **CONFIRMADO** — Parse parcial carrega warnings em `Degraded(T, Vec<String>)`.
- 🟢 **CONFIRMADO** — `unwrap()` panica quando chamado em `Passthrough`, deixando claro que o consumidor nao deve tratar passthrough como dado estruturado.

### Testes embutidos no modulo

🟢 **CONFIRMADO** — `parser` testa tiers de `ParseResult`, `map()`, truncamento com ASCII, Thai e emoji, extracao JSON limpa, com prefixos pnpm/dotenv/CJK, braces aninhados, strings com braces e valores CJK/emoji.

🟢 **CONFIRMADO** — `formatter` testa que listagem simples de dependencias nao vira falso "up-to-date" e que `format_compact()` preserva detalhes importantes de erro de testes.

### Complexidade

🟢 **CONFIRMADO** — Complexidade media. O modulo e pequeno, mas concentra contratos transversais sensiveis: degradacao sem falsos positivos, truncamento Unicode-safe e compressao de saida para diferentes ferramentas.

### Lacunas

- 🔴 **LACUNA** — O README cita tipos planejados como `LintResult` e `BuildOutput`, mas `types.rs` atualmente define apenas `TestResult` e `DependencyState`.
- 🔴 **LACUNA** — `OutputParser` define contrato, mas nao ha implementadores concretos dentro de `src/parser/`; a migracao de parsers por ferramenta aparece como roadmap.
- 🔴 **LACUNA** — `extract_json_object()` privilegia marcador Vitest (`numTotalTests`), portanto outros JSONs embutidos dependem da heuristica de linha iniciando com `{`.

## Modulo `filters`

### Proposito

🟢 **CONFIRMADO** — `src/filters/` e o catalogo declarativo de filtros TOML embutidos do RTK. Cada arquivo define um filtro por comando/subcomando e seus testes inline, que sao concatenados por `build.rs` e consumidos em runtime por `core::toml_filter`.

### Arquivos analisados

- `src/filters/README.md`
- `src/filters/*.toml` (63 filtros embutidos)
- `.rtk/filters.toml` (template project-local sem filtros ativos)
- `build.rs` (pipeline de concatenacao/validacao)
- `src/core/toml_filter.rs` (runtime consumidor do catalogo)

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Declarar filtros para comandos com output textual previsivel, removendo ruido linha-a-linha sem reformatar a saida para algo que deixe de parecer output real.
- 🟢 **CONFIRMADO** — Cobrir install/update logs, monorepos, linters/typecheckers, infra/IaC, build tools, sistema operacional, cloud/devops e utilitarios.
- 🟢 **CONFIRMADO** — Fornecer testes inline por filtro via `[[tests.<filter-name>]]`; foram encontrados 154 blocos de teste.
- 🟢 **CONFIRMADO** — Permitir overrides project-local/user-global, mas custom filters so entram no runtime quando passam pelo gate de trust.
- 🟢 **CONFIRMADO** — Manter filtros built-in sempre trusted por estarem embutidos no binario.

### Fluxo de controle

#### Build dos filtros built-in

1. 🟢 **CONFIRMADO** — `build.rs` le `src/filters`, coleta arquivos `.toml` e ordena alfabeticamente.
2. 🟢 **CONFIRMADO** — O build injeta `schema_version = 1`, concatena cada arquivo com comentario de origem e valida o TOML combinado.
3. 🟢 **CONFIRMADO** — O build verifica duplicidade de nomes sob `[filters]`.
4. 🟢 **CONFIRMADO** — O resultado e escrito em `OUT_DIR/builtin_filters.toml` e embutido por `include_str!`.

#### Lookup em runtime

🟢 **CONFIRMADO** — A prioridade documentada e first-match-wins: `.rtk/filters.toml`, `~/.config/rtk/filters.toml`, built-ins e, sem match, passthrough pelo caller.

🟢 **CONFIRMADO** — Caminhos project/global sao carregados por `hooks::trust::gated_filter_paths()` e so sao parseados quando `check_trust_with_content()` retorna trusted/env override.

🟢 **CONFIRMADO** — `RTK_NO_TOML=1` desabilita o motor TOML e `RTK_TOML_DEBUG=1` habilita logs de match/contagem de linhas.

#### Pipeline de aplicacao

🟢 **CONFIRMADO** — `apply_filter_with_info()` aplica oito estagios em ordem: `strip_ansi`, `replace`, `match_output`, `strip/keep_lines`, `truncate_lines_at`, `head/tail_lines`, `max_lines`, `on_empty`.

🟢 **CONFIRMADO** — `match_output` e short-circuit de blob inteiro; a regra pode ter `unless` para nao engolir erros/warnings.

🟢 **CONFIRMADO** — `strip_lines_matching` e `keep_lines_matching` sao mutuamente exclusivos no modelo compilado via `LineFilter`.

### Catalogo observado

🟢 **CONFIRMADO** — Foram encontrados 63 filtros built-in. Exemplos por familia:

- Build/test/dev tools: `dotnet-build`, `gcc`, `gradle`, `make`, `swift-build`, `trunk-build`, `xcodebuild`, `pio-run`, `spring-boot`.
- JS/monorepo/task runners: `biome`, `nx`, `turbo`, `just`, `task`, `mise`, `oxlint`.
- Package managers/installers: `brew-install`, `bundle-install`, `composer-install`, `poetry-install`, `uv-sync`.
- IaC/cloud/devops: `terraform-plan`, `tofu-*`, `pulumi-*`, `helm`, `gcloud`, `skopeo`, `rsync`, `ssh`.
- Linters/typecheckers: `basedpyright`, `ty`, `shellcheck`, `yamllint`, `hadolint`, `markdownlint`.
- Sistema/utilitarios: `df`, `du`, `ps`, `stat`, `systemctl-status`, `ping`, `iptables`, `fail2ban-client`, `jq`.

🟢 **CONFIRMADO** — `.rtk/filters.toml` contem apenas template comentado e `schema_version = 1`; nao ha filtros project-local ativos neste repositorio no momento da analise.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/configuracoes principais:

- `TomlFilterFile`
- `TomlFilterDef`
- `MatchOutputRule`
- `ReplaceRule`
- `TomlFilterTestDef`
- `CompiledFilter`
- `LineFilter`
- `TestOutcome`
- `VerifyResults`

### Dependencias internas

🟢 **CONFIRMADO** — `filters` e um modulo declarativo consumido por `core::toml_filter` e empacotado por `build.rs`. Trust de filtros customizados depende de `hooks::trust`. O runtime usa `regex`, `RegexSet`, `toml`, `serde` e utilitarios de `core::utils`.

### Tratamento de erros

- 🟢 **CONFIRMADO** — TOML invalido em built-ins falha no build; TOML invalido em runtime emite warning e nao derruba o processo.
- 🟢 **CONFIRMADO** — `schema_version` diferente de 1 e rejeitado.
- 🟢 **CONFIRMADO** — Regex invalida em filtro individual gera warning daquele filtro.
- 🟢 **CONFIRMADO** — Filtros customizados untrusted ou com conteudo alterado sao ignorados no hot path.

### Testes embutidos/guardrails

🟢 **CONFIRMADO** — `core::toml_filter` testa que os built-ins compilam, que ha exatamente 63 filtros embutidos e que todo filtro built-in tem pelo menos um teste inline.

🟢 **CONFIRMADO** — Ha teste de prioridade project-local sobre built-in e teste de descobribilidade de novos filtros apos concatenacao.

### Complexidade

🟢 **CONFIRMADO** — Complexidade media-alta. Os arquivos TOML sao simples individualmente, mas o conjunto e grande e tem semantica de seguranca/trust, prioridade de override, short-circuit de output inteiro e politicas de perda de informacao.

### Lacunas

- 🔴 **LACUNA** — A analise nao executou `cargo test`; a validacao aqui e estatica e por contagem/estrutura.
- 🔴 **LACUNA** — O teste `test_builtin_all_expected_filters_present` lista apenas um subconjunto historico dos 63 filtros, embora `test_builtin_filter_count` cubra a contagem total.
- 🔴 **LACUNA** — A qualidade semantica de economia de tokens por filtro depende dos fixtures inline; nao foi medida contra outputs reais recentes de cada ferramenta.

## Modulo `openclaw`

### Proposito

🟢 **CONFIRMADO** — `openclaw/` implementa um plugin TypeScript fino para OpenClaw que intercepta chamadas da ferramenta `exec` e delega a decisao de rewrite para o binario `rtk rewrite`.

### Arquivos analisados

- `openclaw/index.ts`
- `openclaw/openclaw.plugin.json`
- `openclaw/package.json`
- `openclaw/README.md`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Registrar hook `before_tool_call` com prioridade 10.
- 🟢 **CONFIRMADO** — Interceptar apenas tool calls cujo `toolName` e `exec` e cujo `params.command` e string.
- 🟢 **CONFIRMADO** — Verificar uma vez se `rtk` esta disponivel no `PATH` via `which rtk`, cacheando o resultado.
- 🟢 **CONFIRMADO** — Executar `rtk rewrite <command>` com timeout de 2000 ms.
- 🟢 **CONFIRMADO** — Aplicar rewrite automatico, bloquear comando ou exigir aprovacao conforme exit code do `rtk rewrite`.
- 🟢 **CONFIRMADO** — Expor configuracao `enabled` e `verbose` no manifesto OpenClaw.

### Fluxo de controle

#### Registro do plugin

1. 🟢 **CONFIRMADO** — `register(api)` le `api.config`.
2. 🟢 **CONFIRMADO** — Se `enabled === false`, retorna sem registrar hook.
3. 🟢 **CONFIRMADO** — Se `checkRtk()` falha, emite warning e desativa o plugin.
4. 🟢 **CONFIRMADO** — Registra `api.on("before_tool_call", handler, { priority: 10 })`.

#### Handler de ferramenta

🟢 **CONFIRMADO** — O handler ignora tudo que nao seja `exec` ou que nao tenha `params.command` string.

🟢 **CONFIRMADO** — Para comando elegivel, chama `tryRewrite(command)`.

🟢 **CONFIRMADO** — Se o verdict for `deny`, retorna `{ block: true, blockReason: "RTK deny rule matched" }`.

🟢 **CONFIRMADO** — Se nao houver rewrite, retorna `undefined`, preservando o comando original.

🟢 **CONFIRMADO** — Se houver rewrite, retorna `params` clonado com `command` substituido.

🟢 **CONFIRMADO** — Para verdict `ask`, adiciona `requireApproval` com titulo, descricao, severidade `info`, `timeoutBehavior: "deny"` e decisoes `allow-once`/`deny`.

### Protocolo de exit code

🟢 **CONFIRMADO** — O protocolo documentado em `index.ts` e:

- `0 + stdout`: rewrite permitido e aplicado automaticamente.
- `1`: sem equivalente RTK; passthrough.
- `2`: deny rule; bloqueia a tool call.
- `3 + stdout`: rewrite disponivel, mas exige aprovacao humana.

🟢 **CONFIRMADO** — Exit code desconhecido, exit `1` ou exit `3` sem stdout util sao tratados como passthrough.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/configuracoes principais:

- `RewriteVerdict`
- Tupla de retorno de `tryRewrite`
- `requireApproval`
- `configSchema.enabled`
- `configSchema.verbose`

### Dependencias internas e externas

🟢 **CONFIRMADO** — O plugin depende de `node:child_process` e do binario `rtk` no `PATH`. A logica real de rewrite permanece no Rust, em especial no fluxo `rtk rewrite`/`src/hooks/rewrite_cmd.rs`/`src/discover/registry.rs`.

### Tratamento de erros

- 🟢 **CONFIRMADO** — Ausencia do binario `rtk` desativa o plugin com warning.
- 🟢 **CONFIRMADO** — Falhas de `execFileSync("rtk", ["rewrite", command])` sao interpretadas por status code; status nao reconhecido cai para passthrough.
- 🟢 **CONFIRMADO** — Timeout de 2000 ms limita travamento do hook.

### Testes embutidos no modulo

🔴 **LACUNA** — Nao foram encontrados testes automatizados dentro de `openclaw/`.

### Complexidade

🟢 **CONFIRMADO** — Complexidade baixa. O modulo e intencionalmente fino, com maior risco concentrado na integracao com protocolo de exit code e API de aprovacao do OpenClaw.

### Lacunas

- 🔴 **LACUNA** — Nao ha tipos OpenClaw importados; `api` e `event` usam `any`/shape manual.
- 🔴 **LACUNA** — A analise estatica nao validou o comportamento real da API OpenClaw nem o suporte a `requireApproval`.
- 🔴 **LACUNA** — O plugin nao diferencia erro operacional do `rtk rewrite` de "sem rewrite"; ambos podem virar passthrough silencioso.

## Modulo `docs`

### Proposito

🟢 **CONFIRMADO** — `docs/` concentra a documentacao tecnica, funcional, de usuario, privacidade, analytics, troubleshooting e governanca do RTK. Ela estabelece contratos publicos importantes para comportamento do produto e para contribuidores.

### Arquivos analisados

- `docs/contributing/TECHNICAL.md`
- `docs/contributing/ARCHITECTURE.md`
- `docs/contributing/CODING_PRACTICES.md`
- `docs/usage/FEATURES.md`
- `docs/usage/TRACKING.md`
- `docs/usage/AUDIT_GUIDE.md`
- `docs/TELEMETRY.md`
- `docs/guide/**`
- `docs/maintainers/MAINTAINERS_APPLY.md`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Documentar a visao do RTK como proxy CLI que reduz output consumido por LLMs em 60-90%.
- 🟢 **CONFIRMADO** — Explicar o fluxo end-to-end: hook/agent, `rtk rewrite`, parser Clap, roteamento, filtros Rust/TOML, tracking SQLite e tee recovery.
- 🟢 **CONFIRMADO** — Definir a matriz de agentes suportados e os tres tiers de integracao: hook completo, plugin e rules file.
- 🟢 **CONFIRMADO** — Documentar instalacao, configuracao, telemetria, comandos otimizados, analytics (`gain`, `discover`, `session`) e troubleshooting.
- 🟢 **CONFIRMADO** — Definir praticas de contribuicao: baixo overhead, fallback seguro, testes no mesmo arquivo, fixtures reais e savings >= 60%.
- 🟢 **CONFIRMADO** — Documentar responsabilidades de maintainers por ecossistema e core.

### Fluxo documental do produto

#### Usuario final

1. 🟢 **CONFIRMADO** — `installation.md` alerta para colisao de nome com outro `rtk` e recomenda verificar com `rtk gain`.
2. 🟢 **CONFIRMADO** — `quick-start`/`index` encaminham instalacao, inicializacao do hook e medicao de savings.
3. 🟢 **CONFIRMADO** — `supported-agents.md` detalha como cada agente intercepta ou apenas recebe instrucoes.
4. 🟢 **CONFIRMADO** — `configuration.md` documenta `config.toml`, env vars, tee, exclusoes de rewrite, telemetria e trust de custom filters.
5. 🟢 **CONFIRMADO** — `troubleshooting.md` cobre pacote errado, PATH, hooks sem efeito, Windows e diagnostico.

#### Contribuidor

🟢 **CONFIRMADO** — `TECHNICAL.md` e a porta de entrada tecnica, com mapa de pastas, rewrite pipeline, fallback path, tracking, tee, testes e restricoes de performance.

🟢 **CONFIRMADO** — `CODING_PRACTICES.md` reforca portabilidade, extensibilidade, `anyhow::Result`, comentarios de "why", fixtures reais e evitar dependencias desnecessarias.

🟢 **CONFIRMADO** — `ARCHITECTURE.md` aprofunda command lifecycle, filtering strategies, tracking SQLite, flags globais, error handling, config e ADRs.

### Contratos e regras documentados

- 🟢 **CONFIRMADO** — RTK deve degradar graciosamente: falha de filtro ou hook cai para output bruto/passthrough.
- 🟢 **CONFIRMADO** — RTK deve preservar exit code dos comandos, especialmente para CI/CD.
- 🟢 **CONFIRMADO** — Hooks e plugins sao delegates finos; a decisao real de rewrite vive no binario Rust (`rtk rewrite`).
- 🟢 **CONFIRMADO** — Output filtrado deve parecer uma versao menor do output real, sem formato inventado que confunda o LLM.
- 🟢 **CONFIRMADO** — Startup alvo e `<10ms`, sem async runtime e com minimo de I/O no caminho critico.
- 🟢 **CONFIRMADO** — Tracking local usa SQLite com retencao padrao de 90 dias.
- 🟢 **CONFIRMADO** — Telemetria e desabilitada por padrao e exige consentimento explicito; nao coleta codigo, caminhos, argumentos completos, secrets ou PII.
- 🟢 **CONFIRMADO** — Custom filters sao trust-gated porque podem alterar o que o agente ve.

### Dominios cobertos pela documentacao

🟢 **CONFIRMADO** — `FEATURES.md` cataloga comandos de arquivos, Git, GitHub CLI, testes, build/lint, formatacao, package managers, containers/orquestracao, dados/rede, cloud/database, analytics, hooks, config, tee e telemetria.

🟢 **CONFIRMADO** — `what-rtk-covers.md` resume savings tipicos por ecossistema: Git, GitHub, Graphite, Cargo/Rust, JS/TS, Python, Go, Ruby, .NET, Docker/Kubernetes, arquivos/search e cloud/data.

🟢 **CONFIRMADO** — `gain.md` documenta daily/weekly/monthly breakdowns, JSON/CSV, quota estimates e token estimation por `text.len() / 4`.

🟢 **CONFIRMADO** — `discover.md` documenta analise de historico Claude Code para encontrar oportunidades perdidas e `session` para medir cobertura de uso RTK.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/contratos principais documentados:

- Config TOML: `[tracking]`, `[display]`, `[filters]`, `[tee]`, `[telemetry]`, `[hooks]`
- Tracking API: `Tracker`, `GainSummary`, `DayStats`, `WeekStats`, `MonthStats`, `CommandRecord`, `TimedExecution`
- Telemetry payload: identity anonima, environment, usage volume, quality, ecosystem, retention, economics, adoption, config, feature adoption
- Agent support matrix: full hook, plugin, rules file

### Dependencias internas

🟢 **CONFIRMADO** — `docs` referencia e sincroniza contratos com `src/core`, `src/hooks`, `src/analytics`, `src/cmds`, `src/discover`, `src/learn`, `src/parser`, `src/filters`, `hooks/` e `openclaw/`.

### Tratamento de erros e privacidade documentados

- 🟢 **CONFIRMADO** — Hooks devem falhar abertos: RTK ausente, JSON invalido, versao antiga ou erro de filtro preservam comando bruto.
- 🟢 **CONFIRMADO** — Tee recovery salva output bruto apenas para permitir recuperacao sem reexecutar comando.
- 🟢 **CONFIRMADO** — Telemetria usa ping diario em background, timeout de 2s, sem retries/fila, e respeita `RTK_TELEMETRY_DISABLED=1`.
- 🟢 **CONFIRMADO** — `rtk telemetry forget` desabilita telemetria, apaga dados locais e solicita erasure server-side.

### Testes/documentacao de qualidade

🟢 **CONFIRMADO** — Docs de contribuicao exigem testes com fixtures reais, snapshots e assercao de economia minima de 60% para novos filtros.

🟢 **CONFIRMADO** — Documentacao de contribuicao diz que docs devem ser atualizadas para novos filtros, features e mudancas que afetem comportamento documentado.

### Complexidade

🟢 **CONFIRMADO** — Complexidade media. O modulo nao executa codigo, mas e transversal e contem contratos publicos que precisam permanecer sincronizados com implementacao, guias e promessas de privacidade.

### Lacunas

- 🔴 **LACUNA** — Ha inconsistencias aparentes de caminho/nome do banco entre docs: alguns trechos citam `tracking.db`, outros `history.db`.
- 🔴 **LACUNA** — `TECHNICAL.md` cita suporte a 7 agentes em uma secao, enquanto guias/README citam um conjunto maior de agentes.
- 🔴 **LACUNA** — `FEATURES.md` esta em frances enquanto varios guias estao em ingles, indicando estrategia multilíngue parcial ou documento legado.

## Modulo `scripts`

### Proposito

🟢 **CONFIRMADO** — `scripts/` e arquivos auxiliares da raiz automatizam instalacao local/remota, diagnostico, smoke tests, testes de tracking, benchmarks locais, benchmarks em VM Multipass, validacoes de docs/testes e analises economicas.

### Arquivos analisados

- `install.sh`
- `build.rs`
- `scripts/install-local.sh`
- `scripts/check-installation.sh`
- `scripts/test-all.sh`
- `scripts/test-tracking.sh`
- `scripts/check-test-presence.sh`
- `scripts/validate-docs.sh`
- `scripts/benchmark.sh`
- `scripts/benchmark/**`
- `scripts/benchmark-sessions/lib/runner.py`
- `scripts/rtk-economics.sh`
- `scripts/update-readme-metrics.sh`

### Responsabilidades principais

- 🟢 **CONFIRMADO** — Instalar binarios release por plataforma, com checksum SHA-256 e protecao contra path traversal no archive.
- 🟢 **CONFIRMADO** — Instalar build local de `target/release/rtk` em diretorio escolhido, reconstruindo quando fonte/Cargo estiverem mais novos.
- 🟢 **CONFIRMADO** — Diagnosticar instalacao local e distinguir Rust Token Killer de outro binario `rtk`.
- 🟢 **CONFIRMADO** — Rodar smoke tests de comandos RTK em ambiente local.
- 🟢 **CONFIRMADO** — Validar que novos `*_cmd.rs` modificados tenham `#[cfg(test)]`.
- 🟢 **CONFIRMADO** — Executar suite de integracao em VM Multipass com fases para build, qualidade, comandos built-in, filtros TOML, rewrite, exit codes, savings, pipes, edge cases, performance e concorrencia.
- 🟢 **CONFIRMADO** — Combinar dados de `ccusage` e `rtk gain` para relatorio economico.

### Fluxo de controle

#### Instalador remoto (`install.sh`)

1. 🟢 **CONFIRMADO** — Detecta OS (`Linux`/`Darwin`) e arquitetura (`x86_64`/`aarch64`).
2. 🟢 **CONFIRMADO** — Resolve versao pela redirect `/releases/latest`, com fallback para GitHub API; `RTK_VERSION` permite pin.
3. 🟢 **CONFIRMADO** — Monta target triple e baixa `rtk-<target>.tar.gz` e `checksums.txt`.
4. 🟢 **CONFIRMADO** — Verifica checksum com `sha256sum` ou `shasum -a 256`; `RTK_SKIP_CHECKSUM=1` permite bypass com warning.
5. 🟢 **CONFIRMADO** — Lista archive antes de extrair e rejeita caminhos absolutos ou componentes `..`.
6. 🟢 **CONFIRMADO** — Extrai em temp dir, move binario para `RTK_INSTALL_DIR` ou `~/.local/bin`, aplica `chmod +x` e alerta se PATH nao contem o diretorio.

#### Smoke tests locais

🟢 **CONFIRMADO** — `test-all.sh` exige `rtk` no PATH e execucao dentro de repo Git, contabiliza pass/fail/skip e cobre ajuda, arquivos, Git, GitHub CLI, Cargo, curl, npm/npx, pnpm, grep e muitos outros comandos.

🟢 **CONFIRMADO** — `test-tracking.sh` executa comandos otimizados, passthrough, gh opcional e stdin, verificando presenca em `rtk gain --history`.

#### Benchmark VM

🟢 **CONFIRMADO** — `scripts/benchmark/run.ts` usa Bun e Multipass, cria/reusa VM `rtk-test`, transfere source excluindo `target`, `.git`, `node_modules` e builds release.

🟢 **CONFIRMADO** — A suite gera relatorio e declara `READY FOR RELEASE` apenas quando nao ha falhas.

🟢 **CONFIRMADO** — Fases incluem qualidade cargo, comandos Rust built-in, filtros TOML, rewrite engine, exit code preservation, token savings, pipe compatibility, edge cases, performance com hyperfine/memoria e concorrencia 10x.

### Estruturas de dados

Ver detalhes em `data-dictionary.md`.

Entidades/contratos principais:

- `install.sh`: `OS`, `ARCH`, `TARGET`, `VERSION`, `DOWNLOAD_URL`, `CHECKSUMS_URL`, `INSTALL_DIR`
- Benchmark TS: `VmInfo`, `TestResult`, `TestStatus`, `BuildInfo`
- Smoke shell: contadores `PASS`, `FAIL`, `SKIP`, lista `FAILURES`

### Dependencias internas e externas

🟢 **CONFIRMADO** — Scripts dependem de ferramentas como `curl`, `tar`, `sha256sum`/`shasum`, `cargo`, `git`, `rtk`, `gh`, `jq`, `bc`, `numfmt`, `bun`, `multipass`, `hyperfine`, `ccusage` e ferramentas de ecossistema opcionais.

🟢 **CONFIRMADO** — `build.rs` tambem faz parte desta automacao: concatena `src/filters/*.toml`, valida TOML e nomes duplicados, e ajusta stack no Windows.

### Tratamento de erros e seguranca

- 🟢 **CONFIRMADO** — Instalador remoto usa `set -e`, falha para OS/arch desconhecidos, falha em checksum ausente/divergente e recusa archive inseguro.
- 🟢 **CONFIRMADO** — `check-test-presence.sh` sai com codigo 1 quando um `*_cmd.rs` modificado nao tem testes inline.
- 🟢 **CONFIRMADO** — Benchmark VM usa timeout por comando (`vmExec`) e timeout de cloud-init.
- 🟢 **CONFIRMADO** — Smoke tests preservam contadores e retornam numero de falhas.

### Testes embutidos/guardrails

🟢 **CONFIRMADO** — `check-test-presence.sh --self-test` cria arquivo temporario sem testes e valida que o guard detecta a ausencia.

🟢 **CONFIRMADO** — Benchmark TS possui helpers para expectativa de exit code exato ou `"any"`, medicao de savings e validacao de rewrite input -> expected output.

🟢 **CONFIRMADO** — `validate-docs.sh` checa que comandos Python/Go estejam mencionados no README e faz verificacao simples do hook `.claude/hooks/rtk-rewrite.sh` quando presente.

### Complexidade

🟢 **CONFIRMADO** — Complexidade alta. O modulo mistura shell portable, release/install security, CI guardrails, smoke tests locais, automacao de VM, provisionamento de ecossistemas e medicao economica.

### Lacunas

- 🔴 **LACUNA** — `update-readme-metrics.sh` e explicitamente placeholder: so verifica markers, nao atualiza metricas.
- 🔴 **LACUNA** — `check-installation.sh` contem referencias a "fork" e `feat/all-features`, possivelmente desatualizadas para o estado atual do produto.
- 🔴 **LACUNA** — `validate-docs.sh` procura `.claude/hooks/rtk-rewrite.sh`, mas o projeto atual centraliza hooks em `hooks/`; isso pode gerar warning mesmo com instalacao moderna.
- 🔴 **LACUNA** — A analise nao executou scripts pesados/destrutivos como instalador remoto, smoke suite completa ou benchmark VM.
