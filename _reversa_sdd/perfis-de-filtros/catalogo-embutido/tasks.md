# Catalogo Embutido de Filtros, Tarefas de Implementacao

> Sequencia para reconstruir a montagem e as garantias do catalogo built-in.

## Pre-requisitos

- [ ] 🟢 Manter os perfis-fonte em um diretorio conhecido pelo processo de build.
- [ ] 🟢 Disponibilizar parser TOML tanto no build quanto no runtime.
- [ ] 🟢 Disponibilizar uma forma de incorporar texto gerado pelo build ao binario.

## Tarefas

- [ ] T-01, Descobrir arquivos `.toml` no diretorio de perfis e ordenar seus nomes alfabeticamente. Confianca: 🟢
  - Origem no legado: `build.rs:12-23`.
  - Criterio de pronto: a mesma arvore de fontes produz a mesma sequencia de entradas em qualquer execucao.

- [ ] T-02, Construir o documento combinado com `schema_version = 1` e marcadores de origem por arquivo. Confianca: 🟢
  - Origem no legado: `build.rs:25-37`.
  - Criterio de pronto: o artefato possui schema valido e permite identificar a origem de cada trecho concatenado.

- [ ] T-03, Validar TOML combinado e interromper o build quando ele for invalido. Confianca: 🟢
  - Origem no legado: `build.rs:40-44`.
  - Criterio de pronto: uma fixture malformada impede a escrita do catalogo e devolve erro de parse.

- [ ] T-04, Gravar `builtin_filters.toml` no diretorio de saida do build e inclui-lo como texto no binario. Confianca: 🟢
  - Origem no legado: `build.rs:15-16`, `build.rs:51`, `src/core/toml_filter.rs:31-32`.
  - Criterio de pronto: o registry compila os perfis sem consultar arquivos built-in no disco em runtime.

- [ ] T-05, Manter testes de inventario para schema, nomes esperados, contagem e exemplos inline. Confianca: 🟢
  - Origem no legado: `src/core/toml_filter.rs:1821-1930`.
  - Criterio de pronto: remover um perfil, alterar a contagem ou omitir testes faz a suite falhar com causa identificavel.

## Tarefas de Teste

- [ ] TT-01, Testar ordenacao deterministica e inclusao de todos os `.toml` encontrados. Confianca: 🟢 `build.rs:14-37`
- [ ] TT-02, Testar falha de build para TOML combinado invalido. Confianca: 🟢 `build.rs:40-44`
- [ ] TT-03, Testar que o catalogo tem schema 1, compila integralmente e contem os 63 perfis desta revisao. Confianca: 🟢 `src/core/toml_filter.rs:1237-1243`, `src/core/toml_filter.rs:1821-1907`
- [ ] TT-04, Testar que todos os perfis built-in possuem e passam por ao menos um exemplo inline. Confianca: 🟢 `src/core/toml_filter.rs:1909-1930`
- [ ] TT-05, Executar testes de compatibilidade contra as ferramentas externas suportadas e suas versoes declaradas. Confianca: 🔴

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 O artefato de catalogo e derivado das fontes TOML durante o build e nao armazena dados de usuario.

## Ordem Sugerida

1. Implementar T-01 e T-02 para estabelecer uma composicao deterministica.
2. Implementar T-03 antes de expor o artefato ao compilador Rust.
3. Implementar T-04 e T-05 para fechar a cadeia de embutimento e suas protecoes de regressao.

## Lacunas Pendentes (🔴)

- 🟢 A matriz certificada nasce apenas de combinacoes com fixture ou teste; versoes e locales sem cobertura permanecem experimentais. Decisao do usuario em 2026-07-16.
- Definir politica de atualizacao da sentinela de contagem quando novos perfis forem adicionados.
