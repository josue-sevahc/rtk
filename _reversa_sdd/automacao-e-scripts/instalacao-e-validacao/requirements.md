# Instalacao e Validacao

## Visao Geral

🟢 Esta subunit define o caminho operacional para disponibilizar o binario RTK, comprovar que ele e o produto esperado e impedir instalacoes ou mudancas de comando sem validacao minima. Ela separa instalacao remota verificada, instalacao local e diagnostico posterior.

## Responsabilidades

- 🟢 Resolver uma release compativel e instala-la somente apos controles de integridade.
- 🟢 Construir e copiar o binario local de release para um diretorio configuravel.
- 🟢 Verificar executabilidade, versao, presenca em `PATH` e identidade funcional pelo comando `gain`.
- 🟢 Inspecionar recursos CLI e integracoes de Claude Code como diagnostico auxiliar.
- 🟢 Bloquear alteracoes de comandos sem testes inline quando o guard e usado.

## Regras de Negocio

- 🟢 `RTK_INSTALL_DIR` define o destino remoto; sem ele, o destino e `~/.local/bin`.
- 🟢 O primeiro argumento da instalacao local define o destino; sem ele, usa `~/.cargo/bin`.
- 🟢 O instalador remoto nao usa API de release se o redirect de latest fornecer uma tag valida.
- 🟢 Sem `RTK_SKIP_CHECKSUM=1`, a ausencia de checksum ou de ferramenta SHA-256 interrompe a instalacao.
- 🟢 A verificacao considera o RTK correto apenas quando `rtk gain` ou `rtk gain --help` funciona.
- 🟡 Recursos ausentes no diagnostico sao warnings e nao necessariamente tornam a instalacao invalida.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|---|---|---|---|
| RF-01 | Resolver e baixar o asset correspondente a OS, arquitetura e versao selecionada. | Must | O nome do asset usa o target correto e a falha de download encerra o fluxo. |
| RF-02 | Garantir integridade e seguranca do archive antes de extrair. | Must | Checksum e estrutura interna sao validados; falhas nao deixam binario instalado. |
| RF-03 | Instalar binario local de release com permissao de execucao e rebuild incremental. | Must | O binario em destino passa a responder `--version`. |
| RF-04 | Diagnosticar localizacao, identidade e capacidades do RTK instalado. | Should | O relatorio diferencia ausencia, binario errado, recursos faltantes e configuracao opcional. |
| RF-05 | Validar teste inline em comandos modificados. | Must | O guard retorna `1` quando detectar `*_cmd.rs` modificado sem `#[cfg(test)]`. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Seguranca | Usar verificacao SHA-256 e recusar paths absolutos ou `..` no archive. | `install.sh:91-121` | 🟢 |
| Disponibilidade | Encerrar no primeiro erro critico em instaladores. | `install.sh:5`; `scripts/install-local.sh:4` | 🟢 |
| Usabilidade | Informar destino, versao e instrucao de `PATH` quando necessario. | `install.sh:153-157`; `scripts/install-local.sh:27-34` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Instalar release segura
Dado uma versao RTK e um archive com checksum valido
Quando o instalador remoto e executado em plataforma suportada
Entao o binario e instalado no diretorio configurado e sua versao e exibida

Cenario: Rejeitar archive inseguro
Dado que a lista do archive contem um caminho absoluto ou com `..`
Quando o instalador valida seu conteudo
Entao ele encerra com erro antes da extracao

Cenario: Identificar binario RTK incorreto
Dado um executavel chamado `rtk` que nao suporta `gain`
Quando a verificacao de instalacao e executada
Entao ela informa que o binario nao e o Rust Token Killer e retorna falha
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Integridade e extracao segura | Must | Protege a fronteira de distribuicao de binarios. |
| Instalacao local e remota | Must | E o caminho para disponibilizar o produto. |
| Guard de testes de comandos | Must | Impede regressao em superficie frequente do CLI. |
| Diagnostico de recursos e integracoes | Should | Facilita suporte, mas nao substitui o fluxo de instalacao. |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---|---|---|
| `install.sh` | instalacao e verificacao remota | 🟢 |
| `scripts/install-local.sh` | instalacao incremental local | 🟢 |
| `scripts/check-installation.sh` | diagnostico de identidade e recursos | 🟢 |
| `scripts/check-test-presence.sh` | validacao de testes inline | 🟢 |
