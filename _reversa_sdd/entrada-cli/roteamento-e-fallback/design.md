# Roteamento e Fallback, Design Tecnico

> Implementacao observada principalmente em `run_fallback()` de `src/main.rs`. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Interface

🟢 O caso de uso e uma funcao interna acionada por `run_cli()` quando `Cli::try_parse()` falha fora dos casos de ajuda e versao. Ela recebe o `clap::Error`, relê os argumentos do processo e produz `Result<i32>`; os efeitos colaterais sao execucao de processo filho, emissao de stdout/stderr e registros de tracking.

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `run_fallback` | `(parse_error: clap::Error) -> Result<i32>` | exit code ou erro | 🟢 Decide erro de CLI, filtro TOML ou passthrough. |
| `RTK_META_COMMANDS` | `&[&str]` | lista estatica | 🟢 Define tokens que nunca podem cair em execucao crua. |
| `toml_disabled` | `() -> bool` | booleano | 🟢 Retorna verdadeiro apenas para `RTK_NO_TOML=1`. |
| `find_matching_filter` | `(&str) -> Option<&CompiledFilter>` | filtro opcional | 🟢 Seleciona filtro para o comando normalizado. |
| `apply_filter_with_info` | `(&CompiledFilter, &str) -> (String, Lossiness)` | texto e tipo de perda | 🟢 Expõe se a transformacao preservou, removeu cauda ou removeu trechos nao recuperaveis. |
| `emit_guarded` | `(&str, Option<&str>, &str) -> String` | texto efetivamente emitido | 🟢 Combina hint, executa `never_worse`, imprime e devolve exatamente o texto rastreado. |

## Fluxo Principal

1. 🟢 `run_cli()` encaminha apenas erro comum de `Cli::try_parse()` para `run_fallback()`; ajuda e versao terminam pelo Clap.
2. 🟢 `run_fallback()` coleta `std::env::args().skip(1)`; sem argumentos, chama `parse_error.exit()`.
3. 🟢 O primeiro token e comparado a `RTK_META_COMMANDS`; quando existe correspondencia, `parse_error.exit()` bloqueia a resolucao pelo `PATH`.
4. 🟢 Para token externo, a unit monta `raw_command`, remove ANSI da mensagem de parse e cria `TimedExecution` antes de iniciar o processo.
5. 🟢 A unit transforma o primeiro token em basename e o recombina com os demais argumentos como `lookup_cmd`.
6. 🟢 `toml_disabled()` interrompe a busca quando `RTK_NO_TOML=1`; caso contrario `find_matching_filter()` decide entre o caminho filtrado e o passthrough.
7. 🟢 O caminho escolhido retorna um codigo inteiro: o status real do filho, ou `127` para falha de spawn.

## Caminho Filtrado

1. 🟢 A unit inicia `resolved_command(args[0])`, herda stdin e canaliza stdout; stderr tambem e canalizado apenas quando `filter.filter_stderr` esta ativo.
2. 🟢 Depois de `output()`, `exit_code_from_output()` conserva o status do filho e os bytes de stdout/stderr passam por `String::from_utf8_lossy`.
3. 🟢 Com `filter_stderr`, stdout e stderr sao concatenados antes de filtrar; sem a opcao, stderr ja foi exposto diretamente e apenas stdout e filtrado.
4. 🟢 `apply_filter_with_info()` executa a pipeline declarativa: remove ANSI, aplica substituicoes, avalia regras de blob, filtra linhas, trunca caracteres Unicode-safe, aplica head/tail/max e trata saida vazia.
5. 🟢 A pipeline retorna `Lossiness::None`, `Lossiness::Tail { tee_payload, tail_offset }` ou `Lossiness::Whole`; a unit marca como lossy qualquer variante diferente de `None`.
6. 🟢 Em falha do filho, `tee_and_hint()` tenta preservar recuperacao da saida completa; em sucesso, `Tail` usa `force_tee_tail_hint()` e `Whole` usa `force_tee_hint()`.
7. 🟢 Se a filtragem for lossy e nenhum hint existir, `emit_guarded(raw, None, raw)` emite a referencia bruta; nos demais casos, `emit_guarded(filtered, hint, raw)` aplica `never_worse` antes de imprimir.
8. 🟢 `timer.track()` recebe a saida bruta e exatamente o texto emitido; `record_parse_failure_silent(..., true)` registra que o fallback conseguiu executar a ferramenta.
9. 🟢 Falha de spawn registra `succeeded=false`, escreve `[rtk: <erro>]` e devolve `127`.

## Caminho Passthrough

1. 🟢 Sem filtro, a unit inicia o executavel resolvido com stdin, stdout e stderr em `Stdio::inherit()`.
2. 🟢 Em sucesso de spawn, `track_passthrough()` registra apenas duracao e tokens zero para nao diluir as estatisticas de economia.
3. 🟢 A unit registra parse failure com sucesso e retorna `exit_code_from_status()` do filho.
4. 🟢 Em falha de spawn, o comportamento coincide com o caminho filtrado: tracking de falha, uma mensagem local e codigo `127`.

## Fluxos Alternativos

- 🟢 **Nenhum argumento:** delega o encerramento ao erro de parse; nao cria timer nem processo filho.
- 🟢 **Token RTK interno:** delega o encerramento ao erro de parse; essa e a protecao contra binario homonimo externo.
- 🟢 **Caminho absoluto:** a busca usa somente o basename como primeiro termo, mas a execucao mantem o caminho original do executavel.
- 🟢 **Stderr filtravel:** permite remover banners de ferramentas que escrevem ruido em stderr, ao custo de postergar a exibicao ate `output()` concluir.
- 🟢 **Filtro sem perda:** nao cria hint; o guard ainda compara o texto filtrado com a referencia bruta.
- 🟢 **Filtragem sem recuperacao:** a referencia bruta e mostrada para evitar marcador de truncamento irrecuperavel.

## Dependencias

- 🟢 `clap::Error`: erro de parse que identifica a entrada do caso de uso e controla o encerramento de meta-comandos.
- 🟢 `core::constants::RTK_META_COMMANDS`: classificador de fronteira entre CLI RTK e ferramenta externa.
- 🟢 `core::utils`: remove ANSI, resolve executavel e extrai codigos de `Output`/`ExitStatus`.
- 🟢 `core::toml_filter`: verifica `RTK_NO_TOML`, localiza o filtro e executa sua pipeline.
- 🟢 `core::tee`: persiste material de recuperacao e constroi hints para perda de saida.
- 🟢 `core::runner` e `core::guard`: emitem e limitam o texto visivel pelo invariante `never_worse`.
- 🟢 `core::tracking`: mede a execucao e registra parse failures de modo tolerante a falhas.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| A classificacao por meta-comando ocorre antes de qualquer tentativa de resolver executavel externo. | `src/main.rs:1253-1261`, `src/core/constants.rs:10` | 🟢 |
| O filtro casa contra basename, mas o spawn usa o argumento original. | `src/main.rs:1269-1285` | 🟢 |
| Stderr e capturado somente sob opt-in do filtro para preservar visibilidade direta por padrao. | `src/main.rs:1289-1306` | 🟢 |
| O texto emitido pelo guard e o mesmo enviado ao tracking, preservando a coerencia das metricas. | `src/core/runner.rs:12-21`, `src/main.rs:1342-1353` | 🟢 |
| Passthrough registra zero tokens porque a saida nao e capturada e nao deve reduzir artificialmente a taxa de economia. | `src/core/tracking.rs:1392-1402` | 🟢 |
| A transformacao TOML pode alterar a ordem e o conteudo textual por uma pipeline declarativa, por isso a paridade depende das regras distribuídas no build. | `src/core/toml_filter.rs:515-650` | 🟡 |

## Estado Interno

- 🟢 `args`, `raw_command`, `error_message` e `lookup_cmd` existem apenas durante uma invocacao de fallback.
- 🟢 `TimedExecution` conserva o instante inicial e grava duracao quando `track()` ou `track_passthrough()` e chamado.
- 🟢 O registro de filtros e lazy/static e carregado uma vez por processo; o caso de uso apenas consulta sua API.
- 🟢 O tracking e best-effort: `record_parse_failure_silent()` ignora erro ao abrir ou gravar no banco.

## Observabilidade

- 🟢 Caminho filtrado registra comando original, rotulo `rtk:toml <comando>`, texto bruto e texto efetivamente exibido.
- 🟢 Caminho passthrough registra comando original, rotulo `rtk fallback: <comando>`, duracao e zero tokens.
- 🟢 As duas rotas registram a mensagem de parse sanitizada e o resultado de sucesso/falha via `record_parse_failure_silent()`.
- 🟢 Falha de spawn escreve uma unica mensagem em stderr com o prefixo `[rtk: ...]`.

## Riscos e Lacunas

- 🔴 A equivalencia runtime dos filtros TOML com as versoes reais das ferramentas externas nao foi executada nesta extracao.
- 🔴 O comportamento de `parse_error.exit()` e do codigo de encerramento do Clap deve ser validado na stack de reimplementacao escolhida.
- 🟡 Filtrar stderr deliberadamente altera quando o usuario ve erros da ferramenta, embora preserve o texto combinado para o filtro.
- 🟡 `record_parse_failure_silent()` pode ocultar indisponibilidade do tracking; isso privilegia a execucao do comando sobre a telemetria local.
