# Integrações de Agentes, Design Técnico

> Design reconstruído de `src/hooks/`. 🟢 confirmado no código; 🟡 inferido; 🔴 lacuna.

## Interface

| Símbolo | Assinatura / protocolo | Retorno | Observação |
|---|---|---|---|
| `init::run` | flags de agente, escopo e modo | `Result<()>` | Instala, migra ou remove artefatos. 🟢 `src/hooks/init.rs` |
| `rewrite_cmd::run` | comando shell | exit code 0–3 | Ponte para hooks shell. 🟢 `src/hooks/rewrite_cmd.rs` |
| `rewrite_cmd::evaluate` | comando e regras | decisão de rewrite | Nega antes de delegar ao registry. 🟢 `src/hooks/rewrite_cmd.rs` |
| `permissions::check_command_with_rules` | comando e regras do host | `PermissionVerdict` | Avalia comandos compostos. 🟢 `src/hooks/permissions.rs` |
| `integrity::runtime_check` | caminho de hook | status | Bloqueia somente `Tampered`. 🟢 `src/hooks/integrity.rs` |
| `trust::check_trust_with_content` | caminho TOML | conteúdo opcional | Só libera `Trusted` ou `EnvOverride`. 🟢 `src/hooks/trust.rs` |

## Fluxo Principal

1. `rtk init` valida flags e seleciona o instalador do host. 🟢 `src/hooks/init.rs`
2. O patcher lê ou cria a configuração, verifica idempotência, faz backup quando necessário e grava por tempfile/rename. 🟢 `src/hooks/init.rs`
3. Um host envia payload ou invoca `rtk rewrite`; o handler limita stdin, interpreta o formato e consulta as permissões nativas. 🟢 `src/hooks/hook_cmd.rs`, `src/hooks/permissions.rs`
4. `evaluate` aplica deny, rejeita construtos não atestáveis e chama `discover::registry::rewrite_command`. 🟢 `src/hooks/rewrite_cmd.rs`
5. A decisão é convertida no payload e semântica próprios do host; no caso CLI, o código 0–3 comunica a ação ao shell. 🟢 `src/hooks/hook_cmd.rs`, `src/hooks/rewrite_cmd.rs`
6. Integridade, trust e auditoria complementam o caminho sem deslocar a política de autorização do host. 🟢 `src/hooks/integrity.rs`, `src/hooks/trust.rs`, `src/hooks/hook_audit_cmd.rs`

## Fluxos Alternativos

- **Payload vazio, inválido ou no-op:** handlers retornam sucesso/fallback silencioso para não bloquear o host. 🟢 `src/hooks/hook_cmd.rs`
- **Regra `Ask`:** a resposta inclui rewrite, mas deixa a confirmação no host. 🟢 `src/hooks/hook_cmd.rs`
- **Filtro inválido ou hash divergente:** o conteúdo não é carregado. 🟢 `src/hooks/trust.rs`
- **Hook adulterado:** o runtime bloqueia a integração protegida. 🟢 `src/hooks/integrity.rs`

## Dependências

- `discover::registry` e lexer: classificação, divisão e rewrite. 🟢 `src/hooks/rewrite_cmd.rs`
- `core`: configuração, helpers de stream e carregamento posterior de filtros. 🟢 `src/hooks/mod.rs`
- `serde_json`: payloads e arquivos de configuração de hosts. 🟢 `src/hooks/hook_cmd.rs`, `src/hooks/init.rs`
- `sha2` e `tempfile`: integridade e escrita atômica. 🟢 `src/hooks/integrity.rs`, `src/hooks/init.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| O host conserva a autoridade final; RTK devolve decisão em vez de executar o comando. | `src/hooks/hook_cmd.rs`, `src/hooks/permissions.rs` | 🟢 |
| Deny tem precedência sobre qualquer allow no comando composto. | `src/hooks/permissions.rs` | 🟢 |
| Reescrita shell usa códigos de saída para evitar um protocolo textual frágil. | `src/hooks/rewrite_cmd.rs` | 🟢 |
| Patches de configuração preservam conteúdo e são idempotentes. | `src/hooks/init.rs` | 🟢 |
| Trust de filtros é baseado em hash, com exceção de ambiente CI controlado. | `src/hooks/trust.rs` | 🟢 |

## Estado Interno e Observabilidade

- Baselines SHA-256 ficam em sidecars `.rtk-hook.sha256`; o trust store guarda hashes e estado de aprovação. 🟢 `src/hooks/integrity.rs`, `src/hooks/trust.rs`
- Auditoria escreve `timestamp | action | original | rewritten` somente com `RTK_HOOK_AUDIT=1`, sanitizando quebras de linha e pipes. 🟢 `src/hooks/hook_audit_cmd.rs`
- Avisos de hook ausente ou desatualizado são limitados a uma ocorrência diária. 🟢 `src/hooks/hook_check.rs`

## Riscos e Lacunas

- 🔴 Instalações reais nos ambientes Claude, Cursor, Gemini, Droid e Copilot não foram exercitadas.
- 🔴 Contratos externos dos hosts podem mudar sem alteração do código RTK.
- 🟡 A extensão de `init.rs` recomenda a decomposição por host nas subunits seguintes; esta unit mantém o contrato transversal.
