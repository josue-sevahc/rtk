# Matriz de Permissoes - RTK

Projeto: `rtk`  
Fase: Interpretacao (Detetive)

RTK nao possui RBAC de usuarios. O controle relevante e uma camada ACL de comandos delegada aos agentes hospedeiros e complementada por controles locais de confianca e integridade.

## Ordem de decisao

| Prioridade | Resultado | Efeito RTK |
|---:|---|---|
| 1 | `Deny` | Nao reescreve; preserva a politica nativa do host. |
| 2 | `Ask` | Reescreve quando conhecido, mantendo confirmacao. |
| 3 | `Allow` | Reescreve com autoallow apenas quando o adaptador pode preservar a semantica do host. |
| 4 | `Default` | Trata como ask; nao eleva privilegio. |

🟢 Evidencia: `src/hooks/permissions.rs`, `src/hooks/hook_cmd.rs`.

## Matriz por superficie

| Superficie | Fonte de regras | Allow automatico | Deny/Block | Observacoes |
|---|---|---|---|---|
| Claude | Regras de permissao do Claude | Sim, somente com match explicito e atestavel | RTK nao reescreve | Default e ask; cadeias exigem allow em todos os segmentos. |
| Cursor | `~/.cursor/cli-config.json` global | Sim, apenas subconjunto global confiavel | Prevalece | O projeto continua sob controle do host; RTK evita ser mais permissivo. |
| Gemini | Projeto quando workspace confiavel; senao global | Sim, sob as mesmas restricoes | Prevalece | Folder trust influencia se a configuracao do projeto pode ser lida. |
| Droid | `settings.json` e `settings.local.json` de escopos usuario/projeto | Nao | RTK deixa o comando original para o Droid decidir | As listas de deny/block sao unidas; allowlists nao sao espelhadas. |
| Copilot / VS Code | Caminho Claude no adaptador observado | Condicional ao veredito e formato | Nao reescreve no deny | Preserva metadados de `toolArgs` no modo Copilot CLI. |
| OpenClaw | `rtk rewrite` retorna allow, ask ou deny | Allow e ask delegados ao mecanismo do plugin | Plugin bloqueia chamada em deny | Timeout do pedido de permissao resulta em deny; nao ha `allow-always`. |
| Filtros TOML customizados | Store local de SHA-256 | Carregados so em estado confiavel | Nao confiaveis sao ignorados | Override limitado a CI via `RTK_TRUST_PROJECT_FILTERS=1`. |
| Hook legado Claude | SHA-256 sidecar | Execucao permitida quando verificado/compatibilidade | Alteracao detectada bloqueia operacionais | Sem bypass por variavel de ambiente. |
| Telemetria | Consentimento e configuracao local | Envio somente opt-in | Opt-out por config ou `RTK_TELEMETRY_DISABLED=1` | Nao e um controle de execucao de comandos. |

## Regras de seguranca da ACL

- 🟢 `Deny > Ask > Allow > Default`.
- 🟢 Um comando com substituicao ou redirecionamento para arquivo nao recebe allow automatico.
- 🟢 Em comandos compostos, um segmento sem allow reduz a cadeia a ask/default; um deny em qualquer segmento vence.
- 🟢 A resposta vazia de RTK para deny no Droid e intencional: evita que renomear o programa contorne padroes de bloqueio do proprio Droid.
- 🟡 A camada RTK funciona como preservacao de politicas de host, nao como autoridade de permissao independente.

## Lacunas

- 🔴 Nao foram exercitados arquivos de configuracao reais de cada host nem suas versoes atuais.
- 🔴 O contrato de permissao do OpenClaw foi inspecionado no plugin, mas nao em uma instancia OpenClaw ativa.
