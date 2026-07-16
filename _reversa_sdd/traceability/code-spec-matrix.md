# Matriz Codigo-Spec - RTK

> Gerada durante a revisao cruzada em 2026-07-16. A matriz liga cada unit canonica aos artefatos legados que sustentam seu contrato.

## Cobertura por Unit

| Unit | Codigo e artefatos fonte | Cobertura | Observacao |
|---|---|---|---|
| `entrada-cli` | `src/main.rs`, `src/core/constants.rs` | Direta | Parse, dispatch, ciclo de vida e proxy. |
| `entrada-cli/roteamento-e-fallback` | `src/main.rs`, `src/core/toml_filter.rs`, `src/core/tee.rs` | Direta | Distincao entre meta-comando, filtro e passthrough. |
| `wrappers-comandos` | `src/cmds/`, `src/core/runner.rs`, `src/core/stream.rs` | Direta | Catalogo de wrappers e infraestrutura compartilhada. |
| `wrappers-comandos/execucao-filtrada` | `src/core/runner.rs`, `src/core/stream.rs`, `src/core/guard.rs`, `src/core/tee.rs`, `src/core/tracking.rs` | Direta | Streams, filtro, guard, tee e tracking. |
| `nucleo` | `src/core/` | Direta | Configuracao, execucao, persistencia e utilitarios centrais. |
| `nucleo/pipeline-de-filtros-toml` | `src/core/toml_filter.rs`, `src/hooks/trust.rs`, `build.rs` | Direta | Parse, trust, selecao e aplicacao da DSL. |
| `nucleo/tracking-e-telemetria` | `src/core/tracking.rs`, `src/core/telemetry.rs`, `src/core/telemetry_cmd.rs`, `src/core/config.rs` | Direta | SQLite local, consentimento, ping e erasure. |
| `integracoes-de-agentes` | `src/hooks/` | Direta | Superficie completa de instalacao, hook, trust e permissoes. |
| `integracoes-de-agentes/instalacao-e-configuracao` | `src/hooks/init.rs`, `src/hooks/integrity.rs`, `src/hooks/hook_check.rs` | Direta | Escrita idempotente, hash e diagnostico. |
| `integracoes-de-agentes/protocolo-de-hooks-e-permissoes` | `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`, `src/hooks/hook_cmd.rs` | Direta | Precedencia de ACL e protocolos de host. |
| `analiticos` | `src/analytics/` | Direta | Agregacoes e comandos analiticos. |
| `analiticos/relatorios-de-economia` | `src/analytics/gain.rs`, `src/analytics/cc_economics.rs`, `src/analytics/ccusage.rs` | Direta | Economia local e correlacao com custos externos. |
| `analiticos/adocao-por-sessao` | `src/analytics/session_cmd.rs`, `src/analytics/ccusage.rs` | Direta | Cobertura RTK por sessao Claude Code. |
| `descoberta` | `src/discover/` | Direta | Scan de historico e relatorio de oportunidades. |
| `descoberta/reescrita-e-classificacao` | `src/discover/lexer.rs`, `src/discover/registry.rs`, `src/discover/rules.rs` | Direta | Tokenizacao shell, classificacao e rewrite. |
| `descoberta/analise-de-historico` | `src/discover/provider.rs`, `src/discover/report.rs`, `src/discover/mod.rs` | Direta | Leitura de sessoes e consolidacao. |
| `aprendizado` | `src/learn/` | Direta | Deteccao de correcoes recorrentes. |
| `aprendizado/recomendacoes-de-adocao` | `src/learn/mod.rs`, `src/learn/report.rs` | Direta | Relatorio e escrita opcional de rules. |
| `perfis-de-filtros` | `src/filters/`, `src/filters/README.md`, `build.rs` | Direta | DSL e catalogo de perfis. |
| `perfis-de-filtros/catalogo-embutido` | `build.rs`, `src/filters/`, `src/core/toml_filter.rs` | Direta | Incorporacao deterministica e lookup runtime. |
| `plugin-openclaw` | `openclaw/index.ts`, `openclaw/openclaw.plugin.json`, `src/hooks/rewrite_cmd.rs` | Direta | Adaptador do protocolo de rewrite. |
| `parser-de-saida` | `src/parser/mod.rs`, `src/parser/types.rs`, `src/parser/README.md` | Direta | Tipos canonicos, tiers e fallback. |
| `parser-de-saida/formatacao-estruturada` | `src/parser/formatter.rs` | Direta | Modos de formatacao e limites de detalhe. |
| `documentacao-do-produto` | `docs/guide/`, `docs/contributing/`, `docs/TELEMETRY.md`, `.github/docs-pipeline-contract.md` | Direta | Jornada publica e contrato com o site externo. |
| `automacao-e-scripts` | `scripts/`, `.github/workflows/` | Direta | Build, release, benchmark e diagnostico. |
| `automacao-e-scripts/instalacao-e-validacao` | `scripts/install-local.sh`, `scripts/check-installation.sh`, `scripts/test-install.sh`, `.github/workflows/release.yml` | Direta | Instalacao local/remota e verificacao operacional. |

## Validacoes

- As 26 units possuem os tres arquivos canonicos: `requirements.md`, `design.md` e `tasks.md`.
- Todas as referencias literais de arquivo nas specs apontam para artefatos existentes; placeholders de exemplo, como `src/filters/<comando>.toml`, nao sao caminhos concretos.
- OpenAPI nao se aplica: nao foi encontrada superficie HTTP servida pelo legado.
- As dependencias de runtime externo permanecem com confianca vermelha ate validacao contra plataformas, ferramentas e hosts reais.

