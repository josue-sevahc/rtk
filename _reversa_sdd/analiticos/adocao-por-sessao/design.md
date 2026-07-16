# Adoção por Sessão, Design Técnico

## Interface

| Símbolo | Assinatura | Retorno | Observação |
|---|---|---|---|
| `session_cmd::run` | `(_verbose: u8)` | `Result<()>` | Gera tabela das sessões. |
| `count_rtk_commands` | `&[ExtractedCommand]` | `(total, rtk, output)` | Função de classificação central. |

## Fluxo Principal

1. `ClaudeProvider` localiza até 30 dias de sessões. 🟢
2. O código remove caminhos contendo `subagents`, ordena por mtime descendente e limita a 10. 🟢
3. Cada JSONL é extraído; arquivos inválidos são ignorados. 🟢
4. As cadeias são divididas, as partes são classificadas e a tabela é impressa. 🟢

## Fluxos Alternativos

- Sem sessões: explica como usar Claude Code. 🟢
- Sem comandos Bash: informa que não há sessões com Bash. 🟢
- Erro de parsing de uma sessão: continua com as demais. 🟢

## Dependências

- `discover::provider::{ClaudeProvider, SessionProvider, ExtractedCommand}`. 🟢
- `discover::registry::{split_command_chain, classify_command, Classification}`. 🟢
- `core::utils::format_tokens`. 🟢

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Medir comando suportado como potencialmente reescrito pelo hook. | `count_rtk_commands` | 🟢 |
| Excluir sessões de subagentes para evitar distorção. | `session_cmd::run` | 🟢 |
| Somar `output_len` original da extração, não uma nova estimativa. | `count_rtk_commands` | 🟢 |

## Estado Interno

`SessionSummary` guarda identificador curto, idade da sessão, totais, cobertos e tokens de saída apenas durante a execução. 🟢

## Riscos e Lacunas

- 🟡 O relatório mede sessões Claude Code; outros hosts não participam desse scanner específico.
