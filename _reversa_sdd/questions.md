# Perguntas para Validacao - RTK

> Gerado pelo Revisor em 2026-07-16.
> As nove perguntas foram respondidas e incorporadas nas specs em 2026-07-16.

## Pergunta 1

**Contexto:** As specs de CLI, wrappers, filtros, hooks e scripts registram compatibilidade externa ainda nao exercitada.
**Specs afetadas:** `entrada-cli/`, `wrappers-comandos/`, `perfis-de-filtros/`, `integracoes-de-agentes/`, `automacao-e-scripts/`
**Pergunta:** Qual baseline deve ser tratado como oficialmente suportado na reconstrucao: sistemas operacionais, shells, agentes, versoes de ferramentas e locales?
**Impacto:** Define quais lacunas viram criterios obrigatorios de paridade e qual matriz de testes precisa existir.

**Status:** ✅ Respondida

**Resposta:** Adotar baseline certificado inicial restrito: Linux x86_64, Bash/Zsh,
UTF-8, Claude Code e OpenClaw. Outros sistemas, shells, agentes, ferramentas,
versões e locales devem permanecer experimentais até validação em matriz
executável. Compatibilidade de wrappers só deve ser declarada quando houver
fixture ou teste correspondente.

## Pergunta 2

**Contexto:** `is_operational_command()` usa whitelist; um wrapper novo esquecido nao passa pela verificacao de integridade.
**Spec afetada:** `entrada-cli/tasks.md`
**Pergunta:** A reconstrucao deve preservar essa falha aberta ou adotar uma politica fechada por padrao para novos comandos operacionais?
**Impacto:** Altera o modelo de seguranca e os testes do dispatch central.

**Status:** ✅ Respondida

**Resposta:** Adotar política fechada por padrão para comandos operacionais e wrappers
pertencentes ao RTK. Comandos externos não reconhecidos continuam em
passthrough. Substituir a whitelist dispersa por um registro central de
comandos com ownership e política de integridade explícitos.

## Pergunta 3

**Contexto:** A captura para filtro e tracking para de acumular apos 10 MiB por stream e emite warning.
**Spec afetada:** `wrappers-comandos/execucao-filtrada/tasks.md`
**Pergunta:** O teto fixo de 10 MiB deve ser preservado, tornado configuravel ou substituido por outra estrategia?
**Impacto:** Afeta uso de memoria, fidelidade de filtros, tee, tracking e comunicacao de truncamento.

**Status:** ✅ Respondida

**Resposta:** Preservar 10 MiB por stream como valor default de compatibilidade, mas
torná-lo configurável dentro de limites seguros (com relativa facilidade e também por interface se possível).
Não permitir modo ilimitado no baseline inicial. Todo truncamento deve emitir warning e ser registrado
no tracking.


## Pergunta 4

**Contexto:** O codigo atual usa `history.db`, enquanto documentos legados ainda citam `tracking.db`; a unit de nucleo pede uma decisao de migracao.
**Spec afetada:** `nucleo/tasks.md`
**Pergunta:** Em uma reconstrucao, os dados locais existentes devem ser migrados, preservados in-place, importados sob demanda ou descartados?
**Impacto:** Define compatibilidade de schema, caminho do banco e tarefas de migracao.

**Status:** ✅ Respondida

**Resposta:** Usar history.db como banco canônico. Detectar tracking.db automaticamente,
mas importar somente mediante confirmação ou comando explícito. A importação
deve ser idempotente, criar backup, deduplicar registros e nunca modificar ou
apagar o banco legado.

## Pergunta 5

**Contexto:** URL e token de telemetria sao injetados no build; o cliente tenta `POST /erasure`, mas o backend nao esta neste repositorio.
**Spec afetada:** `nucleo/tracking-e-telemetria/tasks.md`
**Pergunta:** Existe um contrato operacional do backend que deve ser considerado fonte de verdade para ping, retencao e erasure remoto?
**Impacto:** Sem essa fonte, a paridade pode cobrir apenas o comportamento cliente e o fallback por email.

**Status:** ✅ Respondida

**Resposta:** Não existe contrato operacional de backend validado no escopo analisado.
A paridade deve cobrir apenas o cliente. Telemetria permanece desabilitada por
padrão, e erasure remoto deve ser considerado indisponível enquanto URL,
payloads, autenticação, retenção e SLA não forem fornecidos como contrato
externo canônico.

## Pergunta 6

**Contexto:** No plugin OpenClaw, timeout, exit desconhecido e falha operacional viram passthrough silencioso, assim como ausencia de rewrite.
**Spec afetada:** `plugin-openclaw/tasks.md`
**Pergunta:** Essa degradacao silenciosa deve permanecer como politica de producao ou erros operacionais devem gerar aviso/bloqueio distinto?
**Impacto:** Muda disponibilidade, observabilidade e possivelmente a fronteira de seguranca do plugin.

**Status:** ✅ Respondida

**Resposta:** Preservar passthrough silencioso apenas quando não houver rewrite aplicável.
Timeout, resposta inválida, exit code desconhecido ou falha de processo devem
gerar warning estruturado e defer/passthrough conforme política. Falhas de
integridade ou segurança devem bloquear ou solicitar aprovação explícita.


## Pergunta 7

**Contexto:** `rtk learn --write-rules` recompõe `.claude/rules/cli-corrections.md` com `fs::write` e substitui edicoes manuais.
**Spec afetada:** `aprendizado/recomendacoes-de-adocao/tasks.md`
**Pergunta:** A reconstrucao deve preservar a sobrescrita total, mesclar secoes geradas ou recusar a operacao quando detectar edicoes manuais?
**Impacto:** Define propriedade do arquivo, idempotencia e estrategia de conflito.

**Status:** ✅ Respondida

**Resposta:** Não preservar sobrescrita integral. O arquivo deve possuir seções gerenciadas
por marcadores, e somente essas seções podem ser atualizadas. Conteúdo manual
fora dos marcadores deve ser preservado. Na ausência de marcadores, a operação
deve recusar escrita, gerar arquivo separado ou exigir --force com backup.

## Pergunta 8

**Contexto:** O formatter gera textos em ingles, mas nao existe uma politica de localizacao no modulo.
**Spec afetada:** `parser-de-saida/formatacao-estruturada/design.md`
**Pergunta:** A saida estruturada deve permanecer somente em ingles ou a reconstrucao precisa prever localizacao?
**Impacto:** Afeta strings canonicas, snapshots, compatibilidade de parsers consumidores e testes.

**Status:** ✅ Respondida

**Resposta:** Manter códigos, chaves, enums e schemas estruturados em inglês como contrato
canônico. Permitir localização apenas da apresentação humana. en-US é obrigatório
e pt-BR deve ser o primeiro locale adicional.

## Pergunta 9

**Contexto:** Percentuais de economia, regexes e limiares de descoberta/aprendizado sao heuristicas estaticas sem calibracao de producao disponivel no repositorio.
**Specs afetadas:** `descoberta/`, `aprendizado/`, `perfis-de-filtros/`, `parser-de-saida/`
**Pergunta:** Esses valores devem ser tratados como contrato legado a preservar ou existem dados de producao/benchmarks que devem orientar uma recalibracao?
**Impacto:** Define se a reimplementacao busca equivalencia literal ou qualidade empirica mensuravel.

**Status:** ✅ Respondida

**Resposta:** Tratar os valores atuais como perfil versionado legacy-v1. Preservá-los para
paridade, mas permitir perfis futuros calibrados. Qualquer recalibração deve ser
baseada em dataset, benchmark, métricas de falsos positivos, economia e proteção
never_worse, sem substituir silenciosamente o perfil legado.
