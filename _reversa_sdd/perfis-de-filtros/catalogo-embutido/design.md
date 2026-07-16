# Catalogo Embutido de Filtros, Design Tecnico

> Design reconstruido de `build.rs` e `src/core/toml_filter.rs`. 🟢 confirmado no codigo; 🟡 inferido; 🔴 lacuna.

## Interface

| Simbolo / artefato | Entrada | Saida | Observacao |
|---|---|---|---|
| `build.rs` | `src/filters/*.toml` | `OUT_DIR/builtin_filters.toml` | Executa durante o build Cargo. 🟢 `build.rs:1-51` |
| `BUILTIN_TOML` | Arquivo gerado em `OUT_DIR` | `&'static str` | E incluido pelo binario com `include_str!`. 🟢 `src/core/toml_filter.rs:31-32` |
| `TomlFilterRegistry::parse_and_compile` | Texto TOML e rotulo de fonte | `Vec<CompiledFilter>` ou erro | Confere `schema_version` e compila cada definicao. 🟢 `src/core/toml_filter.rs:225-243` |
| `run_filter_tests` | Nome opcional de filtro | Casos e perfis sem teste | Executa os exemplos declarados no catalogo. 🟢 `src/core/toml_filter.rs:657-799` |

## Fluxo Principal

1. O build declara `cargo:rerun-if-changed=src/filters`, para que alteracoes no diretorio invalidem o artefato. 🟢 `build.rs:12`
2. O processo le as entradas de `src/filters`, mantem somente extensao `.toml` e ordena os `DirEntry` pelo nome. 🟢 `build.rs:14-23`
3. Inicializa o texto combinado com `schema_version = 1` e acrescenta cada arquivo precedido por um comentario de separacao. 🟢 `build.rs:25-37`
4. O texto combinado e parseado como `toml::Value`; falha de parse aborta o build. 🟢 `build.rs:40-44`
5. O build percorre a tabela `filters` para vigiar nomes repetidos e grava o texto final em `OUT_DIR/builtin_filters.toml`. 🟢 `build.rs:46-51`
6. Em compilacao Rust, `BUILTIN_TOML` inclui esse arquivo; o registry o parseia e compila seus perfis. 🟢 `src/core/toml_filter.rs:31-32`, `src/core/toml_filter.rs:195-198`

## Fluxos Alternativos

- **Diretorio de filtros ausente ou arquivo ilegivel:** o build encerra com panic e mensagem de contexto. 🟢 `build.rs:14-17`, `build.rs:31-33`
- **TOML invalido:** o parse do documento combinado encerra o build antes de gravar o artefato. 🟢 `build.rs:40-44`
- **Schema diferente de 1:** o registry devolve erro e nao aceita o catalogo como fonte compilavel. 🟢 `src/core/toml_filter.rs:225-234`
- **Definicao individual invalida:** o registry emite warning e mantem os demais perfis compilados. 🟢 `src/core/toml_filter.rs:236-241`
- **Perfil esperado ausente ou contagem diferente:** os testes de inventario falham. 🟢 `src/core/toml_filter.rs:1831-1907`

## Dependencias

- `std::fs`, `std::path::Path` e `std::collections::HashSet` estruturam o build do catalogo. 🟢 `build.rs`
- A crate `toml` valida a representacao combinada no build e desserializa o schema no runtime. 🟢 `build.rs:40-44`, `src/core/toml_filter.rs:225-228`
- O `OUT_DIR` fornecido pelo Cargo armazena o artefato intermediario consumido por `include_str!`. 🟢 `build.rs:15-16`, `src/core/toml_filter.rs:31-32`
- Os testes do modulo `toml_filter` definem o inventario esperado e exercitam a compilacao. 🟢 `src/core/toml_filter.rs:1237-1243`, `src/core/toml_filter.rs:1821-1930`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Gerar o catalogo no build em vez de ler cada perfil do disco no runtime. | `build.rs`, `src/core/toml_filter.rs:31-32` | 🟢 |
| Prefixar o documento combinado com uma versao unica de schema. | `build.rs:25` | 🟢 |
| Ordenar fontes para tornar o artefato e a precedencia reproduziveis. | `build.rs:20-23` | 🟢 |
| Tratar mudancas de nomes ou quantidade como contrato de teste explicito. | `src/core/toml_filter.rs:1831-1907` | 🟢 |

## Estado Interno e Observabilidade

- `builtin_filters.toml` e efemero em `OUT_DIR`; a fonte de verdade permanece nos TOMLs versionados. 🟢 `build.rs`
- O valor `BUILTIN_TOML` e uma constante textual do binario compilado. 🟢 `src/core/toml_filter.rs:31-32`
- Erros de build exibem o erro TOML; falhas de definicoes no registry sao escritas como warnings em stderr. 🟢 `build.rs:40-44`, `src/core/toml_filter.rs:236-241`

## Riscos e Lacunas

- 🟡 O limite de 63 filtros e uma sentinela da revisao analisada e deve ser atualizado conscientemente quando o catalogo evoluir.
- 🔴 Nao ha verificacao de integracao que execute todos os perfis contra os binarios externos reais durante o build.
