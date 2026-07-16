# Automacao e Scripts, Design Tecnico

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---|---|---|---|
| `install.sh:detect_os` | `() -> OS global` | sucesso ou erro | Aceita Linux e Darwin. |
| `install.sh:detect_arch` | `() -> ARCH global` | sucesso ou erro | Aceita x86_64/amd64 e arm64/aarch64. |
| `install.sh:get_latest_version` | `() -> VERSION global` | tag de release | Usa redirect GitHub e fallback REST. |
| `install.sh:install` | `() -> instalacao` | sucesso ou erro | Faz download, verificacao, extracao e permissao. |
| `install.sh:verify` | `() -> diagnostico` | sucesso ou erro | Executa `rtk --version` e avisa sobre `PATH`. |
| `scripts/install-local.sh` | `[diretorio] -> instalacao local` | sucesso ou erro | Reconstroi somente quando necessario. |
| `scripts/check-test-presence.sh` | `[--self-test] -> codigo de saida` | `0` ou `1` | Protege arquivos de comandos alterados. |
| `scripts/benchmark/run.ts` | `() -> relatorio de benchmark` | estado de release | Orquestra VM Multipass e fases de validacao. |

## Fluxo Principal

1. 🟢 `install.sh` detecta OS e arquitetura e deriva o target da release.
2. 🟢 Resolve `VERSION` a partir de `RTK_VERSION` ou do endpoint de latest release.
3. 🟢 Baixa archive e `checksums.txt` para diretorio temporario.
4. 🟢 Calcula e compara SHA-256, salvo bypass explicito por `RTK_SKIP_CHECKSUM=1`.
5. 🟢 Lista o archive e interrompe caso encontre caminho absoluto ou traversal.
6. 🟢 Extrai, instala em `RTK_INSTALL_DIR` ou `~/.local/bin`, concede permissao e verifica o binario.
7. 🟢 Os scripts de validacao exercitam o binario e devolvem status adequado a CI ou ao operador.

## Fluxos Alternativos

- 🟢 **Versao pinada:** `RTK_VERSION` evita a resolucao de latest release.
- 🟢 **Redirect indisponivel:** `get_latest_version` consulta a API REST do GitHub.
- 🟢 **Checksum ignorado:** o instalador continua apenas com `RTK_SKIP_CHECKSUM=1` e imprime aviso.
- 🟢 **Build local atualizado:** `install-local.sh` evita `cargo build --release` se fontes e manifests nao sao mais novos.
- 🟢 **Precondicao ausente:** smoke tests falham se nao houver `rtk` no `PATH` ou repositorio Git.
- 🟢 **Falhas de benchmark:** o relatorio troca o veredito para `NOT READY`.

## Dependencias

- 🟢 `curl`, `tar`, `grep`, `sed`, `awk` e `mktemp`: download, verificacao e extracao no instalador remoto.
- 🟢 `sha256sum` ou `shasum`: calculo da integridade do asset.
- 🟢 `cargo`: build da instalacao local e etapas de qualidade.
- 🟢 `git` e o binario `rtk`: precondicoes e alvos dos smoke tests.
- 🟢 `bun` e `multipass`: execucao e provisionamento do benchmark de VM.
- 🟡 `hyperfine`, `ccusage`, `jq`, `bc` e `numfmt`: medicao e relatorios quando os scripts correspondentes sao executados.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Preferir redirect de release a API para reduzir dependencia de rate limit. | `install.sh:48-67` | 🟢 |
| Exigir checksum por padrao e tornar o bypass visivel ao operador. | `install.sh:91-114` | 🟢 |
| Validar nomes internos antes da extracao para mitigar CWE-22. | `install.sh:116-121` | 🟢 |
| Reusar binario local quando a arvore de fonte nao mudou. | `scripts/install-local.sh:16-21` | 🟢 |
| Isolar benchmark de release em VM reutilizavel. | `scripts/benchmark/run.ts`; `scripts/benchmark/lib/vm.ts` | 🟢 |

## Estado Interno

- 🟢 O instalador remoto mantem `OS`, `ARCH`, `TARGET`, `VERSION`, URLs, diretorio temporario e diretorio de instalacao em variaveis de shell.
- 🟢 Os smoke tests acumulam `PASS`, `FAIL`, `SKIP` e a lista de falhas.
- 🟢 O benchmark representa a VM, resultados de teste e informacoes de build por contratos TypeScript (`VmInfo`, `TestResult`, `TestStatus`, `BuildInfo`).

## Observabilidade

- 🟢 O instalador emite mensagens `[INFO]`, `[WARN]` e `[ERROR]`; erros encerram com codigo 1.
- 🟢 Smoke tests e benchmarks consolidam contadores, falhas e veredito final para consumo humano e automatizado.
- 🟢 `check-installation.sh` apresenta caminho, versao, funcionalidades e integracoes encontradas.

## Riscos e Lacunas

- 🔴 Os scripts de rede, VM e benchmark pesado nao foram executados durante a analise.
- 🟢 `check-installation.sh` ainda menciona um fork e `feat/all-features`; a referencia legada esta presente diretamente no script.
- 🟢 `validate-docs.sh` procura um hook em `.claude/hooks/`, enquanto a configuracao atual registra hooks em `.github/hooks/`; a divergencia de caminhos e observavel no repositorio.
