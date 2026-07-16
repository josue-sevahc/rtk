# Lacunas da Revisao - RTK

> Estado em 2026-07-16: revisao cruzada concluida; validacao humana pendente.

## Lacunas que Exigem Decisao

| Tema | Units afetadas | Pergunta |
|---|---|---|
| Baseline de compatibilidade | CLI, wrappers, filtros, hooks, scripts | `questions.md#pergunta-1` |
| Gate de integridade | entrada CLI | `questions.md#pergunta-2` |
| Limite de captura | execucao filtrada | `questions.md#pergunta-3` |
| Persistencia local | nucleo e tracking | `questions.md#pergunta-4` |
| Backend de telemetria | tracking e telemetria | `questions.md#pergunta-5` |
| Falhas do OpenClaw | plugin OpenClaw | `questions.md#pergunta-6` |
| Arquivo de recomendacoes | aprendizado | `questions.md#pergunta-7` |
| Localizacao de saida | parser | `questions.md#pergunta-8` |
| Calibracao de heuristicas | descoberta, aprendizado, filtros e parser | `questions.md#pergunta-9` |

## Lacunas de Validacao Externa

- A analise nao executou a matriz completa de ferramentas externas, agentes, sistemas operacionais e locales.
- O contrato runtime do OpenClaw e dos demais hosts nao esta versionado neste repositorio.
- O backend de telemetria e erasure nao faz parte da superficie analisada.
- Benchmarks de producao para economia, falsos positivos e grandes historicos nao estao disponiveis nos artefatos locais.

## Inconsistencias Confirmadas

- O caminho canonico implementado e `history.db`; documentos mais antigos ainda citam `tracking.db`.
- Scripts de instalacao e validacao mantem referencias a fork, branch e caminhos de hook legados.
- O README do parser descreve tipos planejados que ainda nao existem em `src/parser/types.rs`.
- `rtk learn --write-rules` sobrescreve integralmente o arquivo de correcoes; nao ha merge.

## Resolvidas pelo Revisor

- O destino de `docs/guide/` foi confirmado como `rtk-ai/rtk-website`, via `prepare-docs.mjs` e Starlight.
- A truncagem de captura em 10 MiB e seu warning foram confirmados diretamente em `src/core/stream.rs`.
- A ausencia de testes no pacote OpenClaw e o uso de shapes manuais foram reclassificados como fatos observaveis; somente a compatibilidade runtime permanece aberta.

