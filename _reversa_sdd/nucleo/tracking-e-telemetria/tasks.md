# Tracking e Telemetria, Tarefas de Implementacao

## Pre-requisitos

- [ ] 🟢 Disponibilizar SQLite, relogio UTC e diretorio de dados por plataforma. Origem: `src/core/tracking.rs:32-64`.
- [ ] 🟢 Definir configuracao de telemetria, armazenamento de consentimento e endpoint opcional compilado. Origem: `src/core/telemetry.rs:16-48`, `src/core/telemetry_cmd.rs:92`.

## Tarefas

- [ ] T-01, Criar o `Tracker` e schema SQLite de `commands` e `parse_failures`, com indices e migracoes idempotentes.
  - Origem no legado: `src/core/tracking.rs:249-326`.
  - Criterio de pronto: banco novo e banco antigo abrem com colunas `exec_time_ms` e `project_path`; NULLs historicos sao normalizados.
  - Confianca: 🟢

- [ ] T-02, Implementar gravacao de execucao, calculo saturado de economia, escopo de projeto e retencao de 90 dias.
  - Origem no legado: `src/core/tracking.rs:402-449`, `src/core/constants.rs:6`.
  - Criterio de pronto: cada registro armazena timestamp, comandos, tokens, percentual, duracao e cwd; entradas antigas nas duas tabelas sao removidas.
  - Confianca: 🟢

- [ ] T-03, Implementar `TimedExecution` para tracking best-effort normal e passthrough de tokens zero.
  - Origem no legado: `src/core/tracking.rs:1306-1398`.
  - Criterio de pronto: uma falha de banco nao altera o fluxo de comando; passthrough conserva tempo sem diluir metricas de economia.
  - Confianca: 🟢

- [ ] T-04, Implementar gates de telemetria: endpoint compilado, opt-out por ambiente/config, consentimento e intervalo de 23 horas.
  - Origem no legado: `src/core/telemetry.rs:16-68`.
  - Criterio de pronto: qualquer gate falho impede rede; marker e atualizado antes de uma unica thread de envio.
  - Confianca: 🟢

- [ ] T-05, Montar payload pseudonimo a partir de hash de dispositivo e estatisticas locais disponiveis, com timeout de rede.
  - Origem no legado: `src/core/telemetry.rs:71-150`.
  - Criterio de pronto: indisponibilidade do tracker usa valores neutros e falha de rede nao bloqueia o CLI.
  - Confianca: 🟢

- [ ] T-06, Implementar subcomandos `status`, `enable`, `disable` e `forget`.
  - Origem no legado: `src/core/telemetry_cmd.rs:12-182`.
  - Criterio de pronto: enable exige TTY e resposta explicita; forget remove consentimento, salt, marker e banco local antes de tentar erasure remoto.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar schema, migracoes repetidas, retencao e filtro por projeto contendo `_` e `%`.
  - Origem no legado: `src/core/tracking.rs:51-61`, `src/core/tracking.rs:249-449`.
  - Criterio de pronto: consultas nao tratam caracteres de caminho como curingas de `LIKE` e migracoes repetidas sao seguras.
  - Confianca: 🟢

- [ ] TT-02, Testar `TimedExecution` normal e passthrough em banco isolado.
  - Origem no legado: `src/core/tracking.rs:1447-1537`.
  - Criterio de pronto: registros normal e passthrough existem, e o segundo tem tokens zero.
  - Confianca: 🟢

- [ ] TT-03, Testar gates de telemetria sem endpoint, sem consentimento, com opt-out e marker recente.
  - Origem no legado: `src/core/telemetry.rs:22-68`.
  - Criterio de pronto: nenhum desses cenarios agenda requisicao de rede.
  - Confianca: 🟢

- [ ] TT-04, Testar que `enable` recusa entrada por pipe e que `forget` remove estado local mesmo se erasure remoto falhar.
  - Origem no legado: `src/core/telemetry_cmd.rs:63-100`, `src/core/telemetry_cmd.rs:109-157`.
  - Criterio de pronto: consentimento nao e aceito sem TTY e o apagamento local permanece concluido em falha remota.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] TM-01, Definir compatibilidade e importacao do banco de tracking legado.
  - Origem no legado: `src/core/tracking.rs:281-308`.
  - Criterio de pronto: ha decisao registrada sobre preservar comandos e falhas de parse existentes, incluindo versoes de schema anteriores.
  - Confianca: 🔴

## Ordem Sugerida

1. Implementar T-01 e T-02 antes de integrar qualquer consumidor de metricas.
2. Implementar T-03 para tornar tracking tolerante ao erro no caminho de execucao.
3. Implementar T-04 e T-05 antes dos comandos de controle, pois estes dependem do modelo de consentimento.
4. Implementar T-06 e executar TT-01 a TT-04 antes de habilitar coleta remota.

## Lacunas Pendentes (🔴)

- 🔴 Validar concorrencia de SQLite em multiplas instancias e sistemas de arquivos sem suporte a WAL.
- 🔴 Validar contrato do endpoint e processamento efetivo de erasure remoto.
