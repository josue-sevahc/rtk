# Instalacao e Validacao, Tarefas de Implementacao

## Pre-requisitos

- [ ] Ambiente Unix com shell, `curl`, `tar` e uma ferramenta SHA-256.
- [ ] Toolchain Rust e `cargo` para o caminho de instalacao local.
- [ ] Release assets e manifest de checksums publicados para os targets suportados.

## Tarefas

- [ ] T-01, Implementar deteccao de OS/arquitetura e composicao do target de distribuicao.
  - Origem no legado: `install.sh:29-88`
  - Criterio de pronto: plataformas suportadas produzem targets esperados e casos desconhecidos encerram com mensagem de erro.
  - Confianca: 🟢

- [ ] T-02, Implementar resolucao de versao com pin por ambiente, redirect de latest e fallback REST.
  - Origem no legado: `install.sh:48-67`; `install.sh:159-169`
  - Criterio de pronto: `RTK_VERSION` tem precedencia; ausencia de tag nos dois mecanismos encerra o fluxo.
  - Confianca: 🟢

- [ ] T-03, Implementar download, verificacao SHA-256 e validacao de paths antes da extracao.
  - Origem no legado: `install.sh:91-136`
  - Criterio de pronto: checksums ausente/divergente, ferramenta indisponivel ou path traversal impedem a instalacao.
  - Confianca: 🟢

- [ ] T-04, Implementar instalacao local incremental para o binario release.
  - Origem no legado: `scripts/install-local.sh:6-34`
  - Criterio de pronto: fontes mais novas acionam build; caso contrario o binario existente e copiado com permissao `755`.
  - Confianca: 🟢

- [ ] T-05, Implementar verificacao de identidade do binario, recursos CLI e integracoes opcionais.
  - Origem no legado: `scripts/check-installation.sh:20-151`
  - Criterio de pronto: ausencia ou binario sem `gain` retorna falha; recursos ausentes aparecem no resumo.
  - Confianca: 🟢

- [ ] T-06, Implementar guard de comandos alterados sem testes inline, incluindo self-test.
  - Origem no legado: `scripts/check-test-presence.sh`
  - Criterio de pronto: um `*_cmd.rs` alterado sem `#[cfg(test)]` retorna `1`; `--self-test` comprova o detector.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Simular matrix de OS/arquitetura e validar targets e recusas.
  - Origem no legado: `install.sh:29-88`
  - Criterio de pronto: cada entrada suportada mapeia para um target e entradas invalidas falham.
  - Confianca: 🟢

- [ ] TT-02, Testar integridade e seguranca do archive com checksum correto, incorreto e caminho malicioso.
  - Origem no legado: `install.sh:91-121`
  - Criterio de pronto: somente o archive valido chega a fase de extracao.
  - Confianca: 🟢

- [ ] TT-03, Testar caminhos de build local atualizado e desatualizado.
  - Origem no legado: `scripts/install-local.sh:16-21`
  - Criterio de pronto: o build e executado apenas quando pelo menos um arquivo de fonte ou manifesto e mais novo.
  - Confianca: 🟢

- [ ] TT-04, Testar diagnostico com RTK ausente, RTK incorreto e RTK com recursos opcionais ausentes.
  - Origem no legado: `scripts/check-installation.sh:20-151`
  - Criterio de pronto: cada estado produz mensagem e codigo de saida coerentes.
  - Confianca: 🟢

## Ordem Sugerida

1. Construir a deteccao e resolucao de versao antes de integrar downloads.
2. Adicionar checksum e validacao de archive antes de qualquer extracao ou copia.
3. Implementar instalacao local e verificacao; finalizar com o guard de testes.

## Lacunas Pendentes (🔴)

- Confirmar se referencias a fork, branch e hooks antigos devem ser removidas ou atualizadas.
- Executar testes reais em Linux e macOS, com os dois mecanismos de checksum.
