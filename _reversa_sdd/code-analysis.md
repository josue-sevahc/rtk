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
