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
