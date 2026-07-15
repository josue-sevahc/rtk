# Pipeline de Filtros TOML, Design Tecnico

> Reconstrucao de `src/core/toml_filter.rs`. 🟢 Confirmado no codigo legado; 🟡 inferido; 🔴 lacuna.

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `TomlFilterRegistry::load` | `()` | `TomlFilterRegistry` | Carrega fontes confiaveis e built-ins. 🟢 `src/core/toml_filter.rs:185` |
| `TomlFilterRegistry::parse_and_compile` | `(&str, &str)` | `Result<Vec<CompiledFilter>, String>` | Valida schema e compila definicoes. 🟢 `src/core/toml_filter.rs:225` |
| `find_matching_filter` | `(&str, &[CompiledFilter])` | `Option<&CompiledFilter>` | Seleciona o primeiro filtro correspondente. 🟢 `src/core/toml_filter.rs` |
| `apply_filter_with_info` | `(&CompiledFilter, &str)` | `(String, Lossiness)` | Aplica a pipeline e descreve perda. 🟢 `src/core/toml_filter.rs:515` |
| `toml_disabled` | `()` | `bool` | Consulta desabilitacao por ambiente. 🟢 `src/core/toml_filter.rs:402` |

## Fluxo Principal

1. `TomlFilterRegistry::load` pede os caminhos permitidos a `hooks::trust`. 🟢 `src/core/toml_filter.rs:191`
2. Para cada caminho existente, `extend_with_trusted` obtém status e conteudo; somente estados aceitos sao compilados. 🟢 `src/core/toml_filter.rs:203-222`
3. O registry acrescenta filtros built-in, registrando warning se sua compilacao falhar. 🟢 `src/core/toml_filter.rs:195-200`
4. A selecao encontra o primeiro filtro cujo padrao de comando casa; a ordem de carregamento define precedencia. 🟢 `src/core/toml_filter.rs`
5. `apply_filter_with_info` transforma o texto em linhas e executa: `strip_ansi`, `replace`, `match_output/unless`, filtro de linhas, truncamento, `head/tail`, `max_lines` e `on_empty`. 🟢 `src/core/toml_filter.rs:515-628`
6. O algoritmo devolve a saida e `None`, `Tail` ou `Whole`, para que o chamador escolha a estrategia de recuperacao. 🟢 `src/core/toml_filter.rs:631-650`

## Fluxos Alternativos

- **Arquivo ausente, nao confiavel ou alterado:** nao adiciona filtros externos. 🟢 `src/core/toml_filter.rs:203-222`
- **Schema ou definicao invalida:** retorna erro de compilacao ou warning por filtro; o registry segue disponivel. 🟢 `src/core/toml_filter.rs:225-243`
- **`match_output` com `unless`:** a regra que casa e ignorada quando a expressao de excecao tambem casa. 🟢 `src/core/toml_filter.rs:542-555`
- **Resultado vazio:** `on_empty`, se definido, substitui o texto vazio sem marcar perda. 🟢 `src/core/toml_filter.rs:623-628`

## Dependencias

- `hooks::trust`: fornece caminhos e decisao de confianca para filtros externos. 🟢 `src/core/toml_filter.rs:191`, `src/core/toml_filter.rs:208`
- `toml`, `serde`, `regex` e `regex::RegexSet`: parse e execucao das definicoes declarativas. 🟢 `src/core/toml_filter.rs:225`, `src/core/toml_filter.rs:130`
- `core::utils`: remove ANSI e trunca texto em limite seguro de Unicode. 🟢 `src/core/toml_filter.rs:518-523`, `src/core/toml_filter.rs:565-577`

## Decisoes de Design Identificadas

| Decisao | Evidencia | Confianca |
|---------|-----------|-----------|
| Falha de configuracao externa nao interrompe a CLI. | `src/core/toml_filter.rs:185-243` | 🟢 |
| A precedencia depende da ordem do registry, com fontes externas antes do catalogo built-in. | `src/core/toml_filter.rs:191-200` | 🟢 |
| `unless` evita que resumo por `match_output` esconda mensagens de erro. | `src/core/toml_filter.rs:542-555` | 🟢 |
| Perda e modelada explicitamente para viabilizar recuperacao em tee. | `src/core/toml_filter.rs:503-650` | 🟢 |

## Estado Interno

- `TomlFilterRegistry` mantem um vetor de `CompiledFilter` com regexes ja validadas. 🟢 `src/core/toml_filter.rs:181-200`
- Cada filtro mantem regras de substituicao, match global, selecao de linhas e limites de saida. 🟢 `src/core/toml_filter.rs:136-154`
- `Lossiness::Tail` carrega o payload e o offset; `Whole` declara que o texto completo e necessario para recuperar o conteudo removido. 🟢 `src/core/toml_filter.rs:503-513`

## Observabilidade

- Erros de parse de fontes e filtros invalidos sao emitidos como `[rtk] warning` com a origem e o nome do filtro. 🟢 `src/core/toml_filter.rs:195-197`, `src/core/toml_filter.rs:240`
- Testes inline cobrem compilacao do catalogo built-in e cenarios de `Lossiness`. 🟢 `src/core/toml_filter.rs:943-975`, `src/core/toml_filter.rs:1237-1243`

## Riscos e Lacunas

- 🟡 A ordem precisa entre todas as fontes externas depende de `hooks::trust::gated_filter_paths`, cuja implementacao pertence a outro modulo.
- 🔴 A analise estatica nao executou filtros de terceiros contra ferramentas reais do ambiente do usuario.
