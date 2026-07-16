# Análise de Histórico, Design Técnico

## Interface

| Símbolo | Entrada | Saída | Confiança |
|---|---|---|---|
| `ClaudeProvider::encode_project_path` | caminho | nome de diretório Claude | 🟢 |
| `discover_sessions` | projeto opcional e dias | caminhos JSONL | 🟢 |
| `extract_commands` | arquivo JSONL | `Vec<ExtractedCommand>` | 🟢 |
| `format_text` | relatório e limite | terminal formatado | 🟢 |
| `format_json` | relatório | JSON pretty | 🟢 |

## Fluxo Principal

1. Resolver `~/.claude/projects` pela inicialização de hooks. 🟢 `provider.rs`
2. Caminhar arquivos JSONL, excluir symlinks e aplicar filtros de mtime/projeto. 🟢 `provider.rs`
3. Ler linhas, coletar tool uses Bash pendentes e resultados indexados por id. 🟢 `provider.rs`
4. Associar cada uso ao resultado opcional, mantendo `session_id` e `sequence_index`. 🟢 `provider.rs`
5. Entregar comandos ao agregador de descoberta, que renderiza o `DiscoverReport`. 🟢 `mod.rs`, `report.rs`

## Dados e Observabilidade

- Um resultado guarda comprimento, preview e `is_error`; esse desenho permite análise posterior de aprendizado sem reler o arquivo. 🟢 `provider.rs`, `src/learn/`
- O texto contém resumo de sessões, comandos Bash, percentual já RTK, tabelas de oportunidades e notas de agentes. 🟢 `report.rs`
- JSON usa `serde` sobre o mesmo modelo de relatório. 🟢 `report.rs`

## Riscos

- 🟡 O pareamento por id pressupõe que a sessão preserva a relação tool use/tool result dentro do arquivo analisado. `provider.rs`
- 🔴 A leitura íntegra da sessão antes do retorno pode precisar de limites adicionais para arquivos excepcionalmente grandes.
