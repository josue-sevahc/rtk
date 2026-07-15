# Pipeline de Filtros TOML

> Contrato operacional do caso de uso `core::toml_filter`. 🟢 Confirmado no codigo legado; 🟡 inferido; 🔴 lacuna.

## Visao Geral

O pipeline localiza filtros TOML built-in e externos confiaveis, compila suas regras e transforma a saida de ferramentas em uma sequencia deterministica de estagios. 🟢 `src/core/toml_filter.rs:185`, `src/core/toml_filter.rs:515`

## Responsabilidades

- Carregar filtros de caminhos submetidos ao gate de confianca e os filtros built-in. 🟢 `src/core/toml_filter.rs:185-200`
- Recusar conteudo externo nao confiavel ou alterado sem interromper a execucao. 🟢 `src/core/toml_filter.rs:203-222`
- Validar `schema_version`, compilar regexes e avisar sobre definicoes invalidas. 🟢 `src/core/toml_filter.rs:225-243`
- Aplicar oito estagios de transformacao e informar `Lossiness`. 🟢 `src/core/toml_filter.rs:515-650`

## Regras de Negocio

- Fontes externas so entram quando classificadas como `Trusted` ou `EnvOverride`. 🟢 `src/core/toml_filter.rs:208-222`
- Filtros built-in permanecem disponiveis mesmo se uma fonte externa falhar. 🟢 `src/core/toml_filter.rs:195-198`
- `match_output` usa a primeira regra que casa, exceto quando seu `unless` tambem casa. 🟢 `src/core/toml_filter.rs:542-555`
- `strip_lines_matching` e `keep_lines_matching` sao mutuamente exclusivos. 🟢 `src/core/toml_filter.rs:558-563`
- Perdas por `head_lines` ou `max_lines` podem ser recuperaveis como `Tail`; cortes nao contiguos ou intra-linha sao `Whole`. 🟢 `src/core/toml_filter.rs:631-648`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|------------|-------------------|
| RF-01 | Carregar filtros confiaveis e built-in em registry ordenado. 🟢 | Must | Dado um arquivo confiavel valido, quando o registry carregar, entao seus filtros e os built-ins ficam disponiveis. |
| RF-02 | Ignorar fontes nao confiaveis e definicoes invalidas sem panicar. 🟢 | Must | Dado filtro alterado ou TOML invalido, quando carregar, entao ele nao e ativado e a execucao permanece disponivel. |
| RF-03 | Aplicar a DSL na ordem fixa observada. 🟢 | Must | Dado um filtro com varios estagios, quando aplicado, entao o resultado respeita a sequencia de oito etapas. |
| RF-04 | Devolver a classificacao de perda junto da saida. 🟢 | Must | Dado truncamento simples por head ou max, quando houver cauda recuperavel, entao o resultado retorna `Lossiness::Tail`. |
| RF-05 | Permitir desabilitar a busca TOML no caminho de fallback. 🟢 | Should | Dado `RTK_NO_TOML`, quando uma ferramenta externa for invocada, entao nenhum filtro TOML e selecionado. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia | Confianca |
|------|--------------------|-----------|-----------|
| Robustez | Erros de parse e de compilacao sao convertidos em warnings, nao panics. | `src/core/toml_filter.rs:185-243` | 🟢 |
| Seguranca | Conteudo de projeto so influencia a filtragem apos verificacao de confianca externa. | `src/core/toml_filter.rs:191-222` | 🟢 |
| Fidelidade | `Lossiness` indica se a informacao removida pode ser recuperada. | `src/core/toml_filter.rs:503-650` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Carregar filtro externo confiavel
  Dado um arquivo TOML confiavel e valido
  Quando o registry e inicializado
  Entao seus filtros ficam ativos antes dos filtros built-in

Cenario: Ignorar filtro externo nao confiavel
  Dado um arquivo TOML cujo conteudo mudou apos a aprovacao
  Quando o registry e inicializado
  Entao o arquivo nao adiciona filtros ativos
  E o comando continua executavel

Cenario: Classificar truncamento recuperavel
  Dado um filtro com head_lines menor que a quantidade de linhas
  Quando o filtro e aplicado
  Entao a saida informa as linhas omitidas
  E o resultado e classificado como Lossiness::Tail
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Registry confiavel e DSL deterministica | Must | Sustentam o fallback filtrado e a seguranca de filtros locais. 🟢 |
| Classificacao de perda | Must | Permite ao chamador preservar recuperabilidade por tee. 🟢 |
| Desabilitacao por ambiente | Should | Oferece compatibilidade e diagnostico quando TOML nao deve atuar. 🟢 |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/core/toml_filter.rs` | `TomlFilterRegistry::load`, `extend_with_trusted` | 🟢 |
| `src/core/toml_filter.rs` | `parse_and_compile`, `find_matching_filter` | 🟢 |
| `src/core/toml_filter.rs` | `apply_filter_with_info`, `Lossiness` | 🟢 |
| `src/hooks/trust.rs` | gate de confianca consumido pelo registry | 🟢 |
