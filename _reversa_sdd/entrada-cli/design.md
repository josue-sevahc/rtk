# Entrada CLI, Design Tecnico

> Implementacao observada em `src/main.rs`. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Interface

🟢 A interface primaria e o processo local `rtk`, implementado por `main()` e `run_cli()`. A entrada vem de `std::env::args()` para o fallback e de `Cli::try_parse()` para a superficie declarada pelo Clap; a saida e um codigo inteiro de processo, stdout/stderr e, nos caminhos rastreados, eventos para o tracking local.

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `main` | `()` | nunca retorna | 🟢 Restaura `SIGPIPE` em Unix, chama `run_cli()` e encerra com codigo de processo. |
| `run_cli` | `() -> Result<i32>` | codigo de saida ou erro | 🟢 Orquestra telemetria, parse, avisos, integridade e dispatch. |
| `run_fallback` | `(parse_error: clap::Error) -> Result<i32>` | codigo de saida ou erro | 🟢 Trata somente falhas comuns de parse que possam representar ferramenta externa. |
| `shell_split` | `(&str) -> Vec<String>` | tokens shell-like | 🟢 Delega ao lexer de `discover` e respeita aspas simples e duplas. |
| `build_k8s_namespace_args` | `(Option<String>, bool) -> Vec<String>` | argumentos Kubernetes | 🟢 Gera `-A` ou `-n <namespace>`, com prioridade para todos os namespaces. |
| `build_k8s_logs_args` | `(String, Option<String>) -> Vec<String>` | argumentos de logs | 🟢 Gera `<pod>` e inclui `-c <container>` quando informado. |
| `merge_pnpm_args` | `(&[String], &[String]) -> Vec<String>` | argumentos de `pnpm` | 🟢 Converte filtros globais em `--filter=<valor>` antes dos demais argumentos. |
| `validate_pnpm_filters` | `(&[String], &PnpmCommands) -> Option<String>` | aviso opcional | 🟢 Rejeita semanticamente filtros globais em `typecheck` por meio de aviso e posterior ignoracao. |
| `is_operational_command` | `(&Commands) -> bool` | classificacao booleana | 🟢 Whitelist que decide se `runtime_check` deve ocorrer. |

## Fluxo Principal

1. 🟢 `main()` registra `SIG_DFL` para `SIGPIPE` em Unix, evitando aborto ao escrever para um pipe fechado, como em `rtk git log | head`.
2. 🟢 `main()` chama `run_cli()`; converte qualquer `Err` em mensagem `rtk: {:#}` e codigo `1`, ou encerra com o codigo inteiro retornado.
3. 🟢 `run_cli()` chama `core::telemetry::maybe_ping()` de maneira fire-and-forget antes do parse.
4. 🟢 `Cli::try_parse()` produz `Cli` para comandos validos; `DisplayHelp` e `DisplayVersion` delegam o encerramento ao Clap, enquanto outros erros seguem para `run_fallback()`.
5. 🟢 Para um comando parseado, `hooks::hook_check::maybe_warn()` e chamado exceto para `Commands::Gain`.
6. 🟢 `is_operational_command()` consulta uma whitelist de wrappers que passam pela pipeline de hook; se verdadeiro, `hooks::integrity::runtime_check()` precisa concluir sem erro antes do dispatch.
7. 🟢 O `match cli.command` encaminha a invocacao para o modulo correspondente e normaliza seu resultado para `i32`; alguns ramos preparam argumentos antes de delegar.
8. 🟢 O codigo do handler retorna a `main()`, que o torna o status final do processo.

## Fluxo de Fallback

1. 🟢 `run_fallback()` coleta os argumentos crus, removendo o nome do binario.
2. 🟢 Sem argumentos, ou quando o primeiro token pertence a `RTK_META_COMMANDS`, o erro original do Clap encerra o processo; nao ha tentativa de executar um binario homonimo no `PATH`.
3. 🟢 Para outros tokens, a unit monta `raw_command`, remove ANSI da mensagem de parse e inicia `TimedExecution`.
4. 🟢 O primeiro token e normalizado pelo basename para permitir que um caminho absoluto case com filtros TOML, como `/usr/bin/make` com `make`.
5. 🟢 Se `RTK_NO_TOML` estiver desligado e houver filtro, o processo filho captura stdout e, quando `filter_stderr` esta ativo, stderr; a entrada do usuario e herdada.
6. 🟢 A saida capturada passa por `apply_filter_with_info`; quando ocorre perda de informacao, `tee` tenta fornecer recuperacao. Sem hint recuperavel, `emit_guarded` mostra a saida bruta em vez de truncamento irrecuperavel.
7. 🟢 O resultado e rastreado, a falha de parse e registrada silenciosamente e o exit code do processo filho e devolvido.
8. 🟢 Sem filtro TOML, o filho recebe os tres streams herdados; o timer registra passthrough e o status do filho e convertido em exit code.
9. 🟢 Falha de spawn em ambos os ramos registra parse failure, imprime `[rtk: <erro>]` e devolve `127`.

## Fluxo de Proxy

1. 🟢 `Commands::Proxy` exige ao menos um argumento; caso contrario devolve erro com uso esperado.
2. 🟢 Com um unico argumento contendo espacos, `shell_split()` separa nome e argumentos respeitando aspas; com varios argumentos, o primeiro e o executavel.
3. 🟢 Em Unix, um `AtomicU32` guarda o PID do filho e handlers de `SIGINT`/`SIGTERM` matam, aguardam e reemitem o sinal.
4. 🟢 O filho e iniciado com stdout e stderr canalizados; duas threads leem blocos de 8 KiB, espelham cada bloco imediatamente e capturam no maximo 1.048.576 bytes por stream.
5. 🟢 A thread principal aguarda o filho e as duas threads; falha de join e convertida em erro explicito.
6. 🟢 A porcao capturada e concatenada e rastreada como entrada e saida identicas, pois o modo nao aplica filtro especializado.
7. 🟢 O status final do filho e retornado ao chamador.

## Fluxos Alternativos

- 🟢 **Help ou version:** `Cli::try_parse()` recebe erro de exibicao e o Clap decide a apresentacao e o encerramento sem fallback.
- 🟢 **Meta-comando com sintaxe invalida:** `run_fallback()` chama `parse_error.exit()` e impede execucao crua.
- 🟢 **Filtro TOML com stderr integrado:** quando `filter.filter_stderr` e verdadeiro, stdout e stderr sao combinados antes da transformacao para remover banners ou ruido que atravessam streams.
- 🟢 **Filtragem com perda:** falha do comando sempre tenta criar hint via `tee`; em sucesso, o tipo de perda decide entre nenhum hint, hint de cauda ou hint integral.
- 🟢 **`pnpm typecheck` com filtros:** a validacao gera aviso; os filtros precedentes nao sao aplicados por suporte ainda incompleto.
- 🟢 **Comando nao operacional:** comandos de administracao, como `init`, `gain`, `verify` e `config`, ignoram `runtime_check` por nao pertencerem a pipeline de hook.
- 🔴 **Sinais e proxy fora de Unix:** o comportamento equivalente nao foi confirmado estaticamente para Windows.

## Dependencias

- 🟢 `clap`: declara a superficie CLI e diferencia ajuda/versao de falhas comuns de parse.
- 🟢 `core::telemetry`: recebe a tentativa inicial de ping e nao deve bloquear o fluxo principal.
- 🟢 `core::toml_filter`, `core::runner` e `core::tee`: identificam, aplicam e protegem o fallback filtrado contra perda nao recuperavel.
- 🟢 `core::tracking`: mede execucao, registra comandos e anota parse failures.
- 🟢 `core::utils`: resolve executaveis, remove ANSI e converte `ExitStatus`/`Output` em codigos de saida.
- 🟢 `hooks::hook_check` e `hooks::integrity`: expõem aviso preventivo e gate de integridade para comandos operacionais.
- 🟢 `cmds`, `analytics`, `discover` e `learn`: recebem os ramos especializados do dispatch central.
- 🟢 `discover::lexer`: fornece o tokenizador usado pelo modo `proxy`.
- 🟢 `libc` em Unix: restaura `SIGPIPE` e instala handlers de sinais do filho proxy.

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| O CLI separa erro de uso interno de comando externo desconhecido para impedir escalonamento acidental pelo `PATH`. | `src/main.rs:1249` e `core::constants::RTK_META_COMMANDS` | 🟢 |
| O fallback usa o basename do executavel para que caminhos absolutos possam reutilizar filtros declarativos. | `src/main.rs:1269` | 🟢 |
| Saida filtrada com perda deve oferecer recuperacao; sem ela, a unit volta a saida bruta. | `src/main.rs:1307` e `core::runner::emit_guarded` | 🟢 |
| Integridade aplica whitelist em vez de regra generica; novos comandos esquecidos falham abertos para evitar falsa sensacao de protecao. | `src/main.rs:2668` | 🟢 |
| O proxy privilegia transparencia de output e limita somente a memoria reservada para tracking. | `src/main.rs:2449` | 🟢 |
| O dispatch contem adaptadores especificos de ferramenta porque a ordem e forma dos argumentos pode alterar a semantica externa. | `src/main.rs`, helpers de Git, pnpm e Kubernetes | 🟡 |

## Estado Interno

- 🟢 A unit nao possui armazenamento persistente proprio; ela delega persistencia ao tracking e a configuracoes/manifests de hooks.
- 🟢 `PROXY_CHILD_PID` e um `AtomicU32` estatico usado apenas durante `Commands::Proxy` em Unix para coordenar a entrega de sinais ao processo filho.
- 🟢 `TimedExecution` delimita a medida de tempo nos caminhos de fallback e proxy, sendo consumido para tracking no final do fluxo.
- 🟢 `Cli`, `Commands` e enums de subcomandos vivem em memoria durante o parse e dispatch.

## Observabilidade

- 🟢 Erros de alto nivel sao emitidos em stderr como `rtk: {:#}` por `main()`.
- 🟢 Falha de spawn no fallback emite `[rtk: <erro>]`, registra parse failure e retorna `127`.
- 🟢 `proxy` escreve stdout e stderr do filho em tempo real nos streams correspondentes e mostra o comando em stderr quando `verbose > 0`.
- 🟢 Fallback e proxy geram registros de tracking por `TimedExecution`; fallback tambem registra a falha de parse de forma silenciosa.
- 🟢 A telemetria e tentada no inicio, mas a evidencia analisada nao a classifica como log de usuario da CLI.

## Riscos e Lacunas

- 🔴 A extracao e estatica e nao validou todos os ramos de dispatch contra as ferramentas externas e versoes reais suportadas.
- 🔴 A whitelist de `is_operational_command()` permite que um novo wrapper esquecido pule a verificacao de integridade, comportamento assumido explicitamente pelo legado.
- 🔴 A equivalencia de sinais, quoting e process spawning em Windows nao foi comprovada no ambiente analisado.
- 🟡 O limite de 1 MiB no `proxy` preserva memoria de tracking, mas pode produzir metricas incompletas para processos de saida muito grande; a saida visivel permanece integral.
