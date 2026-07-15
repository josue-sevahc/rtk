# Execucao Filtrada, Tarefas de Implementacao

> Sequencia para reconstruir o runner compartilhado e garantir que filtros por ferramenta nao alterem semantica de processo, streams ou metricas.

## Pre-requisitos

- [ ] 🟢 Disponibilizar abstrações de processo filho, pipes, TTY, threads e canais na stack alvo. Origem: `src/core/stream.rs`.
- [ ] 🟢 Disponibilizar guard de tokens, tee de recuperacao e tracking best-effort. Origem: `src/core/guard.rs`, `src/core/tee.rs`, `src/core/tracking.rs`.
- [ ] 🟢 Definir convenção de exit code para sinais Unix e comportamento documentado fora de Unix. Origem: `src/core/stream.rs:230-241`.
- [ ] 🔴 Criar ambiente de teste capaz de gerar stdout/stderr grandes, `BrokenPipe`, sinal de termino e stdin piped de forma deterministica. Origem: `src/core/stream.rs`.

## Tarefas

- [ ] T-01, Implementar `RunOptions` e `RunMode` como contrato do runner.
  - Origem no legado: `src/core/runner.rs:20-80`.
  - Criterio de pronto: opcoes cobrem tee, stdout-only, early-exit em falha, newline e heranca de stdin; modos distinguem filtro simples, filtro com exit code, streaming e passthrough.
  - Confianca: 🟢

- [ ] T-02, Implementar conversao de status de processo para exit code.
  - Origem no legado: `src/core/stream.rs:230-241`.
  - Criterio de pronto: status com codigo devolve o mesmo valor; sinal Unix devolve `128+sinal`; ausencia de ambos retorna `1`.
  - Confianca: 🟢

- [ ] T-03, Implementar modo passthrough antes dos modos filtrados.
  - Origem no legado: `src/core/stream.rs:248-271`, `src/core/runner.rs:190-202`.
  - Criterio de pronto: TTY e herdado, nenhum pipe/buffer e criado, `StreamResult` e vazio e tracking persiste apenas duracao com tokens zero.
  - Confianca: 🟢

- [ ] T-04, Implementar captura de stdout/stderr e a protecao de processo filho.
  - Origem no legado: `src/core/stream.rs:272-455`.
  - Criterio de pronto: stdin nulo, herdado ou filtrado e aplicado; `ChildGuard` aguarda o filho; stdout e stderr sao coletados sem deadlock e retornam em `StreamResult`.
  - Confianca: 🟢

- [ ] T-05, Implementar streaming concorrente com `StreamFilter`.
  - Origem no legado: `src/core/stream.rs:300-420`.
  - Criterio de pronto: leitores de stdout/stderr alimentam canal; cada linha chega ao filtro, vai para o descritor correto e `flush`/`on_exit` adicionam texto final.
  - Confianca: 🟢

- [ ] T-06, Implementar limites de 10 MiB para captura de stdout e stderr.
  - Origem no legado: `src/core/stream.rs:244`, `src/core/stream.rs:330-455`.
  - Criterio de pronto: buffers param de crescer ao teto; cada stream emite no maximo um warning; processo e encaminhamento de linhas continuam conforme o modo ativo.
  - Confianca: 🟢

- [ ] T-07, Implementar filtro buffered resiliente a panic e tratamento de escrita em pipe fechado.
  - Origem no legado: `src/core/stream.rs:420-510`.
  - Criterio de pronto: panic do filtro retorna stdout bruto com warning; `BrokenPipe` ao escrever resultado nao converte a execucao em erro; demais erros de I/O continuam visiveis.
  - Confianca: 🟢

- [ ] T-08, Implementar o runner capturado e as politicas de filtro por exit code.
  - Origem no legado: `src/core/runner.rs:80-155`.
  - Criterio de pronto: filtro recebe raw combinado ou stdout-only; `FilteredWithExit` recebe status; `early_exit_on_failure` reemite raw por descritor e nao chama a funcao de filtro.
  - Confianca: 🟢

- [ ] T-09, Integrar tee, guard e tracking com texto efetivamente exibido.
  - Origem no legado: `src/core/runner.rs:12-21`, `src/core/runner.rs:125-155`, `src/core/tracking.rs:1356-1402`.
  - Criterio de pronto: hint e gerado quando configurado; `never_worse` limita a forma final; tracking recebe a mesma saida enviada ao usuario e passthrough permanece neutro.
  - Confianca: 🟢

- [ ] T-10, Expor adaptadores publicos para wrappers especializados.
  - Origem no legado: `src/core/runner.rs:218-296`, `src/core/stream.rs:530-570`.
  - Criterio de pronto: wrappers podem chamar `run_filtered`, `run_filtered_with_exit`, `run_streamed`, `run_passthrough`, `exec_capture` e `exec_capture_stdin` sem duplicar logica de streams/status.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar a conversao de exit codes.
  - Origem no legado: `src/core/stream.rs:581-600`.
  - Criterio de pronto: cobrir `0`, nao zero e termino por sinal em Unix.
  - Confianca: 🟢

- [ ] TT-02, Testar todos os modos de `run_streaming`.
  - Origem no legado: `src/core/stream.rs:663-800`.
  - Criterio de pronto: passthrough, streaming, buffered e capture-only preservam status, dados raw e comportamento de filtro esperado.
  - Confianca: 🟢

- [ ] TT-03, Testar cap de stdout e stderr em 10 MiB.
  - Origem no legado: `src/core/stream.rs:738-780`.
  - Criterio de pronto: cada buffer fica limitado, warning ocorre uma vez e execucao finaliza sem crescimento ilimitado.
  - Confianca: 🟢

- [ ] TT-04, Testar stdin nulo, herdado e filtrado.
  - Origem no legado: `src/core/stream.rs:785-800`, `src/core/stream.rs`.
  - Criterio de pronto: comando que le stdin recebe vazio no modo nulo, entrada do chamador no herdado e linhas transformadas no filtrado.
  - Confianca: 🟢

- [ ] TT-05, Testar `StreamFilter`, `BlockHandler`, `flush` e `on_exit`.
  - Origem no legado: `src/core/stream.rs:887-1120`.
  - Criterio de pronto: linhas descartadas nao aparecem, blocos preservam continuacoes, resumo recebe status e raw corretos.
  - Confianca: 🟢

- [ ] TT-06, Testar panic de filtro, `early_exit_on_failure`, tee e `never_worse`.
  - Origem no legado: `src/core/stream.rs:445-460`, `src/core/runner.rs:99-155`.
  - Criterio de pronto: panic retorna raw, falha com early-exit nao filtra, output maior que raw e substituido e tracking usa o texto exibido.
  - Confianca: 🟢

- [ ] TT-07, Testar captura manual e representacao combinada.
  - Origem no legado: `src/core/stream.rs:808-850`.
  - Criterio de pronto: `exec_capture` cobre sucesso, falha, stderr e `combined`; `exec_capture_stdin` preserva entrada piped quando exigida.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 A unit emite eventos para tracking, mas nao e proprietaria de schema ou migracoes.

## Ordem Sugerida

1. Implementar T-01 a T-04 para estabelecer contratos de processo, status e I/O.
2. Implementar T-05 a T-07 para tornar a camada concorrente e resiliente antes de conectá-la aos filtros de negocio.
3. Implementar T-08 a T-10 para expor o runner para os wrappers com guard, tee e tracking completos.
4. Executar TT-01 a TT-07 em cada alteracao de infraestrutura de streams, pois qualquer regressao afeta todos os ecossistemas.

## Lacunas Pendentes (🔴)

- 🔴 Validar sinais, pipes e `BrokenPipe` na plataforma Windows e nas shells suportadas, pois a conversao por sinal e Unix-especifica.
- 🔴 Determinar se o teto de 10 MiB e suficiente para workloads reais e como comunicar ao usuario quando a filtragem pode estar baseada em captura incompleta.
- 🔴 Exercitar concorrencia e cancelamento sob carga real de stdout/stderr para confirmar ausencia de deadlock fora dos testes estaticos.
