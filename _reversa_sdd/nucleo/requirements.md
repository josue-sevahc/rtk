# Nucleo

> Contrato operacional do modulo `src/core/` do RTK. Afirmacoes marcadas com 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 indicam lacunas.

## Visao Geral

O Nucleo fornece infraestrutura compartilhada e agnostica de dominio para o RTK: execucao de processos, captura e streaming de I/O, protecao de saida, filtros TOML, configuracao, tracking local, recuperacao por tee e telemetria. 🟢 `src/core/README.md:5`, `src/core/mod.rs:1`

Ele deve ser consumido pelos modulos de comando sem conhecer comandos especificos, agentes ou regras de hook. 🟢 `src/core/README.md:5`

## Responsabilidades

- Executar processos filhos em modos filtrado, filtrado sensivel a exit code, streaming ou passthrough, preservando o exit code. 🟢 `src/core/runner.rs:84`, `src/core/runner.rs:159`
- Capturar stdout e stderr concorrentemente, limitar cada captura a 10 MiB e evitar processos zumbis. 🟢 `src/core/stream.rs:244`, `src/core/stream.rs:247`, `src/core/stream.rs:284`
- Nunca emitir uma saida filtrada que estime mais tokens que a saida bruta. 🟢 `src/core/guard.rs:6`
- Carregar e aplicar filtros TOML confiaveis, preservando os filtros built-in quando configuracoes externas forem invalidas ou nao confiaveis. 🟢 `src/core/toml_filter.rs:185`, `src/core/toml_filter.rs:203`
- Registrar execucoes e economia de tokens em SQLite, com agregacoes e escopo opcional por projeto. 🟢 `src/core/tracking.rs:1`, `src/core/tracking.rs:54`
- Persistir saida bruta recuperavel em arquivos tee sob as condicoes configuradas. 🟢 `src/core/tee.rs:76`, `src/core/tee.rs:151`
- Enviar telemetria apenas sob consentimento e sem bloquear a execucao do CLI. 🟢 `src/core/telemetry.rs:22`

## Regras de Negocio

- O Nucleo deve permanecer agnostico de comandos, hooks e agentes. 🟢 `src/core/README.md:5`
- Se a saida filtrada tiver mais tokens estimados que a bruta, a saida bruta deve ser retornada; em empate, a filtrada pode ser mantida. 🟢 `src/core/guard.rs:6`
- O exit code do processo filho deve ser devolvido em todos os modos de execucao; em Unix, termino por sinal vira `128 + sinal`. 🟢 `src/core/runner.rs:156`, `src/core/stream.rs:230`
- Com `skip_filter_on_failure`, uma falha reemite stdout e stderr brutos e e rastreada como raw contra raw. 🟢 `src/core/runner.rs:114`
- Filtros TOML locais so entram no registry quando o fluxo de confianca os classifica como `Trusted` ou `EnvOverride`; filtros invalidos apenas geram aviso. 🟢 `src/core/toml_filter.rs:203`
- A DSL aplica estagios em ordem fixa: ANSI, replace, match_output/unless, selecao de linhas, truncamento, head/tail, max_lines e on_empty. 🟢 `src/core/toml_filter.rs:515`
- Tee respeita a desativacao por `RTK_TEE=0`, habilitacao, modo, exit code e tamanho minimo antes de gravar a saida. 🟢 `src/core/tee.rs:76`, `src/core/tee.rs:151`
- Tracking por projeto usa igualdade ou `GLOB` com separador de caminho, nao `LIKE`. 🟢 `src/core/tracking.rs:51`
- Telemetria exige endpoint compilado, ausencia de opt-out, configuracao habilitada, consentimento explicito e intervalo minimo entre pings. 🟢 `src/core/telemetry.rs:22`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|------------|-------------------|
| RF-01 | Expor um executor compartilhado com os modos `Filtered`, `FilteredWithExit`, `Streamed` e `Passthrough`. 🟢 | Must | Dado um processo em cada modo, quando for executado, entao o modo selecionado determina captura, filtro e encaminhamento de stdio. |
| RF-02 | Preservar o exit code do processo executado, inclusive em falhas filtradas ou em passthrough. 🟢 | Must | Dado um processo que termina com codigo nao zero, quando o runner concluir, entao o mesmo codigo e retornado. |
| RF-03 | Proteger a saida entregue para que nao exceda a estimativa de tokens da saida bruta. 🟢 | Must | Dado um filtro que aumenta a estimativa de tokens, quando a saida for emitida, entao a saida bruta e usada. |
| RF-04 | Carregar filtros TOML built-in e externos confiaveis, compilar regexes e aplicar a DSL deterministica. 🟢 | Must | Dado um filtro confiavel valido, quando seu comando casar, entao suas transformacoes sao aplicadas na ordem documentada. |
| RF-05 | Ignorar filtros TOML externos nao confiaveis, alterados ou invalidos sem interromper o comando. 🟢 | Must | Dado um filtro externo nao confiavel ou invalido, quando o registry carregar, entao ele nao e ativado e o processo permanece executavel. |
| RF-06 | Registrar entrada, saida, duracao e economia estimada de uma execucao em SQLite, sem tornar falhas de tracking fatais para o fluxo de comando. 🟢 | Should | Dado um comando executado, quando o tracking estiver disponivel, entao um registro com tokens e duracao pode ser consultado; indisponibilidade do banco nao altera o exit code do comando. |
| RF-07 | Gravar uma recuperacao da saida bruta e mostrar hint quando a politica de tee permitir. 🟢 | Should | Dado output elegivel e tee habilitado, quando a compactacao for exibida, entao um arquivo de recuperacao e seu hint podem ser produzidos. |
| RF-08 | Fazer ping de telemetria anonima e opt-in em segundo plano. 🟢 | Could | Dado consentimento e elegibilidade diaria, quando `maybe_ping` rodar, entao o envio nao bloqueia o CLI. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Performance | Captura bruta por stream e limitada a 10 MiB para conter uso de memoria. | `src/core/stream.rs:244` | 🟢 |
| Confiabilidade | Um guard RAII aguarda processos filhos ao sair do escopo, reduzindo processos zumbis. | `src/core/stream.rs:284` | 🟢 |
| Confiabilidade | Erro de escrita por `BrokenPipe` na emissao filtrada e tolerado. | `src/core/stream.rs` | 🟢 |
| Privacidade | Telemetria depende de consentimento, opt-out e execucao assincrona. | `src/core/telemetry.rs:22` | 🟢 |
| Persistencia | O historico local usa SQLite com retencao automatica configurada. | `src/core/tracking.rs:9`, `src/core/constants.rs` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Retornar saida bruta quando o filtro piora a estimativa
Dado uma saida bruta e uma saida filtrada com mais tokens estimados
Quando o runner emitir o resultado filtrado
Entao a saida bruta deve ser exibida
E o tracking deve registrar a mesma saida exibida

Cenario: Preservar falha sem filtrar
Dado um processo que termina com exit code diferente de zero
E a opcao skip_filter_on_failure esta ativa
Quando o runner executar o processo
Entao stdout e stderr brutos devem ser reemitidos
E o exit code original deve ser retornado

Cenario: Recusar filtro local nao confiavel
Dado um arquivo TOML externo sem confianca valida
Quando o registry de filtros carregar
Entao o arquivo nao deve adicionar filtros ativos
E a execucao do comando nao deve falhar por essa recusa
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Execucao compartilhada e preservacao de exit code | Must | Caminho central usado pelos wrappers de comando. 🟢 |
| Guard never-worse | Must | Invariante de produto que protege a promessa de economia de tokens. 🟢 |
| Filtros TOML trust-gated | Must | Caminho de fallback configuravel e limite de seguranca. 🟢 |
| Tracking SQLite | Should | Suporta analytics e ganho, mas a execucao deve sobreviver sem ele. 🟢 |
| Recuperacao via tee | Should | Preserva detalhes quando a compactacao omite conteudo. 🟢 |
| Telemetria opt-in | Could | Observabilidade de produto, nao necessaria para executar comandos. 🟢 |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/core/README.md` | Contrato de fronteira do modulo | 🟢 |
| `src/core/runner.rs` | `run`, `run_captured_filter`, `emit_guarded` | 🟢 |
| `src/core/stream.rs` | `run_streaming`, `status_to_exit_code`, `ChildGuard` | 🟢 |
| `src/core/guard.rs` | `never_worse` | 🟢 |
| `src/core/toml_filter.rs` | `TomlFilterRegistry`, `apply_filter_with_info` | 🟢 |
| `src/core/tracking.rs` | `Tracker`, `TimedExecution` | 🟢 |
| `src/core/tee.rs` | `tee_raw`, `tee_and_hint` | 🟢 |
| `src/core/telemetry.rs` | `maybe_ping` | 🟢 |
