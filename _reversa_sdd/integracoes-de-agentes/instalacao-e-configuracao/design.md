# Instalação e Configuração de Integrações, Design Técnico

## Interface

| Símbolo | Retorno | Observação |
|---|---|---|
| `init::run` | `Result<()>` | Despacha init/uninstall por agente e escopo. 🟢 `src/hooks/init.rs` |
| `write_if_changed` | resultado de alteração | Evita escrita redundante; respeita dry-run. 🟢 `src/hooks/init.rs` |
| `atomic_write` | `Result<()>` | Tempfile no diretório destino seguido de rename. 🟢 `src/hooks/init.rs` |
| `integrity::store_hash` | `Result<()>` | Persiste SHA-256 sidecar. 🟢 `src/hooks/integrity.rs` |
| `hook_check::status` | `HookStatus` | Detecta hook atual, ausente ou obsoleto. 🟢 `src/hooks/hook_check.rs` |

## Fluxo Principal

1. Validar combinação de flags, agente, escopo, confirmação e dry-run. 🟢 `src/hooks/init.rs`
2. Resolver diretórios/configuração do host e construir artefato: settings JSON, instruções Markdown, plugin ou script. 🟢 `src/hooks/init.rs`
3. Ler o conteúdo existente, verificar idempotência, fazer backup quando necessário e escrever atomicamente apenas se houver delta. 🟢 `src/hooks/init.rs`
4. Para hook script, gravar baseline SHA-256 após a instalação. 🟢 `src/hooks/integrity.rs`
5. Em diagnósticos, preferir registro de hook binário nativo; caso contrário, inspecionar script e versão. 🟢 `src/hooks/hook_check.rs`

## Fluxos Alternativos

- **Dry-run:** imprime ação e não altera filesystem. 🟢 `src/hooks/init.rs`
- **Patch recusado ou `--no-patch`:** preserva arquivo e entrega instruções manuais. 🟢 `src/hooks/init.rs`
- **Hash ausente, órfão ou hook ausente:** reporta estado distinto sem classificá-lo como adulteração. 🟢 `src/hooks/integrity.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Patching é idempotente e não destrutivo. | `src/hooks/init.rs` | 🟢 |
| Persistência atômica reduz risco de configuração parcialmente escrita. | `src/hooks/init.rs` | 🟢 |
| Integridade é uma barreira adicional, não proteção absoluta contra quem controla o filesystem. | `src/hooks/integrity.rs` | 🟢 |

## Riscos e Lacunas

- 🔴 Caminhos e formatos de configuração de hosts externos precisam ser validados em instalações reais.
