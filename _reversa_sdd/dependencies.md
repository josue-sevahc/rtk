# Dependencias — Scout

Projeto: `rtk`
Gerado em: `2026-07-14T14:23:54Z`

## Manifestos

🟢 **CONFIRMADO**:

- `Cargo.toml` — manifesto principal do binario Rust `rtk`.
- `Cargo.lock` — lockfile do Cargo.
- `openclaw/package.json` — manifesto do plugin TypeScript `@rtk-ai/rtk-rewrite`.
- `openclaw/openclaw.plugin.json` — manifesto do plugin OpenClaw.
- `.github/dependabot.yml` — automacao de atualizacoes de dependencias.
- `release-please-config.json` e `.release-please-manifest.json` — automacao de releases.

## Pacote Principal

🟢 **CONFIRMADO**:

- Nome: `rtk`
- Versao: `0.42.4`
- Edicao Rust: `2021`
- Rust minimo: `1.91`
- Tipo: CLI / command-line utility
- Descricao: `Rust Token Killer - High-performance CLI proxy to minimize LLM token consumption`
- Licenca declarada no Cargo: `Apache 2.0`

## Dependencias Rust

Dependencias declaradas em `Cargo.toml`:

| Dependencia | Versao | Papel provavel |
|---|---|---|
| `clap` | `4` + `derive` | CLI, subcomandos e parsing de argumentos |
| `anyhow` | `1.0` | Tratamento ergonomico de erros |
| `ignore` | `0.4` | Walk de arquivos respeitando ignores |
| `walkdir` | `2` | Percurso de diretorios |
| `regex` | `1` | Parsing e filtros por padrao |
| `lazy_static` | `1.4` | Inicializacao estatica de regex/dados |
| `serde` | `1` + `derive` | Serializacao/deserializacao |
| `serde_json` | `1` + `preserve_order` | JSON estruturado |
| `colored` | `2` | Saida colorida no terminal |
| `dirs` | `5` | Localizacao de diretorios do usuario/sistema |
| `rusqlite` | `0.31` + `bundled` | Tracking local em SQLite |
| `toml` | `0.8` | Configuracao e filtros TOML |
| `chrono` | `0.4` | Datas/tempos |
| `tempfile` | `3` | Arquivos temporarios |
| `sha2` | `0.10` | Hashes/integridade |
| `ureq` | `2` | HTTP client simples |
| `getrandom` | `0.4` | Randomness |
| `flate2` | `1.0` | Compressao/descompressao gzip |
| `quick-xml` | `0.37` | Parsing XML/TRX/Maven e similares |
| `which` | `8` | Resolucao de binarios no PATH |
| `automod` | `1` | Declaracao automatica de modulos |

Dependencia condicional Unix:

| Dependencia | Versao | Condicao |
|---|---|---|
| `libc` | `0.2` | `cfg(unix)` |

Build dependencies:

| Dependencia | Versao |
|---|---|
| `toml` | `0.8` |

## Perfis e Lints

🟢 **CONFIRMADO**:

- Release otimizado com `opt-level = 3`, `lto = true`, `codegen-units = 1`, `panic = "abort"` e `strip = true`.
- Lint Rust: `unsafe_code = "deny"` e `warnings = "deny"`.
- Empacotamento configurado para `.deb` e RPM.

## Plugin OpenClaw

🟢 **CONFIRMADO**:

- Pacote: `@rtk-ai/rtk-rewrite`
- Versao: `1.0.0`
- Entrada: `index.ts`
- Licenca: `Apache-2.0`
- Dependencias runtime declaradas: nenhuma no `package.json`.
- Gerenciador sugerido: npm/package.json sem lockfile local detectado no subdiretorio.

## Gerenciadores de Pacotes

🟢 **CONFIRMADO**:

- Rust/Cargo — principal.
- npm/package.json — plugin OpenClaw.
- Homebrew Formula — distribuicao em `Formula/rtk.rb`.

## Dependencias Operacionais Externas

🟢 **CONFIRMADO** — A aplicacao chama ferramentas externas como alvo de proxy/filtro. Exemplos detectados por subcomandos e modulos:

- Git/GitHub/GitLab: `git`, `gh`, `glab`, `gt`.
- JS/TS: `npm`, `npx`, `pnpm`, `vitest`, `jest`, `tsc`, `next`, `prettier`, `playwright`, `prisma`.
- Python: `ruff`, `pytest`, `mypy`, `pip`, `uv`.
- JVM: `mvn`, `gradle`, `gradlew`.
- .NET: `dotnet`.
- Go: `go`, `golangci-lint`.
- PHP: `php`, `phpunit`, `phpstan`, `pest`, `paratest`, `ecs`, `pint`.
- Ruby: `rake`, `rubocop`, `rspec`.
- Cloud/sistema: `aws`, `docker`, `kubectl`, `oc`, `curl`, `wget`, `psql`, `ls`, `tree`, `find`, `grep`, `rg`, `wc`.

## CI/CD e Segurança

🟢 **CONFIRMADO**:

- CI usa `cargo fmt`, `cargo clippy --all-targets`, `cargo test --all`, security scan, Semgrep, benchmark e revisao documental.
- Release builda multiplas plataformas via GitHub Actions.
- Dependabot esta presente para manutencao automatica.
- `.semgrep.yml` existe como configuracao de analise estatica.

## Lacunas

- 🔴 **LACUNA** — O Scout nao calculou arvore completa transitive do `Cargo.lock`; isso pode ser feito por um agente dedicado se for necessario auditar supply chain.
- 🔴 **LACUNA** — O Scout nao executou `cargo metadata`, `cargo audit` ou testes; as dependencias foram extraidas por leitura estatica dos manifestos.
