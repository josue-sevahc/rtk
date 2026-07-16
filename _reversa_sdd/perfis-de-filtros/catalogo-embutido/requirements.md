# Catalogo Embutido de Filtros

> Subunit de `perfis-de-filtros`, reconstruida de `src/filters/`, `build.rs` e dos testes de `src/core/toml_filter.rs`. Afirmacoes 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 exigem validacao humana.

## Visao Geral

O catalogo embutido transforma os arquivos TOML versionados em um unico documento com `schema_version = 1`, gerado durante a compilacao e incorporado ao binario. 🟢 `build.rs`, `src/core/toml_filter.rs:31-32` Ele torna disponiveis 63 perfis built-in sem depender de arquivos instalados na maquina do usuario. 🟢 `src/core/toml_filter.rs:1895-1902`

## Responsabilidades

- Descobrir todos os arquivos `.toml` em `src/filters/`. 🟢 `build.rs:14-19`
- Ordenar os arquivos alfabeticamente para uma composicao reproduzivel. 🟢 `build.rs:20-23`
- Injetar a versao de schema, concatenar o conteudo e gravar `builtin_filters.toml` em `OUT_DIR`. 🟢 `build.rs:25-47`
- Reprovar no build um catalogo TOML combinado invalido. 🟢 `build.rs:40-44`
- Garantir via testes que os perfis esperados estao presentes, totalizam 63 e possuem exemplos inline. 🟢 `src/core/toml_filter.rs:1821-1930`

## Regras de Negocio

- O catalogo deve ser remontado quando `src/filters/` mudar. 🟢 `build.rs:12`
- O artefato combinado deve comecar com `schema_version = 1`. 🟢 `build.rs:25`, `src/core/toml_filter.rs:1821-1828`
- A inclusao do catalogo no binario ocorre por `include_str!` a partir de `OUT_DIR`. 🟢 `src/core/toml_filter.rs:31-32`
- O total esperado nesta revisao e 63 filtros built-in. 🟢 `src/core/toml_filter.rs:1895-1902`
- A mudanca de quantidade ou nomes de filtros requer ajuste dos testes de inventario correspondentes. 🟢 `src/core/toml_filter.rs:1831-1907`
- A presenca no catalogo nao comprova, por si so, compatibilidade da ferramenta externa em runtime. 🔴

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de aceite |
|---|---|---|---|
| RF-01 | Descobrir e concatenar todos os TOMLs de `src/filters/` em um unico artefato de build. 🟢 | Must | O artefato contem cada perfil de origem uma vez. |
| RF-02 | Ordenar a composicao alfabeticamente pelo nome de arquivo. 🟢 | Must | Duas compilacoes com a mesma arvore produzem a mesma ordem de perfis. |
| RF-03 | Injetar o schema do documento combinado e validar TOML antes de gravar o artefato. 🟢 | Must | Conteudo invalido interrompe o build; conteudo valido gera `builtin_filters.toml`. |
| RF-04 | Embutir o resultado no binario para que os perfis built-in nao dependam de leitura de disco em runtime. 🟢 | Must | O registry consegue desserializar o texto incluido por `include_str!`. |
| RF-05 | Vigiar nomes, contagem e cobertura de testes inline do catalogo. 🟢 | Should | A suite detecta um perfil ausente, uma contagem inesperada ou um perfil sem teste. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Reprodutibilidade | A ordenacao explicita elimina a dependencia da ordem retornada pelo sistema de arquivos. | `build.rs:20-23` | 🟢 |
| Confiabilidade | A validacao do TOML combinado antecipa erro de configuracao para o build. | `build.rs:40-44` | 🟢 |
| Portabilidade | O catalogo e incorporado como texto no binario compilado. | `src/core/toml_filter.rs:31-32` | 🟢 |
| Manutenibilidade | Testes de presenca e de contagem tornam mudancas do catalogo explicitas. | `src/core/toml_filter.rs:1831-1907` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Gerar o catalogo embutido
  Dado uma arvore valida de arquivos TOML em src/filters
  Quando o build executar
  Entao builtin_filters.toml recebe schema_version 1
  E os perfis aparecem em ordem alfabetica de arquivo

Cenario: Detectar um perfil invalido
  Dado um TOML invalido no diretorio de filtros
  Quando o build concatenar o catalogo
  Entao a compilacao falha
  E a mensagem aponta para a revisao dos arquivos de filtros

Cenario: Proteger a cobertura do catalogo
  Dado um perfil embutido sem teste inline
  Quando a suite de verificacao for executada
  Entao o perfil aparece como sem teste
  E a validacao obrigatoria falha
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Concatenacao valida com schema | Must | E o unico caminho de disponibilidade dos perfis built-in. 🟢 |
| Ordenacao deterministica | Must | A precedencia do primeiro match depende da ordem. 🟢 |
| Inventario e testes inline | Should | Protegem evolucoes, mas nao montam o artefato em si. 🟢 |
| Medicao de compatibilidade runtime | Could | Exige ferramentas externas fora do build. 🔴 |

## Rastreabilidade de Codigo

| Arquivo | Cobertura | Confianca |
|---|---|---|
| `build.rs` | Descoberta, ordenacao, schema, concatenacao e validacao. | 🟢 |
| `src/filters/*.toml` | Conteudo-fonte do catalogo e fixtures inline. | 🟢 |
| `src/core/toml_filter.rs` | Inclusao do artefato e testes de integridade do catalogo. | 🟢 |
| `src/filters/README.md` | Contrato de contribuicao e fluxo de build. | 🟢 |

## Lacunas

- 🔴 Validar o catalogo contra ferramentas reais para detectar formatos que passaram a divergir dos exemplos versionados.
