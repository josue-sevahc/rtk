# Roteamento e Fallback, Tarefas de Implementacao

> Sequencia executavel para reconstruir o fallback seguro e compatível com o legado. Cada tarefa referencia sua origem, criterio de pronto e confianca.

## Pre-requisitos

- [ ] 🟢 Disponibilizar um parser de CLI que exponha erro de ajuda/versao separadamente de erro de sintaxe comum. Origem: `src/main.rs:1546-1553`.
- [ ] 🟢 Disponibilizar resolucao de executavel, `ExitStatus`/`Output`, streams herdados/canalizados e um mecanismo de execucao de processo filho. Origem: `src/main.rs:1289-1384`.
- [ ] 🟢 Disponibilizar o registro de filtros TOML, o guard `never_worse`, o tee de recuperacao e o tracking best-effort. Origem: `src/core/toml_filter.rs`, `src/core/runner.rs`, `src/core/tee.rs`, `src/core/tracking.rs`.
- [ ] 🟢 Definir `RTK_META_COMMANDS` e uma classificacao obrigatoria para toda superficie de subcomandos. Origem: `src/core/constants.rs:10`, `src/main.rs:3043-3123`.

## Tarefas

- [ ] T-01, Implementar a classificacao de entrada entre erro de CLI interno e ferramenta externa.
  - Origem no legado: `src/main.rs:1249-1261`, `src/core/constants.rs:10`.
  - Criterio de pronto: sem argumentos ou com primeiro token presente em `RTK_META_COMMANDS`, o fluxo encerra pelo erro do parser e nao resolve nem inicia executavel externo.
  - Confianca: 🟢

- [ ] T-02, Criar guardrail de classificacao para novos subcomandos.
  - Origem no legado: teste `test_every_subcommand_is_classified` em `src/main.rs:3043-3123`.
  - Criterio de pronto: a suite falha quando um subcomando nao esta explicitamente classificado como meta-comando ou passthrough; a mensagem aponta a classificacao obrigatoria.
  - Confianca: 🟢

- [ ] T-03, Preparar comando externo e lookup de filtro pelo basename.
  - Origem no legado: `src/main.rs:1263-1285`.
  - Criterio de pronto: o fluxo preserva `raw_command`, sanitiza a mensagem de parse, inicia medicao de tempo e transforma `/caminho/ferramenta arg` em `ferramenta arg` apenas para lookup; o spawn conserva o caminho original.
  - Confianca: 🟢

- [ ] T-04, Implementar selecao entre filtro TOML e passthrough.
  - Origem no legado: `src/main.rs:1281-1285`, `src/core/toml_filter.rs:402`.
  - Criterio de pronto: `RTK_NO_TOML=1` impede `find_matching_filter`; sem filtro, o fluxo escolhe passthrough; com filtro, escolhe captura filtrada.
  - Confianca: 🟢

- [ ] T-05, Implementar execucao filtrada e composicao de streams.
  - Origem no legado: `src/main.rs:1287-1325`.
  - Criterio de pronto: stdin e herdado; stdout e capturado; stderr e herdado por padrao e somente canalizado/concatenado quando `filter_stderr` for verdadeiro; o exit code do filho e retido.
  - Confianca: 🟢

- [ ] T-06, Implementar pipeline de perda, tee e guard de saida.
  - Origem no legado: `src/main.rs:1320-1347`, `src/core/toml_filter.rs:515-650`, `src/core/runner.rs:12-21`.
  - Criterio de pronto: `Lossiness::Tail` e `Whole` produzem hint quando possivel; perda sem hint emite raw; todos os outros casos passam pelo guard e nunca exibem mais tokens estimados que a referencia bruta.
  - Confianca: 🟢

- [ ] T-07, Registrar resultado do caminho filtrado sem alterar a resposta ao usuario.
  - Origem no legado: `src/main.rs:1349-1362`, `src/core/tracking.rs:1259`.
  - Criterio de pronto: o tracking recebe entrada bruta e exatamente a saida exibida; parse failure recebe sucesso apos spawn; erros do tracking nao interrompem a ferramenta.
  - Confianca: 🟢

- [ ] T-08, Implementar passthrough com streams herdados e metricas neutras.
  - Origem no legado: `src/main.rs:1365-1384`, `src/core/tracking.rs:1392-1402`.
  - Criterio de pronto: sem filtro, o filho recebe stdin/stdout/stderr herdados, o status e devolvido e o registro de tracking usa tokens de entrada/saida iguais a zero.
  - Confianca: 🟢

- [ ] T-09, Padronizar tratamento de falha de spawn nos dois caminhos.
  - Origem no legado: `src/main.rs:1358-1362`, `src/main.rs:1381-1385`.
  - Criterio de pronto: executavel ausente gera uma unica mensagem `[rtk: <erro>]`, registra parse failure com `succeeded=false` e devolve `127`, sem repetir o erro de parse.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar que cada meta-comando rejeita flag desconhecida sem cair em execucao crua.
  - Origem no legado: teste `test_meta_commands_reject_bad_flags` em `src/main.rs:3026-3041`.
  - Criterio de pronto: para cada meta-comando que nao aceita argumentos livres, `--nonexistent-flag-xyz` causa erro de parse e um spy confirma ausencia de spawn externo.
  - Confianca: 🟢

- [ ] TT-02, Testar o guardrail de cobertura da classificacao de subcomandos.
  - Origem no legado: teste `test_every_subcommand_is_classified` em `src/main.rs:3043-3123`.
  - Criterio de pronto: adicionar temporariamente um subcomando sem categoria faz o teste falhar; classificá-lo restaura o teste.
  - Confianca: 🟢

- [ ] TT-03, Testar lookup pelo basename e bypass de TOML.
  - Origem no legado: `src/main.rs:1269-1285`, `src/core/toml_filter.rs:402`.
  - Criterio de pronto: um fixture com caminho absoluto encontra o filtro pelo basename; com `RTK_NO_TOML=1`, a funcao de busca nao e chamada e o comando segue em passthrough.
  - Confianca: 🟢

- [ ] TT-04, Testar composicao de stdout/stderr no caminho filtrado.
  - Origem no legado: `src/main.rs:1289-1318`.
  - Criterio de pronto: com `filter_stderr=false`, stderr permanece visivel sem passar pelo filtro; com `true`, a entrada do filtro contem stdout seguido de stderr.
  - Confianca: 🟢

- [ ] TT-05, Testar perda, hint e `never_worse`.
  - Origem no legado: `src/main.rs:1320-1347`, `src/core/runner.rs:12-21`.
  - Criterio de pronto: `Tail` e `Whole` produzem hint quando o tee esta disponivel; perda sem hint emite raw; uma saida filtrada+hints maior que raw e substituida por raw.
  - Confianca: 🟢

- [ ] TT-06, Testar preservacao de exit code e tracking nos dois caminhos.
  - Origem no legado: `src/main.rs:1299-1355`, `src/main.rs:1370-1380`.
  - Criterio de pronto: filhos com sucesso e falha devolvem seu codigo original; caminho filtrado registra texto efetivo; passthrough registra tokens zero.
  - Confianca: 🟢

- [ ] TT-07, Testar executavel ausente.
  - Origem no legado: `src/main.rs:1358-1362`, `src/main.rs:1381-1385`.
  - Criterio de pronto: os dois caminhos retornam `127`, registram `succeeded=false` e produzem apenas a mensagem local esperada.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 O caso de uso apenas emite registros de tracking e parse failure; nao e proprietario de tabelas ou migracoes.

## Ordem Sugerida

1. Implementar T-01 e T-02 primeiro, pois a classificacao e a barreira que impede comportamento externo inesperado.
2. Implementar T-03 e T-04 para estabelecer a decisao de rota sem ainda transformar output.
3. Implementar T-05 a T-07 em conjunto: captura, fidelidade e tracking dependem da mesma saida efetivamente emitida.
4. Implementar T-08 e T-09 para completar o caminho compatível de passthrough e falha de spawn.
5. Executar TT-01 a TT-07; os testes de guardrail e bloqueio de meta-comando devem rodar em toda mudanca da superficie CLI.

## Lacunas Pendentes (🔴)

- 🔴 Validar filtros TOML contra as versoes reais de cada ferramenta externa e seus formatos de stdout/stderr.
- 🔴 Definir o comportamento de encerramento equivalente ao `parse_error.exit()` se a reimplementacao nao usar Clap.
- 🔴 Decidir se o modelo best-effort de tracking deve continuar silenciando indisponibilidade de banco ou se a reimplementacao deve expor diagnostico opcional.
