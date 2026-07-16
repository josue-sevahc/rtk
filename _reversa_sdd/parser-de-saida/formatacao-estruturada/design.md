# Formatacao Estruturada, Design Tecnico

> Design reconstruido de `src/parser/formatter.rs`. 🟢 confirmado no codigo; 🟡 inferido; 🔴 lacuna.

## Interface

| Simbolo | Entrada | Saida | Observacao |
|---|---|---|---|
| `FormatMode::from_verbosity` | `u8` | `FormatMode` | Seleciona Compact, Verbose ou Ultra. 🟢 |
| `TokenFormatter::format_compact` | `&self` | `String` | Resumo token-efficient. 🟢 |
| `TokenFormatter::format_verbose` | `&self` | `String` | Detalhes ampliados. 🟢 |
| `TokenFormatter::format_ultra` | `&self` | `String` | Siglas e simbolos. 🟢 |
| `TokenFormatter::format` | `&self`, `FormatMode` | `String` | Despacha para o metodo do modo. 🟢 |
| `TokenFormatter for TestResult` | dados de testes | `String` | Implementa os tres modos. 🟢 |
| `TokenFormatter for DependencyState` | dados de dependencias | `String` | Implementa os tres modos. 🟢 |

## Fluxo Principal

1. O consumidor converte `verbose: u8` em `FormatMode` por `from_verbosity`. 🟢 `src/parser/formatter.rs`
2. Com um `TestResult` ou `DependencyState` de tier 1 ou 2, chama `format(mode)`. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`
3. O trait despacha para `format_compact`, `format_verbose` ou `format_ultra`. 🟢 `src/parser/formatter.rs`
4. O texto resultante segue para exibicao no comando; alguns consumidores ainda aplicam guardas para nao piorar a saida original. 🟢 `src/cmds/js/pnpm_cmd.rs`

## Fluxos Alternativos

- **Testes sem falhas:** o compacto fica curto; a duracao entra somente quando disponivel. 🟢 `src/parser/formatter.rs`
- **Mais de cinco falhas:** o compacto mostra as cinco primeiras e `... +N more failures`. 🟢 `src/parser/formatter.rs`
- **Listagem simples de pacotes:** quando todas as dependencias nao possuem `latest_version`, o compacto mostra nome, versao e marcador dev. 🟢 `src/parser/formatter.rs`
- **Nenhuma dependencia desatualizada:** fora do caso de listagem simples, o compacto retorna `All packages up-to-date`. 🟢 `src/parser/formatter.rs`
- **Dependencia com versao wanted divergente:** o detalhado acrescenta a versao desejada quando ela difere da mais recente. 🟢 `src/parser/formatter.rs`

## Regras de Renderizacao

| Dado | Compact | Verbose | Ultra |
|---|---|---|---|
| `TestResult` | Contadores, ate cinco falhas e duracao. 🟢 | Contadores totais, todas as falhas, arquivo e preview de stack. 🟢 | `[ok]`, `[x]`, `[skip]` e duracao. 🟢 |
| `DependencyState` | Lista simples ou ate dez atualizacoes. 🟢 | Totais e cada atualizacao com detalhe wanted. 🟢 | `pkg:<total> ^<outdated>`. 🟢 |

## Dependencias

- `TestResult`, `TestFailure`, `DependencyState` e `Dependency` sao definidos em `src/parser/types.rs`. 🟢
- `CAP_INVENTORY` limita `MAX_DEPS_LISTING` em listagem simples. 🟢 `src/parser/formatter.rs`, `src/core/truncate.rs`
- Os comandos JS usam o trait depois do parse. 🟢 `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Nao ocultar testes ignorados no resumo compacto. | `src/parser/formatter.rs` | 🟢 |
| Preservar toda a mensagem das falhas compactas exibidas, mesmo multiline. | `src/parser/formatter.rs` | 🟢 |
| Tratar uma listagem sem latest como inventario, nao como sucesso de atualizacao. | `src/parser/formatter.rs` | 🟢 |
| Limitar stack trace detalhado a tres linhas. | `src/parser/formatter.rs` | 🟢 |

## Estado Interno e Observabilidade

- A formatacao nao persiste estado; constroi vetores locais de linhas e retorna `String`. 🟢 `src/parser/formatter.rs`
- Nao ha logs ou metricas emitidos pelo formatador em si. 🟢 `src/parser/formatter.rs`

## Riscos e Lacunas

- 🟡 O modo Ultra usa marcadores ASCII, apesar de o README citar simbolos; a compatibilidade desejada de apresentacao precisa de validacao humana.
- 🟢 Codigos, chaves, enums e schemas permanecem em ingles como contrato canonico; somente a apresentacao humana pode ser localizada, com `en-US` obrigatorio e `pt-BR` como primeiro locale adicional. Decisao do usuario em 2026-07-16.
