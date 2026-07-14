# Inventario do Projeto — Scout

Projeto: `rtk`
Gerado em: `2026-07-14T14:23:54Z`
Nivel de documentacao solicitado: `completo`

## Resumo Executivo

🟢 **CONFIRMADO** — `rtk` e um CLI Rust de alta performance para reduzir consumo de tokens de LLMs filtrando, comprimindo e resumindo saidas de comandos antes que cheguem ao contexto do agente.

🟢 **CONFIRMADO** — A aplicacao principal fica em `src/main.rs`, usa `clap` para roteamento de subcomandos e organiza a logica em modulos Rust por responsabilidade tecnica.

🟢 **CONFIRMADO** — O projeto tambem inclui documentação ampla, scripts de instalacao/teste, workflows GitHub Actions, hooks para agentes de IA e um plugin TypeScript para OpenClaw.

## Estrutura de Pastas

Pastas relevantes do produto:

- `src/` — codigo-fonte Rust principal.
- `src/analytics/` — dashboards e relatorios de economia/adocao.
- `src/cmds/` — wrappers e filtros por ecossistema/comando externo.
- `src/core/` — infraestrutura compartilhada de execucao, tracking, configuracao, filtros e streaming.
- `src/discover/` — descoberta de uso do RTK e classificacao de comandos.
- `src/filters/` — filtros declarativos TOML para comandos suportados.
- `src/hooks/` — instalacao, auditoria, verificacao e execucao de hooks para agentes.
- `src/learn/` — deteccao de correcoes recorrentes em sessoes de coding agents.
- `src/parser/` — infraestrutura de parsers estruturados com degradacao.
- `tests/` — testes de integracao Rust e fixtures de saida de ferramentas externas.
- `hooks/` — integracoes por agente/editor (`claude`, `codex`, `cursor`, `copilot`, `windsurf`, `cline`, `kilocode`, `hermes`, `opencode`, `pi`, `antigravity`).
- `docs/` — documentacao de usuario, mantenedores, uso e contribuicao.
- `scripts/` — scripts de benchmark, instalacao, testes e validacao.
- `openclaw/` — plugin OpenClaw em TypeScript.
- `Formula/` — formula Homebrew.

Pastas de ambiente/agentes detectadas, nao tratadas como nucleo do produto:

- `.agents/` — skills do Reversa instalados no workspace.
- `.claude/` — agentes, comandos, hooks, regras e skills auxiliares.
- `.rtk/` — configuracao local do RTK.

## Linguagens e Arquivos

Contagem excluindo `.git`, `.reversa`, `_reversa_sdd`, `target`, `.agents`, `.claude`, `node_modules`, `dist`, `build`, `coverage`:

| Extensao | Arquivos | Uso principal |
|---|---:|---|
| `.rs` | 126 | CLI principal, filtros, hooks, parsers e tracking |
| `.md` | 75 | Documentacao |
| `.toml` | 65 | Cargo e filtros declarativos |
| `.json` | 13 | Fixtures, manifests e configuracoes |
| `.sh` | 17 | Scripts de instalacao, teste e hooks |
| `.ts` | 9 | Plugin OpenClaw e scripts de benchmark |
| `.yml` | 7 | GitHub Actions e configs |
| `.py` | 4 | Scripts/fixtures de benchmark e hooks Hermes |
| `.java` | 5 | Fixtures JVM |
| `.xml` | 4 | Fixtures Maven |
| `.txt` | 25 | Fixtures de saida de comandos |
| `.yaml` | 2 | Configuracoes YAML |
| `.rb` | 1 | Formula/empacotamento Ruby/Homebrew |
| outros | 5 | lockfile, gzip, git attributes/ignore, executavel sem extensao |

🟢 **CONFIRMADO** — Linguagem principal: Rust.

## Modulos Identificados

| Modulo | Caminho | Papel |
|---|---|---|
| `main` | `src/main.rs` | Entrypoint CLI, definicao de comandos e dispatch |
| `cmds` | `src/cmds/` | Execucao/filtragem de comandos externos por ecossistema |
| `core` | `src/core/` | Runner, streaming, tracking, configuracao, filtros, truncamento e utilitarios |
| `hooks` | `src/hooks/` | Instalacao, rewrite, permissoes, auditoria e integridade de hooks |
| `analytics` | `src/analytics/` | Relatorios de ganho, economia e sessoes |
| `discover` | `src/discover/` | Descoberta de comandos/sessoes e recomendacoes |
| `learn` | `src/learn/` | Identificacao de erros recorrentes e regras de correcao |
| `parser` | `src/parser/` | Contratos de parsing e formatacao compacta |
| `filters` | `src/filters/` | 63 filtros TOML declarativos |
| `openclaw` | `openclaw/` | Plugin TypeScript para rewrite de comandos |
| `docs` | `docs/` | Documentacao publica e de manutencao |
| `hooks-assets` | `hooks/` | Artefatos por agente/editor |
| `scripts` | `scripts/` | Automacao de testes, instalacao, benchmark e releases |

## Pontos de Entrada

🟢 **CONFIRMADO**:

- `src/main.rs` — binario Rust `rtk`; define `Cli`, `Commands` e roteamento principal.
- `build.rs` — script de build do Cargo.
- `openclaw/index.ts` — entrada do plugin OpenClaw.
- `install.sh` — instalador shell.
- `Formula/rtk.rb` — empacotamento Homebrew.

Subcomandos principais expostos por `src/main.rs`:

`ls`, `tree`, `read`, `smart`, `git`, `gh`, `glab`, `aws`, `psql`, `pnpm`, `err`, `test`, `json`, `deps`, `env`, `find`, `diff`, `log`, `dotnet`, `docker`, `kubectl`, `oc`, `summary`, `grep`, `rg`, `init`, `wget`, `wc`, `gain`, `cc-economics`, `config`, `jest`, `vitest`, `prisma`, `tsc`, `next`, `lint`, `prettier`, `format`, `playwright`, `cargo`, `npm`, `npx`, `curl`, `discover`, `session`, `telemetry`, `learn`, `run`, `proxy`, `pipe`, `trust`, `untrust`, `verify`, `ruff`, `pytest`, `mypy`, `php`, `phpunit`, `phpstan`, `pest`, `paratest`, `ecs`, `pint`, `rake`, `rubocop`, `rspec`, `pip`, `uv`, `go`, `gt`, `golangci-lint`, `gradlew`, `mvn`, `hook-audit`, `rewrite`, `hook`.

## Configuracoes e CI/CD

Arquivos de configuracao detectados:

- `Cargo.toml`
- `Cargo.lock`
- `.semgrep.yml`
- `.rtk/filters.toml`
- `release-please-config.json`
- `.release-please-manifest.json`
- `openclaw/package.json`
- `openclaw/openclaw.plugin.json`

Workflows GitHub Actions:

- `.github/workflows/ci.yml` — fmt, clippy, testes multi-OS, seguranca, semgrep, benchmark e documentacao.
- `.github/workflows/cd.yml` — pre-release em `develop` e release estavel em `master`.
- `.github/workflows/release.yml` — build multi-plataforma e artefatos de release.
- `.github/workflows/next-release.yml` — atualizacao automatica de PR de proxima release.
- `.github/workflows/pr-target-check.yml` — validacao de branch alvo de PR.
- `.github/workflows/CICD.md` — documentacao dos fluxos.

Docker:

- Nenhum `Dockerfile` ou `docker-compose.yml` detectado.

## Banco de Dados

🟡 **INFERIDO** — Nao ha migrations, DDL ou schema ORM detectados. O projeto usa `rusqlite` e menciona banco SQLite de tracking nos modulos `core/tracking` e `analytics`; a analise detalhada fica para `reversa-data-master` se necessario.

## Testes

🟢 **CONFIRMADO**:

- Framework principal: `cargo test` / testes Rust nativos.
- Testes Rust de integracao detectados: 6 arquivos em `tests/*.rs`.
- Fixtures extensas em `tests/fixtures/` para Maven, Gradle, GitLab, .NET, PHPStan, AWS, OpenShift, Go, UV/Pytest e outros.
- Scripts auxiliares de teste: `scripts/test-all.sh`, `scripts/test-install.sh`, `scripts/test-ruby.sh`, `scripts/test-tracking.sh`, `scripts/check-test-presence.sh`.
- Testes de hooks adicionais: `hooks/hermes/tests/test_rtk_rewrite_plugin.py`, `hooks/claude/test-rtk-rewrite.sh`, `hooks/copilot/test-rtk-rewrite.sh`.

## Integracoes Externas Detectadas

🟢 **CONFIRMADO** — O CLI atua como proxy/filtro para muitos ecossistemas externos:

- VCS/plataformas: `git`, `gh`, `glab`, `gt`.
- Build/test/lint: `cargo`, `mvn`, `gradle`, `go`, `golangci-lint`, `dotnet`, `npm`, `pnpm`, `npx`, `vitest`, `jest`, `tsc`, `next`, `playwright`, `ruff`, `pytest`, `mypy`, `phpunit`, `phpstan`, `pest`, `paratest`, `rubocop`, `rspec`.
- Cloud/sistema: `aws`, `docker`, `kubectl`, `oc`, `curl`, `wget`, `psql`, `ls`, `tree`, `find`, `grep`, `rg`, `wc`, `env`, `log`.
- Agentes/editores: Claude, Cursor, Windsurf, Cline/Roo, Kilo Code, Antigravity, Pi, Hermes, Factory Droid, Codex/Copilot assets.
- Distribuicao: GitHub Releases, Homebrew, scripts de instalacao shell.

## Observacoes para Proximas Etapas

- 🟢 **CONFIRMADO** — A organizacao do codigo favorece specs por modulo tecnico (`cmds`, `core`, `hooks`, `analytics`, `discover`, `learn`, `parser`, `filters`, `openclaw`).
- 🟡 **INFERIDO** — Dentro de `cmds`, pode ser util detalhar por submodulo/ecossistema na fase Archaeologist: `git`, `js`, `python`, `jvm`, `dotnet`, `cloud`, `system`, `rust`, `php`, `ruby`, `go`.
- 🔴 **LACUNA** — O Scout nao executou comandos de build/teste; validade runtime e cobertura real precisam ser confirmadas em fases posteriores se o usuario desejar.
