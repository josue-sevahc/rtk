# Pipeline de Filtros TOML, Tarefas de Implementacao

## Pre-requisitos

- [ ] 🟢 Disponibilizar parser TOML, serializacao e regexes compilaveis. Origem: `src/core/toml_filter.rs`.
- [ ] 🟢 Integrar uma fonte de caminhos e status de confianca para filtros externos. Origem: `src/core/toml_filter.rs:191-222`.

## Tarefas

- [ ] T-01, Modelar definicoes TOML, filtros compilados e validacao de `schema_version`.
  - Origem no legado: `src/core/toml_filter.rs:100-154`, `src/core/toml_filter.rs:225-243`.
  - Criterio de pronto: schema invalido, campo desconhecido ou regex invalida retorna erro compreensivel sem panic.
  - Confianca: 🟢

- [ ] T-02, Construir registry com fontes confiaveis e catalogo built-in em ordem deterministica.
  - Origem no legado: `src/core/toml_filter.rs:185-222`.
  - Criterio de pronto: somente conteudo `Trusted` ou `EnvOverride` e incorporado; built-ins permanecem carregados diante de falha externa.
  - Confianca: 🟢

- [ ] T-03, Implementar lookup de filtro e opcao de desabilitacao TOML para o caminho chamador.
  - Origem no legado: `src/core/toml_filter.rs:399-402`, `src/core/toml_filter.rs`.
  - Criterio de pronto: um comando seleciona apenas o primeiro filtro correspondente e a configuracao de desabilitacao evita a busca.
  - Confianca: 🟢

- [ ] T-04, Implementar a pipeline de oito estagios de transformacao.
  - Origem no legado: `src/core/toml_filter.rs:515-628`.
  - Criterio de pronto: o resultado respeita a ordem de ANSI, substituicoes, match/unless, linhas, truncamento, head/tail, max e on_empty.
  - Confianca: 🟢

- [ ] T-05, Implementar `Lossiness` e a classificacao de perdas recuperaveis e integrais.
  - Origem no legado: `src/core/toml_filter.rs:503-513`, `src/core/toml_filter.rs:631-650`.
  - Criterio de pronto: head/max contiguo produz `Tail` quando aplicavel; corte intra-linha ou nao contiguo produz `Whole`.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar gate de confianca, schema invalido e filtros built-in compilaveis.
  - Origem no legado: `src/core/toml_filter.rs:1193-1243`.
  - Criterio de pronto: fontes invalidas nao derrubam o registry e catalogo built-in compila por completo.
  - Confianca: 🟢

- [ ] TT-02, Testar `match_output` com e sem `unless`, selecao de linhas e `on_empty`.
  - Origem no legado: `src/core/toml_filter.rs:542-628`.
  - Criterio de pronto: mensagens de erro protegidas por `unless` nao sao ocultadas e saida vazia respeita fallback configurado.
  - Confianca: 🟢

- [ ] TT-03, Testar todos os estados de `Lossiness`.
  - Origem no legado: `src/core/toml_filter.rs:943-975`.
  - Criterio de pronto: a classificacao coincide com o tipo de perda introduzida pelo filtro.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 O caso de uso consome arquivos de configuracao e nao possui schema persistente proprio.

## Ordem Sugerida

1. Implementar T-01 e T-02 para garantir definicoes validas e fontes confiaveis.
2. Implementar T-03 antes de expor o registry ao fallback.
3. Implementar T-04 e T-05 juntos, pois a classificacao depende da pipeline final.

## Lacunas Pendentes (🔴)

- 🔴 Validar compatibilidade de regexes e filtros customizados reais com as ferramentas instaladas pelos usuarios.
