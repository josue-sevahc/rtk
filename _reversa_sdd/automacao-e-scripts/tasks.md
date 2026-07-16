# Automacao e Scripts, Tarefas de Implementacao

## Pre-requisitos

- [ ] Ferramentas de shell, `curl`, `tar`, checksum, `cargo` e `git` disponiveis conforme o fluxo escolhido.
- [ ] Binario RTK release e estrutura de build documentados em `design.md`.
- [ ] Ambiente de VM Multipass e Bun disponiveis para o benchmark completo.

## Tarefas

- [ ] T-01, Implementar deteccao de plataforma, resolucao de versao e mapeamento para target de release.
  - Origem no legado: `install.sh:29-88`
  - Criterio de pronto: Linux/Darwin e arquiteturas suportadas resultam no target correto; entradas nao suportadas encerram com erro.
  - Confianca: 🟢

- [ ] T-02, Implementar download verificado, validacao de conteudo do archive e instalacao atomica do binario.
  - Origem no legado: `install.sh:91-151`
  - Criterio de pronto: checksum divergente, ausente ou archive com traversal nao instalam arquivo; um asset valido e instalado como executavel.
  - Confianca: 🟢

- [ ] T-03, Implementar instalacao local incremental e aviso de `PATH`.
  - Origem no legado: `scripts/install-local.sh:6-34`
  - Criterio de pronto: o build e reutilizado quando atualizado, recompilado quando fonte muda e instalado com modo `755`.
  - Confianca: 🟢

- [ ] T-04, Implementar diagnostico de binario, recursos CLI e integracoes de agente.
  - Origem no legado: `scripts/check-installation.sh:20-151`
  - Criterio de pronto: ausencia de RTK ou binario sem `gain` resulta em falha clara; recursos ausentes ficam listados.
  - Confianca: 🟢

- [ ] T-05, Implementar smoke tests, tracking tests e guard de testes inline com contadores e codigos de saida preservados.
  - Origem no legado: `scripts/test-all.sh`, `scripts/test-tracking.sh`, `scripts/check-test-presence.sh`
  - Criterio de pronto: as suites reportam PASS/FAIL/SKIP e o guard falha para `*_cmd.rs` alterado sem `#[cfg(test)]`.
  - Confianca: 🟢

- [ ] T-06, Implementar orquestracao de benchmark VM, fases de qualidade e relatorio de prontidao.
  - Origem no legado: `scripts/benchmark/run.ts`, `scripts/benchmark/lib/vm.ts`, `scripts/benchmark/lib/report.ts`
  - Criterio de pronto: a VM e criada ou reutilizada, as fases sao registradas e o resultado e `READY FOR RELEASE` apenas sem falhas.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar instalacao remota valida com versao pinada e checksum correspondente.
  - Origem no legado: `install.sh:159-177`
  - Criterio de pronto: o binario instalado responde a `--version`.
  - Confianca: 🟢

- [ ] TT-02, Testar recusa de checksum divergente e de entrada de archive com traversal.
  - Origem no legado: `install.sh:91-121`
  - Criterio de pronto: nenhum arquivo e instalado e o processo retorna falha.
  - Confianca: 🟢

- [ ] TT-03, Executar o self-test do guard de presenca de testes.
  - Origem no legado: `scripts/check-test-presence.sh --self-test`
  - Criterio de pronto: o fixture sem teste e detectado pelo guard.
  - Confianca: 🟢

- [ ] TT-04, Validar que falhas de benchmark impedem o veredito de release.
  - Origem no legado: `scripts/benchmark/run.ts`
  - Criterio de pronto: uma fase com falha produz `NOT READY`.
  - Confianca: 🟢

## Ordem Sugerida

1. Implementar primeiro o instalador remoto e local, pois fornecem o binario utilizado pelos demais scripts.
2. Adicionar diagnostico e guardrails locais antes das suites maiores.
3. Fechar com benchmark e relatorios, dependentes do binario, ferramentas externas e ambiente VM.

## Lacunas Pendentes (🔴)

- Validar em runtime os caminhos de instalacao para todas as plataformas e versoes de ferramentas suportadas.
- Decidir se as referencias a fork, branch e caminho de hook ainda devem fazer parte do diagnostico.
