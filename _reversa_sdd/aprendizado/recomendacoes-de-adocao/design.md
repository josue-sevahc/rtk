# Recomendações de Adoção, Design Técnico

> Design reconstruído de `src/learn/mod.rs` e `src/learn/report.rs`. 🟢 confirmado no código; 🟡 inferido; 🔴 lacuna.

## Interface

| Símbolo | Entrada | Saída | Observação |
|---|---|---|---|
| Filtro de confiança | `Vec<CorrectionPair>`, `f64` | pares elegíveis | Executado antes da deduplicação. 🟢 |
| Filtro de ocorrência | `Vec<CorrectionRule>`, `usize` | regras elegíveis | Executado após a deduplicação. 🟢 |
| `format_console_report` | regras, totais, sessões, dias | `String` | Formato textual. 🟢 |
| `write_rules_file` | regras, caminho | `Result<()>` | Persiste Markdown local. 🟢 |

## Fluxo Principal

1. `learn::run` recebe pares detectados e remove os que não atingem `min_confidence`. 🟢 `src/learn/mod.rs`
2. O detector consolida os pares; a unit remove regras abaixo de `min_occurrences`. 🟢 `src/learn/mod.rs`
3. Com `format == "json"`, a execução monta e imprime o objeto com metadados e regras. 🟢 `src/learn/mod.rs`
4. Nos demais formatos, `format_console_report` constrói o texto. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`
5. Se `write_rules` está ativo e há regras, `write_rules_file` cria o pai necessário e escreve o Markdown. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Fluxos Alternativos

- **Nenhuma regra:** o relatório textual retorna cabeçalho e mensagem de ausência; não há escrita de arquivo. 🟢 `src/learn/report.rs`, `src/learn/mod.rs`
- **Regra recorrente:** o texto exibe marcador `[Nx]` e o Markdown acrescenta `(seen Nx)`. 🟢 `src/learn/report.rs`
- **Falha de filesystem:** a escrita retorna erro ao chamador. 🟢 `src/learn/report.rs`

## Estruturas e Formatos

- O JSON contém `sessions_scanned`, `total_corrections` e regras com `wrong`, `right`, `error_type`, `occurrences` e `base_command`. 🟢 `src/learn/mod.rs`
- O texto mostra o cabeçalho com quantidade de regras, correções, sessões e dias, seguido de pares e primeira linha do erro. 🟢 `src/learn/report.rs`
- O Markdown começa com cabeçalho gerado, agrupa regras por `base_command` e instrui `Use right not wrong`. 🟢 `src/learn/report.rs`

## Dependências

- `CorrectionRule` da subunit de detecção para os dados publicados. 🟢 `src/learn/report.rs`
- `serde_json` para saída estruturada. 🟢 `src/learn/mod.rs`
- `std::fs`, `std::collections::HashMap` e `std::path::Path` para agrupamento e persistência. 🟢 `src/learn/report.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Aplicar confiança antes e recorrência depois preserva a semântica dos pares detectados. | `src/learn/mod.rs` | 🟢 |
| O JSON não aciona escrita de regras, mesmo com `write_rules`. | `src/learn/mod.rs` | 🟢 |
| Ordenar comandos-base torna o arquivo Markdown determinístico por grupo. | `src/learn/report.rs` | 🟢 |

## Riscos e Lacunas

- 🟢 `write_rules_file` recompõe todo o conteúdo e usa `fs::write`, portanto uma nova geração substitui edições manuais; não há mesclagem no módulo. `src/learn/report.rs:54`
- 🔴 A análise não verificou integração efetiva do arquivo gerado com o ambiente de agentes.
