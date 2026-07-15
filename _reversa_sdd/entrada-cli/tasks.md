# Entrada CLI, Tarefas de Implementacao

> Sequencia para reimplementar a unit a partir do comportamento observado no legado. Cada tarefa preserva a rastreabilidade, o criterio de pronto e a confianca da evidencia.

## Pre-requisitos

- [ ] 🟢 Disponibilizar um parser de argumentos equivalente a Clap, capaz de distinguir ajuda/versao de erro comum de parse. Origem: `src/main.rs`.
- [ ] 🟢 Disponibilizar adaptadores para processo filho, streams, codigo de saida, sinais Unix e resolucao de executaveis. Origem: `src/main.rs`, `src/core/utils.rs`.
- [ ] 🟢 Implementar ou integrar os contratos chamados pela entrada: telemetria, integridade de hook, tracking, filtros TOML, tee e handlers especializados. Origem: `src/main.rs`, `src/core/`, `src/hooks/`.
- [ ] 🟢 Definir a lista de meta-comandos RTK para impedir fallback de uso interno invalido. Origem: `src/core/constants.rs`, `src/main.rs:1249`.
- [ ] 🔴 Validar o comportamento equivalente de sinais e spawn de processos em plataformas nao Unix antes de prometer paridade multiplataforma. Origem: blocos `#[cfg(unix)]` de `src/main.rs`.

## Tarefas

- [ ] T-01, Modelar a superficie CLI e o encerramento do processo.
  - Origem no legado: `src/main.rs:1496` e declaracoes de `Cli`/`Commands` no mesmo arquivo.
  - Criterio de pronto: invocacoes validas geram um comando tipado; ajuda e versao usam o comportamento do parser; erro interno de orquestracao escreve `rtk: <erro>` em stderr e encerra com `1`.
  - Confianca: 🟢

- [ ] T-02, Restaurar `SIGPIPE` no inicio do processo para Unix.
  - Origem no legado: `src/main.rs:1496`.
  - Criterio de pronto: em Unix, uma escrita em pipe fechado encerra silenciosamente conforme o handler padrao, sem panic/abort do runtime Rust.
  - Confianca: 🟢

- [ ] T-03, Orquestrar telemetria, parse, aviso de hook e verificacao de integridade antes do dispatch.
  - Origem no legado: `src/main.rs:1542-1568`.
  - Criterio de pronto: todo parse inicia `maybe_ping`; `gain` pula apenas o aviso de hook; comandos da whitelist exigem `runtime_check` bem-sucedido antes do handler.
  - Confianca: 🟢

- [ ] T-04, Implementar a whitelist de comandos operacionais.
  - Origem no legado: `src/main.rs:2668`.
  - Criterio de pronto: wrappers de shell suportados retornam verdadeiro e meta-comandos administrativos retornam falso; a lista e mantida explicitamente e coberta por teste de regressao.
  - Confianca: 🟢

- [ ] T-05, Implementar o dispatch central e os adaptadores de argumentos por ecossistema.
  - Origem no legado: `src/main.rs:1569-2640`, `build_k8s_namespace_args`, `build_k8s_logs_args`, `merge_pnpm_args`, `merge_pnpm_args_os` e `validate_pnpm_filters`.
  - Criterio de pronto: cada variante de `Commands` chama o handler certo; argumentos globais Git, filtros pnpm, namespace/logs Kubernetes/OpenShift e ferramentas `npx` conhecidas preservam a ordem e as restricoes observadas.
  - Confianca: 🟢

- [ ] T-06, Implementar fallback protegido para erros de parse.
  - Origem no legado: `src/main.rs:1249-1340`.
  - Criterio de pronto: sem argumentos ou com meta-comando RTK, o erro do parser encerra sem spawn; para outro token, o caminho cria `raw_command`, inicia tracking e tenta fallback externo.
  - Confianca: 🟢

- [ ] T-07, Implementar fallback filtrado por TOML com recuperacao e guard de saida.
  - Origem no legado: `src/main.rs:1264-1326`, `src/core/toml_filter.rs`, `src/core/tee.rs`, `src/core/runner.rs`.
  - Criterio de pronto: o lookup usa o basename do executavel; stdout e stderr obedecem `filter_stderr`; perdas recebem hint recuperavel e, sem hint, a saida bruta e emitida; tracking e parse failure sao gravados com o exit code do filho.
  - Confianca: 🟢

- [ ] T-08, Implementar fallback passthrough e erro de spawn.
  - Origem no legado: `src/main.rs:1328-1340`.
  - Criterio de pronto: sem filtro ou com `RTK_NO_TOML`, stdin/stdout/stderr sao herdados e o status do filho e preservado; falha de spawn escreve uma unica mensagem `[rtk: <erro>]`, registra parse failure e devolve `127`.
  - Confianca: 🟢

- [ ] T-09, Implementar `run` como execucao por shell e `proxy` como execucao transparente rastreada.
  - Origem no legado: `src/main.rs`, ramos `Commands::Run` e `Commands::Proxy`.
  - Criterio de pronto: `run` executa `sh -c` ou `cmd /C` somente quando ha comando; `proxy` valida argumentos, trata uma string quoted via lexer, preserva streams e devolve o status do filho.
  - Confianca: 🟢

- [ ] T-10, Implementar streaming, limite de captura e propagacao de sinais do `proxy`.
  - Origem no legado: `src/main.rs:2449-2605`.
  - Criterio de pronto: stdout e stderr sao espelhados por threads sem filtro; cada captura e limitada a 1 MiB para tracking; em Unix, `SIGINT`/`SIGTERM` encerra e aguarda o filho antes de reemitir o sinal.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar parse e dispatch de comandos representativos, incluindo flags e argumentos de Git preservados.
  - Origem no legado: testes em `src/main.rs:2732+`.
  - Criterio de pronto: `Cli::try_parse_from` valida comandos tipados, multiplas mensagens de commit e opcoes globais Git sem perder argumentos.
  - Confianca: 🟢

- [ ] TT-02, Testar que meta-comandos invalidos nao chamam fallback externo e que ferramenta externa desconhecida usa fallback.
  - Origem no legado: `src/main.rs:1249-1263` e testes de fallback no mesmo arquivo.
  - Criterio de pronto: um spy de spawn confirma zero execucoes para meta-comando invalido e uma execucao para token externo elegivel.
  - Confianca: 🟢

- [ ] TT-03, Testar fallback TOML, `RTK_NO_TOML`, status do filho e erro `127`.
  - Origem no legado: `src/main.rs:1264-1340`.
  - Criterio de pronto: fixture filtrada preserva exit code; ambiente com TOML desabilitado herda streams; falha de spawn nao imprime erro Clap duplicado e devolve `127`.
  - Confianca: 🟢

- [ ] TT-04, Testar o guard de perda no fallback filtrado.
  - Origem no legado: `src/main.rs:1290-1312`.
  - Criterio de pronto: perda com hint emite saida filtrada+hints; perda sem hint emite integralmente a saida bruta.
  - Confianca: 🟢

- [ ] TT-05, Testar helpers de Kubernetes e pnpm, incluindo aviso de `typecheck` com filtros globais.
  - Origem no legado: `src/main.rs:1435-1492`, `src/main.rs:3420-3485`.
  - Criterio de pronto: `all` gera `-A`, namespace gera `-n <valor>`, container gera `-c <valor>` e os filtros de `pnpm typecheck` recebem o aviso esperado.
  - Confianca: 🟢

- [ ] TT-06, Testar `proxy` com argumentos separados, string unica quoted, streams concorrentes e limite de captura.
  - Origem no legado: `src/main.rs:2449-2605`.
  - Criterio de pronto: o processo filho recebe tokens corretos; stdout/stderr visiveis nao sao truncados; a entrada armazenada em tracking nao excede 1 MiB por stream.
  - Confianca: 🟢

- [ ] TT-07, Executar teste de pipe fechado em Unix.
  - Origem no legado: teste ignorado `src/main.rs:3475+`.
  - Criterio de pronto: apos build, `rtk` em pipeline com consumidor que fecha cedo nao aborta nem deixa core dump.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 A unit apenas delega eventos ao tracking; nao e dona do schema SQLite ou de migracoes.

## Ordem Sugerida

1. Implementar T-01 a T-04 para estabelecer parse, ciclo de vida e o gate de integridade antes de expor handlers.
2. Implementar T-05 para ligar os handlers especializados, mantendo os testes de compatibilidade de argumentos junto aos helpers.
3. Implementar T-06 a T-08 para proteger a borda entre erro de uso RTK e ferramenta externa, antes de adicionar otimizacoes de filtros.
4. Implementar T-09 e T-10 depois que a camada de processo e tracking estiver disponivel.
5. Executar TT-01 a TT-07 na mesma ordem funcional; TT-07 requer ambiente Unix e binario construido.

## Lacunas Pendentes (🔴)

- 🔴 Confirmar em Windows a paridade de quoting, spawn, codigos de saida e propagacao de sinais, pois a extracao validou apenas os blocos Unix do legado.
- 🔴 Exercitar em runtime a matriz completa de dispatch contra as versoes reais das ferramentas externas; a analise atual nao prova compatibilidade dinamica.
- 🔴 Decidir se a whitelist que falha aberta para novos comandos operacionais deve ser preservada ou substituida por uma politica mais restritiva na reimplementacao.
