# Perfis de Filtros, Design Tecnico

> Design reconstruido de `src/filters/`, `src/filters/README.md`, `build.rs` e `src/core/toml_filter.rs`. 🟢 confirmado no codigo; 🟡 inferido; 🔴 lacuna.

## Interface

| Elemento | Entrada | Saida | Observacao |
|---|---|---|---|
| Arquivo de perfil | TOML com `[filters.<nome>]` | Definicao desserializavel | O catalogo adota um arquivo por ferramenta ou subcomando. 🟢 `src/filters/README.md` |
| `description` | Texto opcional | Metadado humano | Descreve a finalidade do perfil. 🟢 `src/core/toml_filter.rs:80-108` |
| `match_command` | Expressao regular | Seletor do comando | Campo obrigatorio no schema. 🟢 `src/core/toml_filter.rs:80-108` |
| Acoes de linha e blob | Regexes, limites e mensagens | Saida compactada | Inclui ANSI, replace, match/unless, strip/keep, truncamento, head/tail, max e `on_empty`. 🟢 `src/core/toml_filter.rs:80-108` |
| `[[tests.<nome>]]` | `name`, `input`, `expected` | Caso verificavel | O executor compara a saida literal. 🟢 `src/core/toml_filter.rs:61-72`, `src/core/toml_filter.rs:657-799` |

## Fluxo Principal

1. Um contribuinte cria ou altera um arquivo `src/filters/<comando>.toml`, com perfil e testes inline. 🟢 `src/filters/README.md`
2. O `build.rs` encontra todos os `.toml` em `src/filters/` e os ordena alfabeticamente pelo nome de arquivo. 🟢 `build.rs:14-23`
3. O build adiciona `schema_version = 1`, concatena os arquivos com separadores e grava `builtin_filters.toml` em `OUT_DIR`. 🟢 `build.rs:25-47`
4. A validacao TOML do build interrompe a compilacao quando o catalogo combinado e invalido. 🟢 `build.rs:40-44`
5. O binario inclui o catalogo gerado e o registry compila cada definicao antes de disponibiliza-la ao caminho de fallback. 🟢 `src/core/toml_filter.rs:31-32`, `src/core/toml_filter.rs:225-243`
6. A verificacao percorre os testes inline, aplica o perfil e compara `actual` com `expected`. 🟢 `src/core/toml_filter.rs:657-799`

## Fluxos Alternativos

- **TOML combinado invalido:** o build falha com indicacao para revisar `src/filters/*.toml`. 🟢 `build.rs:40-44`
- **Regex ou definicao individual invalida:** a compilacao do perfil gera warning e aquele perfil nao entra no conjunto compilado. 🟢 `src/core/toml_filter.rs:236-241`
- **Perfil sem teste inline:** `run_filter_tests` o inclui em `filters_without_tests`; a suite embutida exige cobertura de todos os perfis. 🟢 `src/core/toml_filter.rs:774-799`, `src/core/toml_filter.rs:1909-1930`
- **Comando sem perfil correspondente:** a decisao de passthrough e responsabilidade do pipeline consumidor, fora desta unit. 🟢 `src/core/toml_filter.rs`, `_reversa_sdd/nucleo/pipeline-de-filtros-toml/`

## Dependencias

- `build.rs` usa `std::fs` e `toml::Value` para montar e validar o artefato de build. 🟢 `build.rs`
- `src/core/toml_filter.rs` define o schema de perfil, compila regexes e executa testes inline. 🟢 `src/core/toml_filter.rs`
- `src/core/utils.rs` oferece a remocao ANSI e o truncamento unicode-safe usados durante aplicacao. 🟢 `src/core/toml_filter.rs:515-628`
- `nucleo/pipeline-de-filtros-toml` e dependente funcional do catalogo, mas possui contrato separado para confianca e runtime. 🟢 `_reversa_sdd/nucleo/pipeline-de-filtros-toml/`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Usar TOML para ruido previsivel de linha, reservando Rust para logica rica. | `src/filters/README.md`, `src/cmds/jvm/mvn_cmd.rs` | 🟢 |
| Determinar a ordem por nome de arquivo, nao pela ordem do sistema de arquivos. | `build.rs:20-23` | 🟢 |
| Injetar uma unica versao de schema no artefato concatenado. | `build.rs:25-37` | 🟢 |
| Exigir testes inline para todo perfil embutido. | `src/core/toml_filter.rs:1909-1930` | 🟢 |
| Manter filtros pequenos e nomeados pelo comando para facilitar contribuicao. | `src/filters/README.md` | 🟢 |

## Estado Interno e Observabilidade

- O diretorio de perfis e estatico no repositorio; o catalogo combinado e um artefato temporario em `OUT_DIR` durante o build. 🟢 `build.rs`
- A lista compilada de perfis e carregada de forma lazy uma vez por processo pelo registry. 🟢 `src/core/toml_filter.rs:395-400`
- Falhas de compilacao de um perfil e perfis sem testes podem ser observados por warnings e pelo resultado da verificacao. 🟢 `src/core/toml_filter.rs:236-241`, `src/core/toml_filter.rs:657-799`

## Riscos e Lacunas

- 🔴 Os exemplos inline nao cobrem necessariamente todas as versoes, configuracoes e locales de cada ferramenta externa.
- 🟡 A ordenacao alfabetica torna precedencia reproduzivel, mas perfis sobrepostos podem exigir revisao humana para manter a intencao.
- 🔴 Nao ha no codigo uma medicao por perfil que confirme a economia de tokens prometida pela documentacao.
