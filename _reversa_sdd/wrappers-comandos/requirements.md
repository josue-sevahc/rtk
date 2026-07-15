# Wrappers de Comandos

> Contrato operacional do modulo `cmds`, que executa ferramentas externas e reduz a saida entregue ao agente. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Visao Geral

🟢 A unit organiza wrappers por ecossistema para ferramentas de sistema, VCS, cloud, Rust, JavaScript, Python, PHP, Ruby, Go, .NET e JVM. Cada wrapper monta a chamada da ferramenta, seleciona uma estrategia de filtragem, preserva o resultado de execucao e registra a economia observada.

🟡 O modulo e a principal fronteira de adaptacao entre formatos de CLIs externas e uma representacao compacta para agentes, sem se tornar dono da infraestrutura compartilhada de processos, tracking ou filtros TOML.

## Responsabilidades

- 🟢 Executar ferramentas externas por `resolved_command` e runners de `core`, preservando argumentos e status observavel.
- 🟢 Escolher entre captura estruturada, filtragem buffered, streaming linha a linha, maquina de estados, passthrough ou execucao manual conforme o formato da ferramenta.
- 🟢 Usar JSON, XML, NDJSON, binlog ou TRX quando a ferramenta disponibiliza formato estruturado adequado.
- 🟢 Compactar output textual por parsers especializados sem ocultar erros, warnings ou falhas de teste relevantes.
- 🟢 Oferecer tee/hint de recuperacao quando a compactacao remove blocos ou detalhes importantes.
- 🟢 Roteiar ferramentas entre ecossistemas quando a deteccao do projeto ou do comando exigir essa adaptacao.

## Regras de Negocio

- 🟢 Um modulo pertence a `cmds` quando executa ferramenta externa e filtra sua saida; infraestrutura compartilhada sem execucao pertence a `core`.
- 🟢 Modulos Rust especializados existem quando o filtro exige parsing estruturado, maquina de estados, injecao de flags, roteamento cross-command ou comportamento sensivel a flags.
- 🟢 O wrapper deve preservar o exit code real da ferramenta externa; sucesso de compactacao nao pode converter falha externa em sucesso.
- 🟢 Flags que solicitam formato cru, JSON ou formato de saida especifico podem acionar passthrough para evitar dupla injecao ou transformacao incorreta.
- 🟢 Saida truncada ou blocos omitidos devem receber mecanismo de recuperacao por tee quando aplicavel.
- 🟢 Falha de parsing estruturado deve degradar para saida bruta, passthrough ou filtro explicitamente degradado, sem retornar dados falsos silenciosamente.
- 🟢 A saida emitida, incluindo hints, nao pode exceder a saida bruta em tokens estimados; quando excede, a referencia bruta prevalece.
- 🟢 Passthrough nao captura output para metricas de tokens e usa registro neutro, para nao diluir os indicadores de economia.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|-----------|-------------------|
| RF-01 | 🟢 O modulo deve expor wrappers por ecossistema para system, git, rust, js, python, go, dotnet, cloud, php, ruby e jvm. | Must | Dado um comando roteado pela CLI, quando seu ecossistema e reconhecido, entao o wrapper correspondente recebe subcomando, argumentos e nivel de verbosidade. |
| RF-02 | 🟢 Cada wrapper deve executar a ferramenta externa e devolver seu exit code real por `Result<i32>`. | Must | Dado um filho que retorna sucesso ou falha, quando o wrapper conclui, entao o codigo devolvido coincide com o status do filho. |
| RF-03 | 🟢 O modulo deve escolher filtragem buffered/capturada quando o parser precisa do output completo, e streaming quando a saida e longa ou parseavel por linha/bloco. | Must | Dado output JSON/tabela, quando o wrapper exige estrutura completa, entao usa captura; dado log longo com blocos, entao linhas relevantes sao emitidas progressivamente. |
| RF-04 | 🟢 O modulo deve injetar ou forcar formato estruturado somente quando a ferramenta e o subcomando permitem, e analisar JSON/XML/NDJSON/binlog/TRX conforme a familia. | Must | Dado wrapper configurado para formato estruturado, quando o comando e suportado, entao argumentos de formato sao aplicados e o parser extrai a representacao compacta; dado parsing invalido, entao ocorre degradacao segura. |
| RF-05 | 🟢 O modulo deve suportar filtros de texto com state machines ou handlers de bloco para ferramentas sem formato estruturado confiavel. | Must | Dado Maven, Gradle, Cargo, Pytest ou saida similar, quando a ferramenta produz blocos de falha, entao o wrapper preserva blocos e resumo relevantes sem depender de JSON inexistente. |
| RF-06 | 🟢 O modulo deve encaminhar flags ou subcomandos desconhecidos e pedidos de formato especial por passthrough quando a filtragem puder alterar a semantica solicitada. | Must | Dado uma flag de formato cru/JSON/output ou subcomando nao suportado, quando a ferramenta e executada, entao seus streams e argumentos seguem sem transformacao inadequada. |
| RF-07 | 🟢 O modulo deve anexar hint de recuperacao quando a compactacao for lossy e manter a saida bruta quando a forma compacta+hints nao for vantajosa. | Must | Dado output truncado ou bloco omitido, quando o filtro conclui, entao existe caminho de recuperacao; dado output compacto maior que raw, entao a saida visivel e raw. |
| RF-08 | 🟢 O modulo deve registrar tracking raw versus filtrado para execucoes capturadas ou streamed e registrar passthrough com tokens neutros. | Should | Dado uma execucao filtrada, quando ela termina, entao tracking recebe raw e texto efetivo; dado passthrough, entao tracking nao reduz artificialmente as metricas. |
| RF-09 | 🟢 O modulo deve implementar adaptacoes conhecidas por ferramenta, incluindo filtros Git/forjas, Cargo, AWS, .NET, Maven/Gradle, package managers JS, ferramentas Python/PHP/Ruby/Go e comandos de sistema. | Should | Dado um caso coberto por cada familia, quando o wrapper roda, entao sua regra especifica de parsing, injecao, fallback ou roteamento e aplicada. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Compatibilidade | 🟢 Exit codes e semantica de argumentos de ferramentas externas precisam permanecer observaveis apos filtragem. | `src/cmds/README.md`, `src/core/runner.rs` | 🟢 |
| Desempenho | 🟢 Saida longa deve poder ser processada em streaming para reduzir buffer e disponibilizar informacao progressivamente. | `src/cmds/README.md`, `core::stream::FilterMode::Streaming` | 🟢 |
| Fidelidade | 🟢 A representacao final passa pelo guard de tokens e usa tee para recuperacao quando ha perda. | `src/cmds/README.md`, `src/core/runner.rs`, ADR 001 | 🟢 |
| Robustez | 🟢 Erro de parsing estruturado nao deve inventar dados; a degradacao preserva output ou usa caminho explicitamente menos compacto. | `_reversa_sdd/code-analysis.md:304`, `src/cmds/README.md` | 🟢 |
| Manutenibilidade | 🟡 Filtros devem ficar no ecossistema da ferramenta; um novo ecossistema se justifica quando ha tres ou mais comandos relacionados. | `src/cmds/README.md` | 🟡 |
| Observabilidade | 🟢 Runners centralizam tracking de economia e mantem o mesmo texto emitido como referencia de saida filtrada. | `src/cmds/README.md`, `src/core/tracking.rs` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Compactar um comando com formato estruturado
  Dado uma ferramenta suportada que aceita JSON, XML, NDJSON, binlog ou TRX
  Quando o wrapper executa um subcomando elegivel
  Entao ele solicita o formato estruturado quando necessario
  E produz uma saida compacta baseada nos dados parseados
  E devolve o exit code real da ferramenta

Cenario: Processar log longo em streaming
  Dado uma ferramenta que produz output longo em blocos
  Quando o wrapper usa um filtro de streaming
  Entao linhas e blocos relevantes sao emitidos progressivamente
  E o filtro e finalizado apos o termino do processo
  E o codigo de saida do processo e preservado

Cenario: Preservar pedido de formato especial
  Dado uma invocacao com flag que exige saida crua ou JSON do fornecedor
  Quando o wrapper identifica que sua transformacao alteraria esse contrato
  Entao ele escolhe passthrough
  E os streams da ferramenta permanecem sob seu formato original

Cenario: Recuperar detalhe omitido por compactacao
  Dado uma saida filtrada que remove linhas ou blocos relevantes
  Quando o wrapper prepara a resposta
  Entao ele fornece hint de tee quando disponivel
  E quando a forma compacta nao economiza tokens, mostra a saida bruta

Cenario: Degradar parsing invalido
  Dado uma ferramenta que retorna estrutura invalida ou formato inesperado
  Quando o parser especializado falha
  Entao o wrapper nao inventa resumo
  E preserva output bruto, passthrough ou degradacao explicitamente marcada
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| 🟢 Execucao por wrapper com exit code preservado | Must | E o contrato basico de compatibilidade de toda ferramenta suportada. |
| 🟢 Selecao segura entre captura, streaming e passthrough | Must | Determina a fidelidade e o consumo de recursos de cada integracao. |
| 🟢 Guard de tokens e recuperacao de output | Must | Materializa a promessa de reduzir tokens sem piorar a saida original. |
| 🟢 Parsers estruturados e state machines por ecossistema | Should | Entregam a maior parte da compactacao especializada, com passthrough como alternativa. |
| 🟢 Tracking de economia | Should | Alimenta analitica, mas a ferramenta ainda deve funcionar quando o registro falha. |
| 🟡 Organizacao por ecossistema e roteamento cruzado | Could | Facilita expansao, mas nao muda o contrato externo de uma ferramenta isolada. |

> 🟡 As prioridades foram inferidas pelo compartilhamento de runners, pela cobertura de ferramentas e pelos invariantes de compatibilidade descritos no legado.

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/cmds/README.md` | escopo, modos de filtro, fluxo de execucao e contribuicao | 🟢 |
| `src/cmds/mod.rs` | registro dos ecossistemas | 🟢 |
| `src/cmds/git/git.rs` | wrapper e filtros Git | 🟢 |
| `src/cmds/rust/cargo_cmd.rs` | `CargoCommand`, handlers de build/teste | 🟢 |
| `src/cmds/cloud/aws_cmd.rs` | filtros AWS estruturados | 🟢 |
| `src/cmds/dotnet/dotnet_cmd.rs` | execucao .NET, binlog e TRX | 🟢 |
| `src/cmds/jvm/mvn_cmd.rs` | maquina de estados Maven | 🟢 |
| `src/cmds/system/search.rs` | filtro sensivel a flags para `grep`/`rg` | 🟢 |
| `src/core/runner.rs` | `run_filtered`, `run_streamed`, `run_passthrough` | 🟢 |
| `src/core/stream.rs` | `FilterMode`, `StreamFilter`, handlers de bloco | 🟢 |
| `_reversa_sdd/adrs/001-saida-filtrada-nunca-piora-a-saida-bruta.md` | invariante de saida | 🟢 |
