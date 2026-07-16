# Integrações de Agentes

> Contrato operacional reconstruído de `src/hooks/`. Afirmações 🟢 foram confirmadas no código legado; 🟡 são inferências; 🔴 exigem validação humana.

## Visão Geral

Esta unit integra o RTK aos agentes de IA e seus hooks: instala e remove artefatos, recebe payloads nativos, consulta permissões do host, solicita reescritas e protege a integridade dos hooks. 🟢 `src/hooks/README.md`, `src/hooks/mod.rs`

O módulo é deliberadamente uma camada de integração: a classificação e a reescrita de comandos pertencem ao registro de `discover`, enquanto o RTK usa códigos de saída e payloads específicos de cada host para não assumir controle indevido da autorização. 🟢 `src/hooks/README.md`, `src/hooks/rewrite_cmd.rs`

## Responsabilidades

- Instalar, atualizar e remover integrações para Claude, Cursor, Windsurf, Cline, Codex, Gemini, Copilot, Pi, Droid, Hermes e OpenCode. 🟢 `src/hooks/init.rs`
- Processar payloads de hook de hosts nativos sem corromper seus protocolos JSON. 🟢 `src/hooks/hook_cmd.rs`
- Aplicar a precedência de permissões `Deny > Ask > Allow > Default` antes de autorizar uma reescrita. 🟢 `src/hooks/permissions.rs`
- Expor `rtk rewrite` como ponte de shell com contrato por exit code. 🟢 `src/hooks/rewrite_cmd.rs`
- Detectar adulteração, ausência e obsolescência de hooks. 🟢 `src/hooks/integrity.rs`, `src/hooks/hook_check.rs`
- Controlar a confiança de filtros TOML locais e globais antes que sejam carregados pelo núcleo. 🟢 `src/hooks/trust.rs`
- Registrar auditoria de reescritas somente quando `RTK_HOOK_AUDIT=1`. 🟢 `src/hooks/hook_audit_cmd.rs`

## Regras de Negócio

- Uma regra `Deny` para qualquer segmento de um comando composto prevalece sobre regras `Ask`, `Allow` e o padrão. 🟢 `src/hooks/permissions.rs`
- Construtos shell não atestáveis não recebem autorização automática e são convertidos em `Ask`. 🟢 `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`
- `rtk rewrite` retorna `0` para permitir rewrite, `1` para passthrough, `2` para deny e `3` para rewrite sujeito a confirmação. 🟢 `src/hooks/rewrite_cmd.rs`
- Hook adulterado bloqueia a execução protegida; hook sem baseline, ausente ou com hash órfão não é tratado como adulteração. 🟢 `src/hooks/integrity.rs`
- Um filtro externo é carregável somente quando confiável ou liberado por override de ambiente em CI; arquivo alterado, inválido ou não UTF-8 falha fechado. 🟢 `src/hooks/trust.rs`
- A instalação deve ser idempotente, preservar conteúdo existente e gravar alterações de forma atômica. 🟢 `src/hooks/init.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de Aceite |
|----|-----------|------------|-------------------|
| RF-01 | Instalar e desinstalar integrações por agente, escopo e modo compatíveis, sem duplicar o patch já instalado. 🟢 | Must | Dado um arquivo de configuração com hook equivalente, quando a instalação rodar novamente, então nenhum segundo hook é inserido. |
| RF-02 | Receber e responder payloads de Claude, Cursor, Gemini, Copilot e Droid conforme o protocolo do host. 🟢 | Must | Dado um payload válido e um rewrite autorizado, quando o handler executar, então a resposta contém o campo e a decisão esperados pelo host. |
| RF-03 | Consultar permissões do host antes de expor rewrite automático. 🟢 | Must | Dado um comando composto com segmento negado, quando a decisão for calculada, então nenhuma reescrita auto-allow é retornada. |
| RF-04 | Disponibilizar a ponte `rtk rewrite` por códigos de saída estáveis. 🟢 | Must | Dado comando sem equivalente, negado ou que exige confirmação, quando a ponte rodar, então ela encerra respectivamente com 1, 2 ou 3. |
| RF-05 | Verificar integridade por SHA-256 e impedir o uso de hook adulterado. 🟢 | Must | Dado um script cujo hash diverge do baseline, quando a verificação runtime ocorrer, então a execução protegida é bloqueada. |
| RF-06 | Exigir confiança explícita para filtros TOML externos e listar riscos antes da aprovação. 🟢 | Should | Dado filtro local desconhecido, quando o registry tentar carregá-lo, então seu conteúdo não é devolvido até aprovação válida. |
| RF-07 | Registrar auditoria sanitizada e avisos de hook ausente/desatualizado sem inundar o terminal. 🟢 | Could | Dado `RTK_HOOK_AUDIT=1`, quando houver rewrite, então uma entrada sanitizada é gravada; avisos periódicos ocorrem no máximo uma vez por dia. |

## Requisitos Não Funcionais

| Tipo | Requisito inferido | Evidência no código | Confiança |
|------|--------------------|---------------------|-----------|
| Segurança | Hash SHA-256, arquivos read-only em Unix e falha fechada para filtros não confiáveis protegem a fronteira entre host e CLI. | `src/hooks/integrity.rs`, `src/hooks/trust.rs` | 🟢 |
| Confiabilidade | Patches são idempotentes e usam tempfile no mesmo diretório seguido de persist/rename. | `src/hooks/init.rs` | 🟢 |
| Compatibilidade | Input de hook é limitado a 1 MiB e respostas JSON são emitidas de forma controlada para preservar protocolos de terceiros. | `src/hooks/hook_cmd.rs` | 🟢 |
| Observabilidade | Auditoria é opt-in e avisos de status são rate-limited. | `src/hooks/hook_audit_cmd.rs`, `src/hooks/hook_check.rs` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: Recusar reescrita automática para segmento negado
Dado um comando composto com um segmento coberto por regra Deny
Quando o hook pedir a decisão de reescrita
Então a resposta deve representar bloqueio
E nenhum segmento deve ser auto-autorizado

Cenário: Preservar host em payload inválido
Dado um payload JSON inválido ou vazio
Quando um handler nativo de hook o receber
Então ele deve concluir sem corromper stdout
E o host deve poder seguir com o comando original

Cenário: Bloquear hook adulterado
Dado um hook com baseline SHA-256 registrado
E o conteúdo do hook foi alterado
Quando a verificação runtime ocorrer
Então o status deve ser Tampered
E o caminho protegido deve ser bloqueado
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Decisão de permissões e protocolo de rewrite | Must | Define o caminho crítico que liga os hosts ao RTK. 🟢 |
| Instalação idempotente por agente | Must | Sem artefato correto não há integração executável. 🟢 |
| Integridade e trust | Must | São barreiras de segurança da integração e de filtros externos. 🟢 |
| Auditoria e avisos rate-limited | Could | Aumentam diagnóstico, mas não são necessários para reescrever um comando. 🟢 |

## Rastreabilidade de Código

| Arquivo | Função / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/hooks/init.rs` | `run`, `write_if_changed`, `atomic_write`, patchers por host | 🟢 |
| `src/hooks/hook_cmd.rs` | processadores de payload e `decide_hook_action` | 🟢 |
| `src/hooks/rewrite_cmd.rs` | `run`, `evaluate` | 🟢 |
| `src/hooks/permissions.rs` | `check_command_with_rules` | 🟢 |
| `src/hooks/integrity.rs` | baseline e `runtime_check` | 🟢 |
| `src/hooks/trust.rs` | `check_trust_with_content`, `run_trust` | 🟢 |
| `src/hooks/hook_check.rs` | `maybe_warn` | 🟢 |
| `src/hooks/hook_audit_cmd.rs` | `audit_log` | 🟢 |
