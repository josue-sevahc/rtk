# Protocolo de Hooks e Permissões, Design Técnico

## Interface

| Símbolo | Retorno | Observação |
|---|---|---|
| `check_command_for` | `PermissionVerdict` | Carrega regras do host e decide o comando. 🟢 `src/hooks/permissions.rs` |
| `check_command_with_rules` | `Allow`, `Deny`, `Ask` ou `Default` | Função testável de precedência. 🟢 `src/hooks/permissions.rs` |
| `rewrite_cmd::run` | exit 0–3 | Bridge shell para hooks. 🟢 `src/hooks/rewrite_cmd.rs` |
| `decide_hook_action` | `HookDecision` | Converte permissões e rewrite em ação do host. 🟢 `src/hooks/hook_cmd.rs` |

## Fluxo Principal

1. O handler lê no máximo 1 MiB de stdin e identifica o formato do host. 🟢 `src/hooks/hook_cmd.rs`
2. As regras são carregadas no escopo permitido pelo host; regras inválidas são ignoradas com aviso quando aplicável. 🟢 `src/hooks/permissions.rs`
3. A avaliação divide cadeias; `Deny` é aplicado primeiro. Construtos não atestáveis recebem `Ask` ou defer. 🟢 `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`
4. O registry tenta reescrever o comando. Somente `Allow` em todos os segmentos gera auto-allow; `Ask` e `Default` preservam a confirmação. 🟢 `src/hooks/rewrite_cmd.rs`
5. O resultado vira exit code ou JSON específico: `updatedInput`, `modifiedArgs`, `decision` ou equivalente. 🟢 `src/hooks/hook_cmd.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| A política mais restritiva vence; Default é least privilege. | `src/hooks/permissions.rs` | 🟢 |
| Cada segmento precisa de allow próprio para evitar elevação de cadeia composta. | `src/hooks/permissions.rs` | 🟢 |
| Sem equivalente RTK ou com shell ambíguo, o sistema faz passthrough/defer. | `src/hooks/rewrite_cmd.rs`, `src/hooks/hook_cmd.rs` | 🟢 |
| stdout é reservado ao JSON de protocolo; diagnósticos usam stderr. | `src/hooks/hook_cmd.rs` | 🟢 |

## Riscos e Lacunas

- 🔴 APIs e formatos de hook de terceiros não foram exercitados em runtime.
- 🟡 Cursor e Gemini usam fontes de configuração deliberadamente restritas para não exceder permissões do host.
