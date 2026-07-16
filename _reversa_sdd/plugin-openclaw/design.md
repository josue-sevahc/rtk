# Plugin OpenClaw, Design Técnico

> Design reconstruído de `openclaw/`. 🟢 confirmado no código; 🟡 inferido; 🔴 lacuna.

## Interface

| Símbolo | Assinatura | Retorno | Observação |
|---------|------------|---------|------------|
| `checkRtk` | `() => boolean` | disponibilidade cacheada | Executa `which rtk` uma única vez por processo. 🟢 `openclaw/index.ts` |
| `tryRewrite` | `(command: string) => [string \| null, RewriteVerdict?]` | rewrite e verdict opcional | Executa `rtk rewrite` com timeout de 2 s. 🟢 `openclaw/index.ts` |
| `register` | `(api: any) => void` | nenhum | Lê configuração e registra o hook OpenClaw. 🟢 `openclaw/index.ts` |
| handler | `(event: { toolName; params }) => resultado?` | bloqueio, params alterados ou `undefined` | Registrado em `before_tool_call` com prioridade 10. 🟢 `openclaw/index.ts` |

### Configuração publicada

| Campo | Tipo | Padrão | Efeito |
|-------|------|--------|--------|
| `enabled` | boolean | `true` | Desabilita completamente o registro quando `false`. 🟢 `openclaw/openclaw.plugin.json` |
| `verbose` | boolean | `false` | Habilita logs de registro, decisão e aprovação. 🟢 `openclaw/openclaw.plugin.json` |

### Protocolo com `rtk rewrite`

| Exit code | Stdout | Resultado do plugin |
|-----------|--------|---------------------|
| `0` | rewrite diferente do original | Retorna parâmetros com `command` reescrito. 🟢 |
| `1` | irrelevante | Retorna `undefined` e mantém o comando. 🟢 |
| `2` | irrelevante | Retorna `{ block: true, blockReason }`. 🟢 |
| `3` | rewrite utilizável | Reescreve e inclui `requireApproval`. 🟢 |
| outro, erro ou timeout | irrelevante | Retorna `undefined`. 🟢 |

O CLI Rust é a origem desse protocolo: ele avalia deny antes de rewrite, trata construtos não atestáveis como passthrough e mapeia permissões não explicitamente allow para `ask`. 🟢 `src/hooks/rewrite_cmd.rs`

## Fluxo Principal

1. `register` obtém `api.config`, interpreta `enabled` e `verbose` com comparação estrita a `false` e `true`. 🟢 `openclaw/index.ts`
2. Se estiver desabilitado, retorna; caso contrário, `checkRtk` chama `which rtk` somente se o cache ainda for nulo. 🟢 `openclaw/index.ts`
3. Sem binário disponível, emite warning e encerra o registro; com binário, instala `api.on("before_tool_call", handler, { priority: 10 })`. 🟢 `openclaw/index.ts`
4. O handler ignora ferramentas diferentes de `exec` e comandos que não sejam strings. 🟢 `openclaw/index.ts`
5. `tryRewrite` chama sincronicamente `rtk rewrite <command>`, remove espaços do stdout e interpreta sucesso ou `status` da exceção. 🟢 `openclaw/index.ts`
6. Para `deny`, o handler bloqueia. Para ausência de rewrite, retorna sem resultado. Para rewrite, clona `event.params`, altera `command` e, se necessário, anexa solicitação de aprovação. 🟢 `openclaw/index.ts`

## Fluxos Alternativos

- **Plugin desabilitado:** nenhum hook é registrado. 🟢 `openclaw/index.ts`
- **`rtk` ausente:** o plugin fica inativo e informa `[rtk] rtk binary not found in PATH`. 🟢 `openclaw/index.ts`
- **Exit 0 sem alteração:** stdout vazio ou igual ao comando é tratado como passthrough. 🟢 `openclaw/index.ts`
- **Exit 3 sem stdout utilizável:** não solicita aprovação nem altera parâmetros. 🟢 `openclaw/index.ts`
- **Deny:** o comando não chega ao executor OpenClaw. 🟢 `openclaw/index.ts`
- **Aprovação negada ou expirada:** a configuração de aprovação define `timeoutBehavior: "deny"`. 🟢 `openclaw/index.ts`

## Dependências

- `node:child_process`: `execFileSync` para localizar e invocar o CLI. 🟢 `openclaw/index.ts`
- Binário `rtk` no `PATH`: implementa regra de rewrite e política de permissões. 🟢 `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`
- API de plugins OpenClaw: fornece `config` e `on`. 🟢 `openclaw/index.ts`
- `openclaw.plugin.json`: declara descoberta, configuração e hints de UI. 🟢 `openclaw/openclaw.plugin.json`

## Decisões de Design Identificadas

| Decisão | Evidência no código | Confiança |
|---------|---------------------|-----------|
| Concentrar a semântica de rewrite no Rust e manter o plugin como adaptador fino. | comentário inicial em `openclaw/index.ts` | 🟢 |
| Usar processo síncrono com limite de 2 s dentro do hook. | `execFileSync(..., { timeout: 2000 })` em `openclaw/index.ts` | 🟢 |
| Expressar estado por exit code em vez de um protocolo JSON entre plugin e CLI. | `tryRewrite` e `src/hooks/rewrite_cmd.rs` | 🟢 |
| Não oferecer `allow-always` para hooks de plugin. | `requireApproval.allowedDecisions` em `openclaw/index.ts` | 🟢 |
| Degradar falhas operacionais para passthrough. | bloco `catch` de `tryRewrite` | 🟢 |

## Estado Interno e Observabilidade

- `rtkAvailable` é `boolean | null`, compartilhado pelo módulo e preenchido após a primeira verificação. 🟢 `openclaw/index.ts`
- O plugin não persiste estado nem telemetria própria. 🟢 `openclaw/index.ts`, `openclaw/package.json`
- Com `verbose`, escreve decisões de deny, mapeamentos de comando, resultado de aprovação e registro do plugin. 🟢 `openclaw/index.ts`

## Riscos e Lacunas

- 🔴 Não há testes automatizados dentro de `openclaw/` para validar o contrato contra uma instância real de OpenClaw.
- 🔴 `api` usa `any` e o shape de `event` é manual; a compatibilidade da API e de `requireApproval` não foi executada estaticamente.
- 🟡 Erros operacionais do subprocesso e ausência de rewrite compartilham o mesmo fallback; diagnóstico detalhado dependeria de observabilidade adicional.
- 🟡 A execução síncrona pode afetar a latência da ferramenta em ambientes nos quais o CLI se aproxime do timeout, embora o limite seja explícito.

