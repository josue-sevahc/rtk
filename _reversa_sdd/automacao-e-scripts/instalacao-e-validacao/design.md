# Instalacao e Validacao, Design Tecnico

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---|---|---|---|
| `detect_os` | `() -> OS` | `linux` ou `darwin` | Rejeita outros sistemas. |
| `detect_arch` | `() -> ARCH` | `x86_64` ou `aarch64` | Normaliza aliases usuais. |
| `get_target` | `() -> TARGET` | triple Rust | Deriva target de OS e arquitetura. |
| `get_latest_version` | `() -> VERSION` | tag de release | Redirect primeiro; API como fallback. |
| `install` | `() -> ()` | sucesso ou `exit 1` | Baixa, valida e instala a release. |
| `verify` | `() -> ()` | sucesso ou `exit 1` | Confirma executabilidade e alerta sobre PATH. |
| `check_command` | `(cmd, name) -> ()` | atualiza listas | Alimenta diagnostico de funcionalidades. |

## Fluxo Principal

1. 🟢 Definir diretorio de destino por variavel de ambiente ou valor padrao.
2. 🟢 Detectar plataforma e formar o nome do asset release.
3. 🟢 Determinar versao pinada ou latest release.
4. 🟢 Baixar archive e manifest de checksums para diretorio temporario.
5. 🟢 Buscar hash esperado pelo nome do asset e calcular o hash real com a ferramenta disponivel.
6. 🟢 Inspecionar a lista do tarball antes de extrair e recusar entradas inseguras.
7. 🟢 Mover o binario extraido, aplicar permissao `+x`, confirmar `--version` e diagnosticar `PATH`.
8. 🟢 Quando solicitado, o diagnostico verifica `gain`, recursos CLI e configuracao de integracao.

## Fluxos Alternativos

- 🟢 **OS ou arquitetura invalida:** as funcoes de deteccao chamam `error` e encerram.
- 🟢 **Redirect sem tag:** a resolucao tenta a API de releases do GitHub.
- 🟢 **Checksum dispensado:** aviso explicito, sem calculo de hash, quando a variavel de bypass vale `1`.
- 🟢 **Fonte local inalterada:** o instalador local reutiliza `target/release/rtk` sem novo build.
- 🟢 **RTK ausente ou incorreto:** `check-installation.sh` termina com codigo `1`.
- 🟡 **Recurso CLI faltante:** o diagnostico preserva a instalacao como basica e apenas lista recursos ausentes.

## Dependencias

- 🟢 Rede GitHub via `curl` para resolver e baixar releases.
- 🟢 `sha256sum` GNU ou `shasum -a 256` para comprovacao de integridade.
- 🟢 `tar`, `mktemp`, `mkdir`, `mv`, `chmod` para entrega do binario.
- 🟢 `cargo`, `find` e `install` para build e instalacao local.
- 🟢 Shell Bash para scripts locais que usam arrays e redirecionamento especifico.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---|---|---|
| Executar instalador remoto em `sh` portavel e instalacao local em Bash estrito. | `install.sh:1-5`; `scripts/install-local.sh:1-4` | 🟢 |
| Resolver latest por redirect antes da API para evitar rate limit. | `install.sh:48-67` | 🟢 |
| Tratar verificacao de checksum como obrigatoria, com bypass opt-in. | `install.sh:91-114` | 🟢 |
| Checar paths do tar antes da extracao em vez de confiar no archive remoto. | `install.sh:116-121` | 🟢 |
| Reconhecer o produto pelo subcomando `gain`, nao apenas pelo nome do executavel. | `scripts/check-installation.sh:43-57` | 🟢 |

## Estado Interno

- 🟢 `install.sh` usa globais `OS`, `ARCH`, `TARGET`, `VERSION`, `INSTALL_DIR`, URLs e diretorios temporarios durante uma execucao.
- 🟢 `check-installation.sh` mantem flags `CORRECT_RTK`, `GLOBAL_INIT`, `LOCAL_INIT` e arrays `FEATURES`/`MISSING_FEATURES`.
- 🟢 O instalador local calcula `INSTALL_DIR`, `INSTALL_PATH` e `BINARY_PATH` uma vez no inicio.

## Observabilidade

- 🟢 O instalador remoto centraliza mensagens coloridas em `info`, `warn` e `error`.
- 🟢 A verificacao local imprime uma secao por diagnostico, incluindo caminho e versao do binario.
- 🟢 Avisos de PATH, recursos e integracoes opcionais nao escondem a causa quando a instalacao falha.

## Riscos e Lacunas

- 🔴 Nao ha evidencia de teste de integracao contra cada target distribuido.
- 🟢 `check-installation.sh` contem instrucoes de fork/branch e caminho de hook que divergem da topologia atualmente versionada; trata-se de divida documental confirmada no script.
- 🟡 A verificacao de `PATH` informa o padrao `~/.local/bin` mesmo quando outro destino foi configurado no instalador remoto.
