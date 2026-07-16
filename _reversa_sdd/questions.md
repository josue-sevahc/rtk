# Perguntas para Validacao - RTK

> Gerado pelo Revisor em 2026-07-16.
> As perguntas abaixo consolidam lacunas repetidas entre as specs. Responda no chat ou preencha os campos `Resposta`.

## Pergunta 1

**Contexto:** As specs de CLI, wrappers, filtros, hooks e scripts registram compatibilidade externa ainda nao exercitada.
**Specs afetadas:** `entrada-cli/`, `wrappers-comandos/`, `perfis-de-filtros/`, `integracoes-de-agentes/`, `automacao-e-scripts/`
**Pergunta:** Qual baseline deve ser tratado como oficialmente suportado na reconstrucao: sistemas operacionais, shells, agentes, versoes de ferramentas e locales?
**Impacto:** Define quais lacunas viram criterios obrigatorios de paridade e qual matriz de testes precisa existir.

**Resposta:** <!-- aguardando -->

## Pergunta 2

**Contexto:** `is_operational_command()` usa whitelist; um wrapper novo esquecido nao passa pela verificacao de integridade.
**Spec afetada:** `entrada-cli/tasks.md`
**Pergunta:** A reconstrucao deve preservar essa falha aberta ou adotar uma politica fechada por padrao para novos comandos operacionais?
**Impacto:** Altera o modelo de seguranca e os testes do dispatch central.

**Resposta:** <!-- aguardando -->

## Pergunta 3

**Contexto:** A captura para filtro e tracking para de acumular apos 10 MiB por stream e emite warning.
**Spec afetada:** `wrappers-comandos/execucao-filtrada/tasks.md`
**Pergunta:** O teto fixo de 10 MiB deve ser preservado, tornado configuravel ou substituido por outra estrategia?
**Impacto:** Afeta uso de memoria, fidelidade de filtros, tee, tracking e comunicacao de truncamento.

**Resposta:** <!-- aguardando -->

## Pergunta 4

**Contexto:** O codigo atual usa `history.db`, enquanto documentos legados ainda citam `tracking.db`; a unit de nucleo pede uma decisao de migracao.
**Spec afetada:** `nucleo/tasks.md`
**Pergunta:** Em uma reconstrucao, os dados locais existentes devem ser migrados, preservados in-place, importados sob demanda ou descartados?
**Impacto:** Define compatibilidade de schema, caminho do banco e tarefas de migracao.

**Resposta:** <!-- aguardando -->

## Pergunta 5

**Contexto:** URL e token de telemetria sao injetados no build; o cliente tenta `POST /erasure`, mas o backend nao esta neste repositorio.
**Spec afetada:** `nucleo/tracking-e-telemetria/tasks.md`
**Pergunta:** Existe um contrato operacional do backend que deve ser considerado fonte de verdade para ping, retencao e erasure remoto?
**Impacto:** Sem essa fonte, a paridade pode cobrir apenas o comportamento cliente e o fallback por email.

**Resposta:** <!-- aguardando -->

## Pergunta 6

**Contexto:** No plugin OpenClaw, timeout, exit desconhecido e falha operacional viram passthrough silencioso, assim como ausencia de rewrite.
**Spec afetada:** `plugin-openclaw/tasks.md`
**Pergunta:** Essa degradacao silenciosa deve permanecer como politica de producao ou erros operacionais devem gerar aviso/bloqueio distinto?
**Impacto:** Muda disponibilidade, observabilidade e possivelmente a fronteira de seguranca do plugin.

**Resposta:** <!-- aguardando -->

## Pergunta 7

**Contexto:** `rtk learn --write-rules` recompõe `.claude/rules/cli-corrections.md` com `fs::write` e substitui edicoes manuais.
**Spec afetada:** `aprendizado/recomendacoes-de-adocao/tasks.md`
**Pergunta:** A reconstrucao deve preservar a sobrescrita total, mesclar secoes geradas ou recusar a operacao quando detectar edicoes manuais?
**Impacto:** Define propriedade do arquivo, idempotencia e estrategia de conflito.

**Resposta:** <!-- aguardando -->

## Pergunta 8

**Contexto:** O formatter gera textos em ingles, mas nao existe uma politica de localizacao no modulo.
**Spec afetada:** `parser-de-saida/formatacao-estruturada/design.md`
**Pergunta:** A saida estruturada deve permanecer somente em ingles ou a reconstrucao precisa prever localizacao?
**Impacto:** Afeta strings canonicas, snapshots, compatibilidade de parsers consumidores e testes.

**Resposta:** <!-- aguardando -->

## Pergunta 9

**Contexto:** Percentuais de economia, regexes e limiares de descoberta/aprendizado sao heuristicas estaticas sem calibracao de producao disponivel no repositorio.
**Specs afetadas:** `descoberta/`, `aprendizado/`, `perfis-de-filtros/`, `parser-de-saida/`
**Pergunta:** Esses valores devem ser tratados como contrato legado a preservar ou existem dados de producao/benchmarks que devem orientar uma recalibracao?
**Impacto:** Define se a reimplementacao busca equivalencia literal ou qualidade empirica mensuravel.

**Resposta:** <!-- aguardando -->

