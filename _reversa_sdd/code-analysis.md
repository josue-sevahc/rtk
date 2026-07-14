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
