# Plugin OpenClaw

> Contrato operacional reconstruido de `openclaw/`. Afirmações 🟢 foram confirmadas no código legado; 🟡 são inferências; 🔴 exigem validação humana.

## Visão Geral

O Plugin OpenClaw intercepta chamadas da ferramenta `exec` antes de sua execução e delega a decisão de reescrita ao comando local `rtk rewrite`. 🟢 `openclaw/index.ts`

Ele preserva a autoridade do OpenClaw: pode manter o comando original, bloqueá-lo por regra `deny`, substituir o comando automaticamente ou pedir aprovação explícita para uma substituição. 🟢 `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`

## Responsabilidades

- Registrar um hook `before_tool_call` de prioridade 10 quando o plugin estiver habilitado e o binário `rtk` estiver disponível. 🟢 `openclaw/index.ts`
- Processar somente chamadas `exec` com `params.command` do tipo string. 🟢 `openclaw/index.ts`
- Traduzir o protocolo de saída de `rtk rewrite` para bloqueio, passthrough, rewrite automático ou aprovação. 🟢 `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`
- Preservar os demais parâmetros da chamada ao substituir apenas `params.command`. 🟢 `openclaw/index.ts`
- Expor as opções booleanas `enabled` e `verbose` no manifesto do plugin. 🟢 `openclaw/openclaw.plugin.json`

## Regras de Negócio

- A configuração `enabled: false` impede o registro de qualquer hook. 🟢 `openclaw/index.ts`
- A ausência de `rtk` no `PATH` desabilita o plugin para o processo atual e registra um aviso. 🟢 `openclaw/index.ts`
- A verificação de disponibilidade do binário é cacheada em memória após a primeira tentativa. 🟢 `openclaw/index.ts`
- Somente exit code `0` com stdout diferente do comando original autoriza aplicação automática. 🟢 `openclaw/index.ts`
- Exit code `2` bloqueia a chamada com a razão `RTK deny rule matched`. 🟢 `openclaw/index.ts`
- Exit code `3` somente produz rewrite quando o stdout é utilizável; a aprovação aceita `allow-once` ou `deny` e expira como `deny`. 🟢 `openclaw/index.ts`
- Exit code `1`, código desconhecido, timeout ou erro sem stdout útil resultam em passthrough silencioso. 🟢 `openclaw/index.ts`
- As regras de classificação e permissão não pertencem ao TypeScript; são definidas pelo CLI Rust. 🟢 `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de Aceite |
|----|-----------|-----------|-------------------|
| RF-01 | Registrar o interceptor somente quando habilitado e com `rtk` localizável. 🟢 | Must | Dado `enabled: false` ou binário ausente, quando `register` executar, então nenhum handler é registrado. |
| RF-02 | Ignorar chamadas que não sejam `exec` ou cujo comando não seja string. 🟢 | Must | Dado evento inelegível, quando o hook executar, então ele retorna sem alterar parâmetros. |
| RF-03 | Chamar `rtk rewrite <command>` com timeout de 2 s. 🟢 | Must | Dado `exec` elegível, quando o hook executar, então a invocação usa `execFileSync` e o comando original como argumento único. |
| RF-04 | Aplicar rewrite permitido preservando campos não relacionados da chamada. 🟢 | Must | Dado exit `0` e rewrite distinto, quando o hook responder, então somente `params.command` é substituído. |
| RF-05 | Bloquear regras negadas e solicitar confirmação para rewrites classificados como `ask`. 🟢 | Must | Dado exit `2` ou `3`, quando o hook responder, então produz respectivamente bloqueio ou `requireApproval` com timeout `deny`. |
| RF-06 | Registrar decisões e o registro do plugin quando `verbose` estiver ativo. 🟢 | Should | Dada configuração verbose, quando houver decisão ou registro, então uma mensagem de diagnóstico é escrita no console. |

## Requisitos Não Funcionais

| Tipo | Requisito inferido | Evidência no código | Confiança |
|------|--------------------|---------------------|-----------|
| Segurança | A decisão de autoaprovação permanece no CLI e rewrites `ask` não ganham persistência automática de consentimento. | `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs` | 🟢 |
| Confiabilidade | A chamada síncrona ao CLI possui timeout de 2 s e falhas degradam para o comando original. | `openclaw/index.ts` | 🟢 |
| Compatibilidade | O plugin usa o contrato observado de `before_tool_call` e shapes manuais, sem dependência tipada da API OpenClaw. | `openclaw/index.ts` | 🟢 |
| Observabilidade | Logs de diagnóstico são opt-in via `verbose`; ausência do binário sempre emite warning. | `openclaw/index.ts`, `openclaw/openclaw.plugin.json` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: Aplicar rewrite permitido
Dado um plugin habilitado com rtk disponível
E uma chamada exec cujo comando possui rewrite permitido
Quando rtk rewrite retornar código 0 e um comando diferente
Então o hook deve retornar os parâmetros originais com command substituído
E não deve solicitar aprovação

Cenário: Pedir aprovação para rewrite não autorizado
Dado uma chamada exec elegível
Quando rtk rewrite retornar código 3 com stdout utilizável
Então o hook deve devolver o comando reescrito
E deve anexar requireApproval com allow-once ou deny
E o timeout deve negar a operação

Cenário: Preservar chamada sem rewrite
Dado uma chamada exec elegível
Quando rtk rewrite falhar, expirar ou não produzir rewrite utilizável
Então o hook não deve alterar a chamada
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Protocolo de exit code e bloqueio | Must | Mantém a fronteira de segurança entre RTK e OpenClaw. 🟢 |
| Registro condicionado e filtragem de eventos | Must | Evita interferir em ferramentas ou instalações não elegíveis. 🟢 |
| Aprovação para `ask` | Must | Preserva a decisão humana para comandos sem allow explícito. 🟢 |
| Logs verbosos | Should | Facilitam diagnóstico sem alterar o caminho funcional. 🟢 |

## Rastreabilidade de Código

| Arquivo | Função / Classe | Cobertura |
|---------|-----------------|-----------|
| `openclaw/index.ts` | `checkRtk`, `tryRewrite`, `register`, handler `before_tool_call` | 🟢 |
| `openclaw/openclaw.plugin.json` | manifesto e schema de configuração | 🟢 |
| `openclaw/package.json` | metadados e arquivos publicados | 🟢 |
| `openclaw/README.md` | instalação e uso documentados | 🟢 |
| `src/hooks/rewrite_cmd.rs` | contrato fonte dos exit codes 0, 1, 2 e 3 | 🟢 |

