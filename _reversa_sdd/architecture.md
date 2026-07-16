# Arquitetura - RTK

Projeto: `rtk`  
Fase: Interpretacao (Arquiteto)  
Gerado em: `2026-07-14T22:27:28Z`

## Visao geral

🟢 **CONFIRMADO** - RTK e um CLI Rust local que intercepta ou recebe comandos de desenvolvimento, executa a ferramenta original, reduz a saida antes de ela chegar ao contexto de um agente e registra metricas locais de economia. O processo principal e `src/main.rs`; a extensao OpenClaw e um adaptador TypeScript separado.

O sistema nao possui servico backend proprio, fila, cache distribuido ou banco remoto. A persistencia local e SQLite; configuracao e artefatos de hooks vivem no diretorio do usuario ou no projeto do consumidor.

## Personas e fronteiras

| Ator/Sistema | Relacao com RTK | Confianca |
|---|---|---|
| Desenvolvedor | Executa `rtk`, configura filtros, consulta ganho e instala integrações. | 🟢 |
| Agente de coding | Solicita comandos ao host; recebe output compactado quando o hook reescreve o comando. | 🟢 |
| Hosts de agentes | Claude, Cursor, Gemini, Copilot, Droid e outros invocam handlers por JSON/stdin ou arquivos de configuracao. | 🟢 |
| Ferramentas externas | Git, CLIs de build/teste/lint, cloud e sistema sao processos filhos selecionados pelo comando. | 🟢 |
| OpenClaw | Carrega plugin TypeScript que chama `rtk rewrite` antes de `exec`. | 🟢 |
| Endpoint de telemetria | Recebe ping anonimo somente quando compilado, consentido e habilitado. | 🟢 |

## Containers e responsabilidades

| Container | Tecnologia | Responsabilidade | Persistencia/Interface |
|---|---|---|---|
| CLI RTK | Rust, Clap | Parse, fallback, dispatch e propagacao de exit codes/sinais. | Processo local e argumentos de shell. |
| Motor de execucao e filtros | Rust (`core`, `cmds`, `parser`, `filters`) | Executa processos, filtra streaming/buffer, aplica guard `never_worse`, tee e tracking. | stdout/stderr, TOML embutido. |
| Integracao de hooks | Rust (`hooks`, `discover`) | Classifica/rewrite comandos, preserva permissoes do host, instala e verifica hooks. | JSON stdin/stdout, settings de hosts, hash SHA-256. |
| Analytics e aprendizado | Rust (`analytics`, `learn`) | Consulta economia, descobre oportunidades e infere correcoes recorrentes. | SQLite e historicos locais do host. |
| Banco de tracking | SQLite embutido via `rusqlite` | Historico de comandos e falhas de parse, com retencao. | `history.db` no diretorio de dados RTK. |
| Plugin OpenClaw | TypeScript | Intercepta `exec`, interpreta exit codes do rewrite e solicita aprovacao quando necessario. | API de plugin OpenClaw e subprocesso `rtk`. |

## Fluxo critico

1. Um desenvolvedor ou host envia um comando.
2. O hook, quando instalado, consulta as permissoes do host e tenta um rewrite conhecido; se nao puder atestar o comando, defere ao host.
3. `main` despacha para wrapper especializado, filtro TOML ou passthrough.
4. `core::runner` executa a ferramenta externa, conserva o exit code e evita emitir uma saida maior que a bruta.
5. O tracker grava metricas no SQLite local; telemetria e opcional e assincrona.

## Integracoes externas

| Integracao | Direcao | Protocolo/Formato | Contrato observado |
|---|---|---|---|
| Hosts de agentes | Bidirecional local | JSON por stdin/stdout e arquivos de settings | `updatedInput`, decisoes allow/ask/defer e no-op preservam o controle do host. |
| OpenClaw | Plugin -> CLI | API de plugin + subprocesso/exit code | `0` allow, `2` deny, `3` ask/rewrite; o plugin pode pedir aprovacao. |
| Ferramentas de desenvolvimento | CLI -> processo filho | argumentos, stdout/stderr e exit code | RTK e proxy/filtro; os contratos pertencem a cada ferramenta externa. |
| SQLite | CLI -> arquivo local | SQL | Tracking de `commands` e `parse_failures`; WAL e busy timeout tentados de modo nao fatal. |
| Telemetria | CLI -> endpoint configurado no build | HTTP JSON via `ureq` | Ping diario anonimo, condicionado por consentimento. |
| Distribuicao/CI | Repositorio -> GitHub Releases/Homebrew | GitHub Actions, release assets | Integracao de entrega, nao parte do caminho runtime do CLI. |

## Dividas e riscos arquiteturais

- 🔴 A camada `core` chama `hooks` em `toml_filter.rs` e `telemetry.rs`, apesar da documentacao indicar direcao oposta; a fronteira de modulos precisa ser esclarecida.
- 🔴 O lexer de shell e propositalmente parcial, portanto comandos complexos podem cair em defer/passthrough ou exigir novas regras.
- 🔴 O plugin OpenClaw usa formas manuais/`any`, nao possui testes automatizados no subdiretorio e nao separa claramente erro operacional de ausencia de rewrite.
- 🔴 Filtros extensos (`git`, `cargo`, `aws`, `dotnet`, `mvn`, `search`) dependem de compatibilidade com CLIs externas que nao foi exercitada nesta extracao.
- 🟢 A implementacao usa `history.db` por `src/core/constants.rs`; documentos legados ainda citam `tracking.db`, portanto a divergencia documental esta confirmada.
- 🔴 Nao houve teste real de concorrencia SQLite/WAL, instalacao em todos os hosts ou comportamento em Windows.

## Referencias

- [Contexto C4](c4-context.md)
- [Containers C4](c4-containers.md)
- [Componentes C4](c4-components.md)
- [ERD completo](erd-complete.md)
- [Matriz de impacto](traceability/spec-impact-matrix.md)
