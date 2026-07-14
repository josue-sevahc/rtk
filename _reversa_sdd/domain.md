# Dominio e Regras de Negocio - RTK

Projeto: `rtk`  
Fase: Interpretacao (Detetive)  
Gerado em: `2026-07-14T22:10:33Z`

## Escala de confianca

- 🟢 **CONFIRMADO** - extraido diretamente do codigo ou de um commit associado.
- 🟡 **INFERIDO** - interpretacao sustentada por mais de uma evidencia.
- 🔴 **LACUNA** - depende de validacao humana ou execucao.

## Modelo de dominio

| Conceito | Responsabilidade observada | Evidencia |
|---|---|---|
| Comando externo | Operacao original solicitada ao shell ou a uma ferramenta. Pode ser reescrita, filtrada ou executada em passthrough. | 🟢 `src/main.rs`, `src/core/runner.rs` |
| Filtro | Transformacao de output para reduzir tokens; pode ser Rust especializado ou TOML declarativo. | 🟢 `src/cmds/`, `src/core/toml_filter.rs` |
| Saida bruta e filtrada | A primeira e a referencia de fidelidade; a segunda e a forma entregue ao agente e registrada no tracking. | 🟢 `src/core/guard.rs`, `src/core/runner.rs` |
| Regra de permissao do host | Regra allow, ask ou deny do agente/editor que recebe o comando. | 🟢 `src/hooks/permissions.rs` |
| Decisao de hook | Resultado da combinacao entre permissao e rewrite: permitir rewrite, pedir confirmacao, deferir ou preservar bloqueio nativo. | 🟢 `src/hooks/hook_cmd.rs` |
| Filtro customizado | Arquivo TOML de projeto ou global que pode alterar a visibilidade de output e precisa de confianca explicita. | 🟢 `src/hooks/trust.rs` |
| Registro de confianca | Hash SHA-256 associado ao caminho canonico de um filtro e ao momento de aprovacao. | 🟢 `src/hooks/trust.rs` |
| Integridade de hook | Comparacao entre hash de instalacao e hash atual do script de hook legado. | 🟢 `src/hooks/integrity.rs` |
| Telemetria | Ping diario anonimo, condicionado por build, configuracao, consentimento e opt-out. | 🟢 `src/core/telemetry.rs` |
| Tracking | Historico local de execucoes e estimativas de economia de tokens em SQLite. | 🟢 `src/core/tracking.rs` |

## Regras operacionais

1. 🟢 **O RTK nao deve emitir mais tokens que o comando original.** O guard compara as estimativas de tokens e devolve a saida bruta quando a forma filtrada, incluindo hint de recuperacao, for maior. Saida impressa e saida rastreada devem coincidir. Evidencia: `src/core/guard.rs`, `src/core/runner.rs`; commit `861a46d`.
2. 🟢 **Exit codes reais devem ser preservados.** A compactacao nao deve transformar falha em sucesso; a decisao foi reforcada, por exemplo, para `git stash show`. Evidencia: `src/core/runner.rs`, `src/cmds/`; commit `861a46d`.
3. 🟢 **Meta-comandos RTK falham fechados no parse.** Um uso invalido de comando proprio nao pode cair no fallback de execucao externa. Evidencia: `src/main.rs`; commit `9cc4937`.
4. 🟢 **Filtro TOML customizado so e carregado quando confiavel.** A confianca e por caminho canonico e SHA-256; qualquer alteracao de conteudo revoga a carga ate nova aprovacao. Evidencia: `src/hooks/trust.rs`.
5. 🟢 **O override de confianca de filtros e limitado a CI.** `RTK_TRUST_PROJECT_FILTERS=1` so tem efeito em ambiente CI detectado; fora dele o override e ignorado. Evidencia: `src/hooks/trust.rs`.
6. 🟢 **Falhas ao avaliar confianca nao habilitam filtros.** Arquivo ausente, store ilegivel, conteudo nao UTF-8 ou erro de avaliacao terminam como nao confiavel. Evidencia: `src/hooks/trust.rs`.
7. 🟢 **Telemetria e opt-in.** Sem endpoint compilado, consentimento explicito, configuracao habilitada e ausencia de `RTK_TELEMETRY_DISABLED=1`, nenhum ping e enviado. Evidencia: `src/core/telemetry.rs`; commit `6a5bc84`.
8. 🟢 **Telemetria nao bloqueia o CLI.** O marcador e atualizado antes do envio e o envio ocorre em thread; erros sao ignorados. Evidencia: `src/core/telemetry.rs`.
9. 🟢 **Permissoes do host prevalecem sobre o rewrite.** A precedencia e `Deny > Ask > Allow > Default`; default equivale a pedir confirmacao. Evidencia: `src/hooks/permissions.rs`.
10. 🟢 **Comando composto so recebe allow se todos os segmentos forem permitidos.** `&&`, `||`, `|` e `;` nao podem elevar uma cadeia por um unico segmento permitido. Evidencia: `src/hooks/permissions.rs`; commit `40c9dbc`.
11. 🟢 **Construtos nao atestaveis nao sao autoaprovados.** Substituicao de comandos e redirecionamentos para arquivo impedem allow automatico; o hook defere ou pede confirmacao. Evidencia: `src/hooks/permissions.rs`, `src/hooks/hook_cmd.rs`.
12. 🟢 **Integridade de hook adulterada bloqueia comandos operacionais.** Com hash divergente, o runtime encerra com erro; nao existe bypass por variavel de ambiente. Hooks sem baseline e hashes orfaos geram tratamento tolerante para compatibilidade. Evidencia: `src/hooks/integrity.rs`.
13. 🟢 **A integracao Droid e deliberadamente deny-only.** Regras explicitas de deny/block de todos os escopos fazem RTK deixar o comando original intacto para a politica nativa agir; RTK nao espelha allowlists nem emite permissao allow para um comando renomeado. Evidencia: `src/hooks/permissions.rs`, `src/hooks/hook_cmd.rs`; commit `3d40742`.

## Invariantes transversais

- 🟢 O rewrite precisa ser semanticamente transparente: se nao puder ser provado ou se nao houver rewrite conhecido, o comando original segue sem intervencao.
- 🟢 A coleta local de tracking e distinta da telemetria remota; a segunda exige consentimento adicional.
- 🟡 A prioridade do produto parece ser reduzir tokens sem alterar a decisao de seguranca do agente hospedeiro, combinando o guard de fidelidade, a preservacao de exit code e o defer de permissoes. Esta leitura e apoiada por codigo e historico, mas nao ha declaracao unica de produto executavel.

## Lacunas

- 🔴 A analise e estatica: nao validou a equivalencia de cada filtro contra as versoes reais das ferramentas externas.
- 🔴 A retencao efetiva e as permissoes de arquivo do banco local de tracking nao foram exercitadas neste ambiente.
- 🔴 O comportamento final de confirmacao depende das versoes e configuracoes dos hosts integrados (Claude, Cursor, Gemini, Droid, Copilot e OpenClaw).
