# Automacao e Scripts

## Visao Geral

🟢 Os scripts auxiliares automatizam distribuicao, instalacao, diagnostico, smoke tests, guardrails de testes, validacao documental e benchmarks do RTK. Eles preservam sinais operacionais por codigos de saida e falham cedo quando uma precondicao de seguranca ou ambiente nao e atendida.

## Responsabilidades

- 🟢 Instalar o binario de release com verificacao de integridade e protecao contra extracao insegura.
- 🟢 Construir e instalar uma release local, reutilizando binario atualizado quando possivel.
- 🟢 Diagnosticar a instalacao ativa e distinguir o RTK esperado de outro executavel homonimo.
- 🟢 Executar suites de smoke, tracking e guardrails de testes com resultado automatizavel.
- 🟢 Medir qualidade, compatibilidade e economia de tokens em ambiente local ou VM Multipass.

## Regras de Negocio

- 🟢 Um binario remoto so pode ser instalado apos checksum SHA-256 valido, exceto quando `RTK_SKIP_CHECKSUM=1` foi escolhido explicitamente.
- 🟢 Arquivos compactados com caminho absoluto ou componente `..` sao recusados antes da extracao.
- 🟢 O smoke test exige `rtk` no `PATH` e um repositorio Git como diretorio corrente.
- 🟢 Uma alteracao em `src/cmds/*_cmd.rs` sem testes inline faz o guard retornar falha.
- 🟢 O benchmark em VM declara `READY FOR RELEASE` somente sem falhas.
- 🔴 A compatibilidade dos scripts com todas as ferramentas externas e suas versoes nao foi executada na analise estatica.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|---|---|---|---|
| RF-01 | Obter e instalar a release adequada ao sistema operacional e arquitetura detectados. | Must | A instalacao valida OS/arquitetura, baixa o asset correspondente e torna `rtk` executavel. |
| RF-02 | Permitir instalacao local a partir de `target/release/rtk`, reconstruindo apenas quando a fonte for mais nova. | Must | O script instala um binario release valido e informa sua versao. |
| RF-03 | Verificar uma instalacao ativa e reportar recursos e integracoes ausentes sem mascarar o binario errado. | Should | A verificacao falha quando `rtk gain` nao identifica o produto esperado. |
| RF-04 | Executar smoke tests e guardrails com contadores e codigos de saida confiaveis para automacao. | Must | A suite termina com o numero de falhas; o guard retorna `1` quando encontra comando sem testes. |
| RF-05 | Executar benchmark de release em VM e consolidar seu resultado. | Should | O relatorio identifica falhas e emite o estado de prontidao correspondente. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Seguranca | Validar checksum e rejeitar path traversal antes de extrair a release. | `install.sh:91-126` | 🟢 |
| Disponibilidade | Falhar imediatamente diante de precondicoes e erros de comandos criticos. | `install.sh:5`; `scripts/install-local.sh:4` | 🟢 |
| Performance | Reutilizar build local quando fontes e manifests nao sao mais novos que o binario. | `scripts/install-local.sh:16-21` | 🟢 |
| Disponibilidade | Aplicar timeout na execucao de comandos e no cloud-init do benchmark VM. | `scripts/benchmark/lib/vm.ts`; `scripts/benchmark/run.ts` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Instalar uma release remota verificada
Dado uma plataforma e arquitetura suportadas e checksums disponiveis
Quando o instalador baixa o asset da versao resolvida
Entao ele valida o SHA-256, rejeita caminhos inseguros e instala um binario executavel

Cenario: Recusar uma release sem integridade verificavel
Dado que o checksum esta ausente, divergente ou nao ha ferramenta para calcula-lo
Quando o instalador tenta instalar a release
Entao ele encerra com erro sem extrair nem instalar o arquivo

Cenario: Detectar comando alterado sem teste inline
Dado um arquivo modificado que corresponde a `src/cmds/*_cmd.rs`
Quando o guard de presenca de testes e executado
Entao ele retorna codigo 1 caso o arquivo nao contenha `#[cfg(test)]`
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Instalacao verificada de release | Must | E o caminho de distribuicao e inclui controles de seguranca. |
| Testes e guardrails automatizados | Must | Protegem o comportamento dos wrappers durante mudancas. |
| Instalacao local incremental | Must | Sustenta o ciclo cotidiano de desenvolvimento. |
| Diagnostico detalhado da instalacao | Should | E importante para suporte, mas ha alternativas manuais. |
| Benchmark completo em VM | Should | E relevante para release, porem depende de ambiente externo. |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---|---|---|
| `install.sh` | `detect_os`, `detect_arch`, `get_latest_version`, `install`, `verify` | 🟢 |
| `scripts/install-local.sh` | fluxo principal de build e instalacao | 🟢 |
| `scripts/check-installation.sh` | verificacoes de binario, recursos e integracoes | 🟢 |
| `scripts/test-all.sh` | smoke test local | 🟢 |
| `scripts/check-test-presence.sh` | guard de testes inline | 🟢 |
| `scripts/benchmark/run.ts` | orquestracao de benchmark em VM | 🟢 |
