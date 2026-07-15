# Execucao Filtrada

> Contrato operacional do caminho compartilhado que executa um processo externo, coleta ou transmite seus streams, aplica filtro e registra a representacao realmente mostrada. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Visao Geral

🟢 Este caso de uso sustenta os wrappers especializados com quatro modos de execucao: filtro capturado, filtro ciente de exit code, streaming e passthrough. Ele coordena stdin, stdout, stderr, limites de captura, recuperação por tee, guard de tokens e tracking.

🟢 O objetivo e reduzir a saida de ferramentas externas sem alterar seu status, perder detalhes de forma irrecuperavel ou registrar uma resposta diferente da que foi entregue ao usuario.

## Responsabilidades

- 🟢 Executar processos filhos em modo capturado, streaming ou TTY herdado, conforme a estrategia escolhida pelo wrapper.
- 🟢 Encaminhar stdin nulo, herdado ou filtrado e coordenar stdout/stderr de maneira consistente com o modo de filtro.
- 🟢 Aplicar funcoes de filtro ao output completo ou por linha, finalizando filtros stateful apos o processo.
- 🟢 Limitar a captura usada para filtragem/tracking a 10 MiB por stream e avisar quando o limite for atingido.
- 🟢 Proteger a saida por `never_worse`, gerar hints de tee quando configurado e rastrear exatamente o texto exibido.
- 🟢 Preservar exit codes, inclusive conversao de sinal Unix para `128 + sinal`, e tolerar `BrokenPipe` ao escrever output.

## Regras de Negocio

- 🟢 `Filtered` recebe texto capturado; `FilteredWithExit` recebe texto e exit code; `Streamed` recebe linhas incrementalmente; `Passthrough` herda o TTY sem captura.
- 🟢 No modo filtrado, `RunOptions::filter_stdout_only()` entrega apenas stdout ao filtro e deixa stderr como fluxo separado para fins de exibicao/tracking.
- 🟢 `RunOptions::early_exit_on_failure()` evita filtragem quando o filho falha e reemite stdout/stderr brutos antes de rastrear raw contra raw.
- 🟢 `RunOptions::inherit_stdin()` encaminha stdin do RTK ao filho; sem essa opcao, o caminho capturado nao recebe input interativo/piped do chamador.
- 🟢 `RunOptions::with_tee` e `tee` habilitam hint de recuperacao; `no_trailing_newline` controla a forma de impressao quando nao ha tee.
- 🟢 Filtro buffered que entra em panic deve emitir aviso e retornar stdout bruto em vez de abortar a execucao.
- 🟢 Captura de stdout e stderr possui limite de 10 MiB cada; exceder o limite emite warning e impede crescimento adicional do buffer.
- 🟢 Em passthrough, raw e filtered ficam vazios e tracking registra apenas duracao com tokens zero.
- 🟢 A representacao exibida apos guard, inclusive hint, e a mesma enviada ao tracking de economia.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|-----------|-------------------|
| RF-01 | 🟢 A execucao deve suportar os modos `Filtered`, `FilteredWithExit`, `Streamed` e `Passthrough`. | Must | Dado cada modo, quando um processo filho termina, entao o modo aplica sua estrategia de entrada/saida e devolve o exit code do filho. |
| RF-02 | 🟢 O caminho capturado deve permitir filtro sobre raw combinado ou somente stdout e expor o exit code ao filtro quando solicitado. | Must | Dado `filter_stdout_only`, quando stdout e stderr existem, entao a funcao recebe stdout; dado `FilteredWithExit`, entao recebe o status convertido do filho. |
| RF-03 | 🟢 O caminho streaming deve alimentar um `StreamFilter` linha a linha, aplicar `flush` e permitir resumo de saida dependente do status. | Must | Dado log em varias linhas, quando o filho escreve streams, entao linhas filtradas aparecem progressivamente e o texto de `flush`/`on_exit` e incluido ao final. |
| RF-04 | 🟢 A execucao deve preservar codigos de saida normais e converter termino por sinal Unix para `128 + sinal`. | Must | Dado filho que retorna `0`, `1` ou e terminado por sinal, quando o runner conclui, entao o valor devolvido corresponde a essas convencoes. |
| RF-05 | 🟢 A execucao deve limitar a captura de stdout e stderr a 10 MiB por stream, mantendo o processo funcional apos exceder o limite. | Must | Dado stream superior a 10 MiB, quando a captura atinge o teto, entao apenas um warning e emitido por stream e o runner conclui sem crescer o buffer. |
| RF-06 | 🟢 Filtro buffered que entra em panic deve degradar para stdout bruto e registrar aviso em vez de derrubar o wrapper. | Must | Dado uma funcao de filtro que entra em panic, quando o runner a chama, entao a saida exibida e raw stdout e a execucao conserva o exit code do filho. |
| RF-07 | 🟢 Quando tee estiver configurado, a execucao deve fornecer hint de recuperacao e aplicar `never_worse`; sem tee, deve aplicar o guard diretamente ao texto filtrado. | Must | Dado output filtrado lossy, quando existe tee, entao hint e anexado; dado texto filtrado+hints maior que raw, entao a saida exibida e raw. |
| RF-08 | 🟢 `early_exit_on_failure` deve reemitir output bruto e ignorar o filtro para status nao zero. | Should | Dado opcao ativa e filho falho, quando ele conclui, entao stdout/stderr brutos sao emitidos e tracking compara raw com raw. |
| RF-09 | 🟢 Passthrough deve herdar streams e stdin conforme configuracao, sem transformar output nem contabilizar tokens. | Should | Dado passthrough, quando o filho escreve ou le stdin, entao o TTY e preservado, raw/filtered retornam vazios e tracking registra tokens zero. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Desempenho | 🟢 Streaming entrega output progressivamente; buffers de captura sao limitados a 10 MiB por stream. | `src/core/stream.rs:230-455` | 🟢 |
| Robustez | 🟢 `ChildGuard` aguarda o filho no drop, e filtros buffered que panicam caem para raw stdout. | `src/core/stream.rs`, `src/core/stream.rs` | 🟢 |
| Compatibilidade | 🟢 O runner devolve status real ou convencao `128+sinal`, sem transformar falha externa em sucesso. | `src/core/stream.rs:230-241`, `src/core/runner.rs` | 🟢 |
| Fidelidade | 🟢 `never_worse` e tee fazem a resposta final preservar recuperabilidade e nao exceder a referencia em tokens estimados. | `src/core/runner.rs:12-21`, ADR 001 | 🟢 |
| Disponibilidade | 🟢 `BrokenPipe` durante escrita de output e tratado como encerramento aceitavel para evitar falha do RTK em pipelines curtos. | `src/core/stream.rs:230-455` | 🟢 |
| Observabilidade | 🟢 Tracking usa o texto efetivamente mostrado; warnings sinalizam capture truncada ou panic de filtro. | `src/core/runner.rs`, `src/core/stream.rs` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Filtrar output capturado com status preservado
  Dado um processo filho com stdout e stderr
  Quando o runner usa o modo Filtered
  Entao o filtro recebe o texto configurado
  E o texto emitido passa pelo guard de tokens
  E o codigo devolvido coincide com o status do filho

Cenario: Emitir log em streaming
  Dado um processo que produz linhas continuamente
  Quando o runner usa Streamed com StreamFilter
  Entao cada linha relevante e emitida sem aguardar o fim do processo
  E o filtro e finalizado com flush e resumo de saida

Cenario: Recuperar panic de filtro buffered
  Dado um filtro buffered que entra em panic
  Quando o processo conclui
  Entao o runner avisa sobre o panic
  E exibe stdout bruto
  E preserva o exit code do processo

Cenario: Limitar captura grande
  Dado stdout ou stderr maior que 10 MiB
  Quando o runner coleta o stream
  Entao ele emite um unico warning para o stream
  E nao adiciona mais dados ao buffer correspondente
  E continua a execucao do processo

Cenario: Delegar em passthrough
  Dado um wrapper em modo Passthrough
  Quando a ferramenta e executada
  Entao seus streams sao herdados sem filtragem
  E tracking registra apenas a duracao com tokens zero
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| 🟢 Modos de execucao, streams e exit code preservado | Must | E a base compartilhada de todos os wrappers externos. |
| 🟢 Guard, tee e tracking coerente | Must | Sustenta a promessa de economia sem perda irrecuperavel. |
| 🟢 Limite de captura e recuperacao de panic | Must | Impede consumo de memoria ou falha do processo por output/filtro anormal. |
| 🟢 `early_exit_on_failure` | Should | Protege diagnostico de falhas em wrappers que optam por nao resumir erro. |
| 🟢 Passthrough com metricas neutras | Should | Mantem compatibilidade quando filtragem nao e segura ou desejada. |

> 🟡 As prioridades foram inferidas pelo uso transversal do runner em todos os ecossistemas e pelos guardrails visiveis no codigo.

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/core/runner.rs` | `RunOptions`, `RunMode`, `run_captured_filter`, `run` | 🟢 |
| `src/core/runner.rs` | `run_filtered`, `run_filtered_with_exit`, `run_streamed`, `run_passthrough` | 🟢 |
| `src/core/stream.rs` | `status_to_exit_code`, `RAW_CAP`, `run_streaming` | 🟢 |
| `src/core/stream.rs` | `StreamFilter`, `BlockHandler`, `FilterMode`, `StdinMode`, `StreamResult` | 🟢 |
| `src/core/guard.rs` | `never_worse` | 🟢 |
| `src/core/tee.rs` | `tee_and_hint` e recuperacao de output | 🟢 |
| `src/core/tracking.rs` | `TimedExecution` e registro de economia/passthrough | 🟢 |
| `_reversa_sdd/adrs/001-saida-filtrada-nunca-piora-a-saida-bruta.md` | invariante transversal de saida | 🟢 |
