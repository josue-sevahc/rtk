# Descoberta, Design Técnico

> Design reconstruído de `src/discover/`. 🟢 confirmado no código; 🟡 inferido; 🔴 lacuna.

## Interface

| Símbolo | Entrada | Saída | Observação |
|---|---|---|---|
| `discover::run` | projeto, escopo, dias, limite, formato, verbose | `Result<()>` | Coordena o relatório. 🟢 |
| `ClaudeProvider::discover_sessions` | filtro de projeto e dias | `Vec<PathBuf>` | Localiza JSONL Claude. 🟢 |
| `ClaudeProvider::extract_commands` | caminho da sessão | `Vec<ExtractedCommand>` | Associa tool use e tool result. 🟢 |
| `classify_command` | segmento shell | `Classification` | Escolhe suportado, ignorado ou não suportado. 🟢 |
| `rewrite_command` | comando e filtros | `Option<String>` | Devolve proposta segura ou ausência. 🟢 |
| `format_text` / `format_json` | `DiscoverReport` | `String` | Serializa a visão de descoberta. 🟢 |

## Fluxo Principal

1. A CLI encaminha os argumentos de `discover` para `discover::run`. 🟢 `src/main.rs`
2. O provider resolve o diretório Claude, filtra projetos/datas e encontra arquivos `.jsonl`. 🟢 `src/discover/provider.rs`
3. Cada sessão é lida linha a linha; `tool_use` Bash é associado a `tool_result` pelo id. 🟢 `src/discover/provider.rs`
4. Cada comando é partido por operadores relevantes, passa pelo bypass `RTK_DISABLED` e pela classificação normalizada. 🟢 `src/discover/mod.rs`, `src/discover/registry.rs`
5. Buckets acumulam contagem, exemplo frequente, saída bruta e economia; ao fim são ordenados e materializados em `DiscoverReport`. 🟢 `src/discover/mod.rs`, `src/discover/report.rs`
6. O formatter imprime texto ou JSON; integrações não-Claude recebem nota de cobertura por `rtk gain`. 🟢 `src/discover/report.rs`

## Estruturas e Limites

- `ExtractedCommand` guarda comando, tamanho e preview de resultado, flag de erro, sessão e índice de sequência. 🟢 `src/discover/provider.rs`
- `Classification` diferencia `Supported`, `Unsupported` e `Ignored`; suporte inclui equivalente RTK, categoria, percentual e status. 🟢 `src/discover/registry.rs`
- `DiscoverReport` inclui sessões, comandos, oportunidades, erros de parsing, bypasses e estado de agentes. 🟢 `src/discover/report.rs`
- O lexer gera `Arg`, `Operator`, `Pipe`, `Redirect` e `Shellism`, com offsets em bytes. 🟢 `src/discover/lexer.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Compartilhar registry entre descoberta e hooks evita divergência entre recomendação e rewrite real. | `mod.rs`, `hooks/rewrite_cmd.rs` | 🟢 |
| Preferir degradação por arquivo ou linha a falhar o scan global. | `mod.rs`, `provider.rs` | 🟢 |
| Usar média ponderada por bucket evita que a primeira regra vista determine o percentual. | `mod.rs` | 🟢 |
| Conservar comandos shell incertos protege compatibilidade e permissões do host. | `lexer.rs`, `registry.rs`, ADR 003 | 🟢 |

## Dependências

- `hooks::init` resolve o diretório Claude; `hooks::constants` descreve artefatos de agentes. 🟢
- `core::utils` e `core::toml_filter` participam da normalização e de filtros customizados. 🟢
- `serde_json`, `regex`, `walkdir` e `dirs` apoiam serialização, regras e filesystem. 🟢 `Cargo.toml`, `src/discover/`

## Riscos e Lacunas

- 🔴 O acoplamento funcional de `discover` com `hooks` e `analytics` merece validação de ciclos de dependência em uma reconstrução.
- 🔴 A equivalência de rewrite para sintaxe shell rara depende de testes de regressão, não apenas de inspeção estática.
