# Parser de Saida, Design Tecnico

> Design reconstruido de `src/parser/mod.rs`, `src/parser/types.rs` e seus consumidores. 🟢 confirmado no codigo; 🟡 inferido; 🔴 lacuna.

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---|---|---|---|
| `OutputParser::parse` | `(&str)` | `ParseResult<Self::Output>` | Contrato implementado por parser de ferramenta. 🟢 |
| `OutputParser::parse_with_tier` | `(&str, u8)` | `ParseResult<Self::Output>` | Converte resultado acima do maximo em passthrough. 🟢 |
| `ParseResult::tier` | `(&self)` | `u8` | `1`, `2` ou `3`. 🟢 |
| `ParseResult::is_ok` | `(&self)` | `bool` | Aceita `Full` e `Degraded`. 🟢 |
| `ParseResult::map` | `(self, FnOnce(T) -> U)` | `ParseResult<U>` | Preserva tier e avisos. 🟢 |
| `truncate_output` | `(&str, usize)` | `String` | Mantem o texto ou anexa marcador de truncamento. 🟢 |
| `extract_json_object` | `(&str)` | `Option<&str>` | Retorna um slice do objeto JSON completo. 🟢 |

## Fluxo Principal

1. Um comando especializado implementa `OutputParser` e recebe a saida bruta da ferramenta. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`
2. A implementacao tenta construir o tipo canonico e devolve `Full` quando a estrutura e valida. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`
3. Quando a estrutura principal falha mas ha extracao textual suficiente, ela devolve `Degraded` com o motivo. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`
4. Sem extracao confiavel, `truncate_passthrough` consulta o limite configurado e devolve o texto bruto limitado em `Passthrough`. 🟢 `src/parser/mod.rs`
5. O consumidor seleciona `FormatMode` e formata dados dos tiers 1 ou 2; no tier 3, exibe a saida de fallback. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`

## Fluxos Alternativos

- **JSON com prefixo:** Vitest tenta desserializacao direta e, se falhar, usa `extract_json_object` antes de degradar. 🟢 `src/cmds/js/vitest_cmd.rs`
- **Tier acima do permitido:** `parse_with_tier` substitui qualquer resultado de tier maior por passthrough do input original. 🟢 `src/parser/mod.rs`
- **Texto ja dentro do limite:** `truncate_output` o retorna sem marcador. 🟢 `src/parser/mod.rs`
- **Chaves dentro de string JSON:** o extrator ignora chaves enquanto esta dentro de string e considera escapes. 🟢 `src/parser/mod.rs`

## Estruturas de Dados

| Estrutura | Campos / variantes | Papel |
|---|---|---|
| `ParseResult<T>` | `Full(T)`, `Degraded(T, Vec<String>)`, `Passthrough(String)` | Resultado e nivel de confianca operacional. 🟢 |
| `TestResult` | totais, duracao opcional, falhas | Forma canonica para runners de testes. 🟢 `src/parser/types.rs` |
| `TestFailure` | nome, arquivo, mensagem, stack opcional | Detalhe de uma falha. 🟢 `src/parser/types.rs` |
| `DependencyState` | contagens e dependencias | Forma canonica para listagem/desatualizacao. 🟢 `src/parser/types.rs` |
| `Dependency` | nome, versoes, tipo de dependencia | Item de dependencia. 🟢 `src/parser/types.rs` |

## Dependencias

- `crate::core::config::limits` fornece `passthrough_max_chars`. 🟢 `src/parser/mod.rs`
- `serde::{Deserialize, Serialize}` deriva serializacao para os tipos canonicos. 🟢 `src/parser/types.rs`
- Parsers Vitest, Playwright e pnpm usam o contrato e os tipos canonicos. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`
- `crate::core::utils::strip_ansi` participa do fallback textual de Vitest. 🟢 `src/cmds/js/vitest_cmd.rs`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Tornar a degradacao parte do tipo de retorno, e nao um log lateral. | `src/parser/mod.rs` | 🟢 |
| Aplicar truncamento por caracteres, evitando corte em meio a codigo UTF-8. | `src/parser/mod.rs` | 🟢 |
| Localizar JSON por marcador de Vitest antes da heuristica generica. | `src/parser/mod.rs` | 🟢 |
| Expor `warnings` como vetor clonado apenas para `Degraded`. | `src/parser/mod.rs` | 🟢 |

## Estado Interno e Observabilidade

- A unit nao mantem estado persistente; `ParseResult`, vetores de avisos e buffers de truncamento vivem na chamada. 🟢 `src/parser/mod.rs`
- `emit_degradation_warning` e `emit_passthrough_warning` escrevem marcadores em stderr quando chamados pelos consumidores. 🟢 `src/parser/mod.rs`, `src/cmds/js/vitest_cmd.rs`

## Riscos e Lacunas

- 🔴 Nao ha metricas persistidas de tier ou taxa de degradacao na implementacao atual.
- 🟡 A heuristica de procurar a primeira linha iniciada por `{` pode ignorar JSON valido que comece na mesma linha de um prefixo nao coberto.
