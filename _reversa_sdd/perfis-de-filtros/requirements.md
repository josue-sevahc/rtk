# Perfis de Filtros

> Unit do modulo `filters`, reconstruida de `src/filters/`, `src/filters/README.md` e `build.rs`. Afirmacoes 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 exigem validacao humana.

## Visao Geral

Os perfis de filtros descrevem, em TOML, como reduzir ruido de saidas de ferramentas externas sem reformatar a saida para outro formato. 🟢 `src/filters/README.md` Cada perfil combina um comando por expressao regular com transformacoes declarativas e exemplos executaveis. 🟢 `src/filters/README.md`, `src/core/toml_filter.rs`

## Responsabilidades

- Manter perfis declarativos para comandos de sistema, linters, gerenciadores de pacote e ferramentas de infraestrutura. 🟢 `src/filters/*.toml`
- Associar cada perfil a um `match_command` e a uma ou mais acoes de filtragem da DSL TOML. 🟢 `src/filters/README.md`, `src/core/toml_filter.rs`
- Conservar exemplos `[[tests.<nome>]]` que definem a saida compactada esperada. 🟢 `src/filters/*.toml`, `src/core/toml_filter.rs`
- Seguir a convencao de um arquivo nomeado pela ferramenta ou subcomando para cada perfil. 🟢 `src/filters/README.md`
- Delimitar o conteudo do catalogo; carregamento trust-gated, selecao e aplicacao da DSL pertencem a `nucleo/pipeline-de-filtros-toml`. 🟢 `src/core/toml_filter.rs`, `_reversa_sdd/nucleo/pipeline-de-filtros-toml/`

## Regras de Negocio

- O catalogo embutido contem 63 perfis no estado analisado. 🟢 `src/filters/*.toml`, `src/core/toml_filter.rs:1895-1902`
- O perfil deve preservar uma saida reconhecivel da ferramenta, removendo ruido em vez de produzir uma apresentacao nova. 🟢 `src/filters/README.md`
- Perfis de log previsivel, linha a linha, sao candidatos ao TOML; saidas que exigem parsing estrutural ou logica complexa pertencem a implementacoes Rust especializadas. 🟢 `src/filters/README.md`, `src/cmds/jvm/mvn_cmd.rs`
- Cada exemplo inline descreve entrada e saida literal esperada para a verificacao do catalogo. 🟢 `src/filters/README.md`, `src/core/toml_filter.rs:657-799`
- A equivalencia de cada expressao regular com versoes, idiomas e formatos reais das ferramentas externas nao foi exercitada na extracao. 🔴

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de aceite |
|---|---|---|---|
| RF-01 | Definir um perfil TOML por ferramenta ou subcomando coberto, com nome, descricao, `match_command` e acoes declarativas. 🟢 | Must | Um comando de exemplo casa com a regex de seu perfil e o perfil pode ser desserializado pelo schema. |
| RF-02 | Suportar os campos documentados para remover, reter, substituir, truncar e limitar linhas, incluindo `on_empty` e `filter_stderr` quando aplicaveis. 🟢 | Must | Perfis representativos usam somente campos aceitos e produzem a saida esperada. |
| RF-03 | Registrar testes inline para cada perfil embutido. 🟢 | Must | A verificacao do catalogo nao aponta perfis sem teste. |
| RF-04 | Manter a convencao de nome de arquivo baseada no comando e de perfis pequenos por responsabilidade. 🟢 | Should | Um novo perfil pode ser localizado pelo nome do comando sem alterar perfis nao relacionados. |
| RF-05 | Priorizar filtros que reduzam ruido de forma significativa e preservem diagnosticos relevantes. 🟡 | Should | Fixtures de entrada mantem linhas de erro, resumo ou estado operacional necessario. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Manutenibilidade | A separacao em arquivos TOML pequenos torna revisao e adicao de perfis localizadas. | `src/filters/README.md`, `src/filters/*.toml` | 🟢 |
| Determinismo | A ordem do catalogo e definida pela ordenacao alfabetica dos nomes de arquivo no build. | `build.rs:20-23` | 🟢 |
| Corretude | O catalogo completo precisa ser TOML valido e seus perfis precisam compilar antes de uso. | `build.rs:40-51`, `src/core/toml_filter.rs:1237-1243` | 🟢 |
| Economia de contexto | A meta de ao menos 60% de economia e uma diretriz de contribuicao, nao uma metrica verificada para cada perfil. | `src/filters/README.md` | 🟡 |

## Criterios de Aceitacao

```gherkin
Cenario: Compactar um plano Terraform
  Dado o perfil terraform-plan e uma saida com linhas de refresh e bloqueio
  Quando o perfil for aplicado
  Entao as linhas de ruido sao removidas
  E o resumo do plano continua visivel

Cenario: Verificar o catalogo embutido
  Dado todos os perfis em src/filters
  Quando a verificacao inline for executada
  Entao cada perfil possui ao menos um teste
  E cada exemplo produz exatamente a saida esperada

Cenario: Saida em formato nao coberto
  Dado uma versao externa cuja saida nao corresponde aos exemplos
  Quando o perfil for aplicado
  Entao a fidelidade do resultado requer validacao humana
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Perfil valido com regex e acoes declarativas | Must | Sem ele o catalogo nao oferece comportamento para o comando. 🟢 |
| Teste inline por perfil | Must | A suite exige que todos os filtros embutidos sejam testados. 🟢 `src/core/toml_filter.rs:1909-1930` |
| Convencao de arquivos por comando | Should | Mantem o catalogo navegavel e extensivel. 🟢 |
| Meta quantitativa de economia por perfil | Could | O codigo nao a mede automaticamente. 🔴 |

## Rastreabilidade de Codigo

| Arquivo | Cobertura | Confianca |
|---|---|---|
| `src/filters/*.toml` | 63 definicoes de perfis e seus exemplos inline. | 🟢 |
| `src/filters/README.md` | Contrato de autoria, campos e criterios de escolha TOML versus Rust. | 🟢 |
| `build.rs` | Descoberta, ordenacao, concatenacao e validacao do catalogo. | 🟢 |
| `src/core/toml_filter.rs` | Schema aceito, compilacao dos perfis e executor de testes inline. | 🟢 |

## Lacunas

- 🔴 Validar os perfis contra as versoes e locales das ferramentas instaladas pelos usuarios.
- 🟡 Definir uma medicao automatizada que comprove a economia de tokens por perfil sem ocultar diagnosticos importantes.
