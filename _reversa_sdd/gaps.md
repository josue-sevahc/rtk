# Lacunas da Revisao - RTK

> Estado em 2026-07-16: revisao cruzada e validacao humana concluidas.

## Decisoes Humanas Pendentes

Nenhuma. As nove perguntas de `questions.md` foram respondidas e incorporadas nas specs.

## Lacunas de Validacao Externa

- O baseline certificado inicial e Linux x86_64, Bash/Zsh, UTF-8, Claude Code e OpenClaw; combinacoes sem fixture ou teste permanecem experimentais.
- Ferramentas externas, formatos de hosts e a API runtime do OpenClaw ainda exigem matrizes executaveis de compatibilidade.
- Concorrencia SQLite/WAL, streams sob carga, cancelamento e historicos muito grandes ainda exigem testes dinamicos.
- O backend de telemetria e erasure permanece fora do escopo; a paridade declarada cobre somente o cliente.
- Metricas de producao para falsos positivos e economia nao estao disponiveis; os valores atuais ficam preservados no perfil `legacy-v1`.
- A equivalencia de `parse_error.exit()` e outros detalhes dependentes da stack alvo deve ser validada quando a tecnologia de reconstrucao for escolhida.

## Inconsistencias Confirmadas

- O caminho canonico implementado e `history.db`; documentos mais antigos ainda citam `tracking.db`.
- Scripts de instalacao e validacao mantem referencias a fork, branch e caminhos de hook legados.
- O README do parser descreve tipos planejados que ainda nao existem em `src/parser/types.rs`.
- `rtk learn --write-rules` sobrescreve integralmente o arquivo de correcoes; nao ha merge.

## Resolvidas pelo Revisor

- O destino de `docs/guide/` foi confirmado como `rtk-ai/rtk-website`, via `prepare-docs.mjs` e Starlight.
- A truncagem de captura em 10 MiB e seu warning foram confirmados diretamente em `src/core/stream.rs`.
- A ausencia de testes no pacote OpenClaw e o uso de shapes manuais foram reclassificados como fatos observaveis; somente a compatibilidade runtime permanece aberta.
- Segurança de novos wrappers, migracao SQLite, politica de truncamento, falhas do OpenClaw, merge de rules, localizacao e versionamento de heuristicas receberam decisoes humanas explicitas.
