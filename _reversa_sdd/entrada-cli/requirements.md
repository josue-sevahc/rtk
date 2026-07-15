# Entrada CLI

> Contrato operacional extraido do modulo `main` do RTK. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Visao Geral

🟢 A unit `entrada-cli` recebe a invocacao do binario `rtk`, interpreta a CLI, aplica verificacoes transversais e despacha o comando para o modulo responsavel. Quando a invocacao nao pertence ao proprio RTK, ela preserva a utilidade do binario como proxy ao tentar um filtro TOML conhecido ou executar a ferramenta externa em passthrough.

🟡 Esta fronteira concentra a compatibilidade observavel do processo: codigo de saida, sinais Unix, argumentos reconstruidos e a decisao entre tratamento RTK ou delegacao externa.

## Responsabilidades

- 🟢 Definir a superficie de comandos, subcomandos e flags globais da CLI por meio de `Cli`, `Commands` e enums especializados.
- 🟢 Restaurar o comportamento padrao de `SIGPIPE` no Unix e encerrar o processo com o codigo devolvido por `run_cli()`.
- 🟢 Acionar telemetria elegivel, aviso de hook e verificacao de integridade antes do dispatch aplicavel.
- 🟢 Despachar comandos RTK para `cmds`, `core`, `hooks`, `analytics`, `discover` e `learn`.
- 🟢 Tratar falhas de parse como fallback somente para ferramentas externas; meta-comandos RTK invalidos permanecem erros de parse.
- 🟢 Fornecer caminhos de execucao crua controlada por `run` e `proxy`, preservando a saida e o status do processo filho.

## Regras de Negocio

- 🟢 Um meta-comando RTK com argumentos invalidos nunca pode virar execucao externa pelo fallback.
- 🟢 Comandos classificados como operacionais executam `hooks::integrity::runtime_check()` antes do dispatch; a classificacao usa whitelist explicita.
- 🟢 O aviso de hook potencialmente desatualizado e pulado para `gain` e aplicado aos demais comandos parseados.
- 🟢 No fallback, filtros TOML sao ignorados quando `RTK_NO_TOML` estiver ativo ou nao houver filtro correspondente; nesses casos a ferramenta externa recebe stdio herdado.
- 🟢 O fallback registra falha de parse silenciosamente e devolve `127` quando a ferramenta externa nao pode ser executada.
- 🟢 `proxy` espelha stdout e stderr em tempo real, mas limita a captura para tracking a 1 MiB por stream.
- 🟢 Em Unix, `proxy` encerra e aguarda o filho ao receber `SIGINT` ou `SIGTERM`, restaura o handler padrao e reemite o sinal.
- 🟢 Filtros globais de `pnpm` sao ignorados no subcomando `typecheck`, com aviso explicito.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|-----------|-------------------|
| RF-01 | 🟢 O processo deve restaurar `SIGPIPE` no Unix, executar a CLI e terminar com o codigo retornado; erros nao tratados devem ser exibidos como `rtk: <erro>` e encerrar com `1`. | Must | Dado um handler de `run_cli()` que retorna um codigo, quando o binario encerra, entao o processo expõe esse codigo; dado um erro, entao stderr recebe a mensagem prefixada e o codigo e `1`. |
| RF-02 | 🟢 A CLI deve aceitar os comandos RTK definidos por `Commands`, as flags globais `verbose`, `ultra_compact` e `skip_env`, e encaminhar cada comando parseado ao modulo especializado correspondente. | Must | Dado um subcomando valido, quando `Cli::try_parse()` o interpreta, entao o ramo de dispatch correspondente e chamado com os argumentos preservados. |
| RF-03 | 🟢 Antes do dispatch, a CLI deve tentar o ping de telemetria e, exceto em `gain`, verificar se ha aviso de hook; para comandos operacionais deve executar a verificacao de integridade. | Must | Dado um comando operacional valido, quando a CLI o processa, entao `maybe_ping`, a verificacao de hook aplicavel e `runtime_check` ocorrem antes do handler; dado `gain`, entao o aviso de hook nao e solicitado. |
| RF-04 | 🟢 Uma falha comum de parse deve ser encaminhada ao fallback apenas quando ha argumentos e o primeiro token nao e um meta-comando RTK. | Must | Dado `rtk comando-externo ...`, quando o parse falha, entao o fallback tenta filtro ou passthrough; dado `rtk init --flag-invalida`, entao o erro de parse e preservado e nenhum processo externo e iniciado. |
| RF-05 | 🟢 O fallback deve normalizar o basename do executavel para procurar filtro TOML, aplicar o filtro quando elegivel, registrar tracking e devolver o exit code real do comando. | Must | Dado um comando externo com filtro TOML elegivel, quando ele e executado com sucesso, entao sua saida filtrada ou de recuperacao e emitida, o tracking e registrado e o status devolvido coincide com o do filho. |
| RF-06 | 🟢 O fallback deve executar em passthrough com stdin/stdout/stderr herdados quando TOML estiver desabilitado ou nao houver filtro correspondente. | Must | Dado `RTK_NO_TOML` ativo ou um comando sem filtro, quando o parse falha, entao os streams do processo filho sao herdados e o codigo de saida e preservado. |
| RF-07 | 🟢 O dispatch deve preservar adaptacoes de argumentos que protegem a semantica das ferramentas, incluindo opcoes globais de Git, filtros de `pnpm`, namespace/logs de Kubernetes/OpenShift e deteccao de ferramentas `npx` conhecidas. | Must | Dado um comando dessas familias, quando o dispatch ocorre, entao os argumentos reconstruidos respeitam as regras do helper correspondente antes da chamada ao wrapper. |
| RF-08 | 🟢 `proxy` deve aceitar uma lista de argumentos ou uma unica string separavel por shell, executar o filho sem filtro especializado, espelhar os streams e registrar o resultado. | Should | Dado `rtk proxy <comando>`, quando o filho produz stdout/stderr e termina, entao os streams aparecem em tempo real, o tracking e gravado e o codigo de saida do filho e devolvido. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Compatibilidade | 🟢 Códigos de saida de handlers, fallback e processos filhos devem permanecer observaveis ao chamador para que RTK nao transforme falha externa em sucesso. | `src/main.rs`, `src/core/runner.rs` | 🟢 |
| Desempenho | 🟢 O caminho `proxy` deve limitar a memoria de captura para tracking a 1 MiB por stream, sem deixar de espelhar a saida integral em tempo real. | `src/main.rs:2554` | 🟢 |
| Disponibilidade | 🟢 Telemetria e tentativa nao bloqueante; a execucao da CLI nao deve depender de envio remoto bem-sucedido. | `src/main.rs`, `src/core/telemetry.rs` | 🟢 |
| Seguranca | 🟢 A integridade de hooks e verificada antes de comandos operacionais, bloqueando a operacao quando o estado nao passa na verificacao. | `src/main.rs:1565`, `src/hooks/integrity.rs` | 🟢 |
| Robustez | 🟢 O fallback falha fechado para a superficie propria do RTK e falha aberto somente para ferramentas externas, evitando que erro de uso interno execute comandos inesperados. | `src/main.rs:1257` | 🟢 |
| Portabilidade | 🟢 O tratamento explicito de `SIGPIPE`, `SIGINT` e `SIGTERM` e condicionado a Unix; o comportamento equivalente fora desse ambiente nao foi extraido. | `src/main.rs` | 🔴 |

## Criterios de Aceitacao

```gherkin
Cenario: Despachar um comando RTK valido
  Dado uma invocacao valida de um subcomando RTK
  Quando a CLI conclui o parse
  Entao ela executa as verificacoes transversais aplicaveis
  E encaminha o comando ao handler especializado
  E devolve o codigo de saida informado pelo handler

Cenario: Filtrar uma ferramenta externa desconhecida da CLI
  Dado uma invocacao cujo primeiro token nao e um meta-comando RTK
  E existe um filtro TOML elegivel para o basename da ferramenta
  Quando o parse da CLI falha
  Entao o fallback executa a ferramenta e aplica o filtro
  E registra tracking e falha de parse sem interromper o fluxo
  E devolve o codigo de saida real da ferramenta

Cenario: Rejeitar uso invalido de meta-comando
  Dado uma invocacao invalida de um meta-comando RTK
  Quando o parse da CLI falha
  Entao o erro de parse do Clap e preservado
  E nenhum comando externo e iniciado pelo fallback

Cenario: Preservar erro de execucao externa
  Dado uma ferramenta externa que nao pode ser encontrada no fallback
  Quando o RTK tenta executa-la
  Entao a execucao devolve codigo 127
  E a falha e registrada sem declarar sucesso
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| 🟢 Parse, fallback seguro e dispatch de CLI | Must | E o caminho de entrada de toda funcionalidade executavel do RTK. |
| 🟢 Preservacao de codigos de saida e sinais | Must | A compatibilidade com ferramentas externas depende desse contrato. |
| 🟢 Verificacao de integridade para comandos operacionais | Must | Protege a fronteira de execucao de hooks adulterados. |
| 🟢 Adaptacao de argumentos por ecossistema | Should | E importante para wrappers suportados, mas existe passthrough como alternativa para casos nao reconhecidos. |
| 🟢 Modo `proxy` com tracking limitado | Should | Amplia a observabilidade sem ser a unica rota de execucao do produto. |
| 🟢 Aviso de hook desatualizado | Could | E preventivo; a operacao principal segue submetida a verificacao de integridade quando aplicavel. |

> 🟡 As prioridades foram inferidas pela posicao do `main` na cadeia de chamadas, pelos invariantes de seguranca e pelas rotas de fallback observadas.

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/main.rs` | `main`, `run_cli`, `Cli`, `Commands` | 🟢 |
| `src/main.rs` | `run_fallback`, `is_operational_command` | 🟢 |
| `src/main.rs` | `shell_split`, `build_k8s_namespace_args`, `build_k8s_logs_args` | 🟢 |
| `src/main.rs` | `merge_pnpm_args`, `merge_pnpm_args_os`, `validate_pnpm_filters` | 🟢 |
| `src/main.rs` | handler de `Commands::Proxy` e propagacao de sinais Unix | 🟢 |
| `src/core/telemetry.rs` | `maybe_ping` invocado pela entrada da CLI | 🟢 |
| `src/hooks/integrity.rs` | `runtime_check` invocado para comandos operacionais | 🟢 |
| `src/cmds/` | handlers especializados chamados pelo dispatch | 🟢 |
