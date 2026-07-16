# Protocolo de Hooks e Permissões

> Contrato extraído de `src/hooks/permissions.rs`, `rewrite_cmd.rs` e `hook_cmd.rs`. 🟢 confirmado; 🔴 lacuna.

## Visão Geral

Esta unit decide se um comando pode ser reescrito pelo RTK e traduz essa decisão para os protocolos de hook de cada host. Ela conserva a autoridade do host: sem regra explícita de allow, um rewrite exige confirmação. 🟢 `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`

## Responsabilidades

- Carregar regras de permissão de Claude, Cursor, Gemini e Droid. 🟢 `src/hooks/permissions.rs`
- Aplicar `Deny > Ask > Allow > Default`; cada segmento de comando composto precisa de allow independente. 🟢 `src/hooks/permissions.rs`
- Recusar auto-allow para substituições, heredocs e redirecionamentos não atestáveis. 🟢 `src/hooks/rewrite_cmd.rs`, `src/hooks/hook_cmd.rs`
- Expor exit codes 0/1/2/3 e payloads específicos para VS Code/Copilot, Gemini, Cursor e Droid. 🟢 `src/hooks/rewrite_cmd.rs`, `src/hooks/hook_cmd.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de Aceite |
|---|---|---|---|
| RF-01 | Avaliar deny antes de qualquer allow em todos os segmentos. 🟢 | Must | Segmento negado resulta em `Deny`, mesmo que outro segmento tenha allow. |
| RF-02 | Tratar ausência de regra como `Default` e convertê-la em confirmação, nunca auto-allow. 🟢 | Must | Comando regravável sem allow explícito retorna código 3 ou payload ask. |
| RF-03 | Usar protocolo específico sem emitir ruído em stdout. 🟢 | Must | Payload válido recebe JSON de resposta esperado; no-op preserva o comando do host. |
| RF-04 | Limitar stdin de hook a 1 MiB. 🟢 | Should | Entrada acima do limite é rejeitada sem processar o payload. |

## Critérios de Aceitação

```gherkin
Cenário: Default não é autorização
Dado um comando regravável sem regras allow
Quando rtk rewrite o avaliar
Então a saída deve conter o rewrite
E o código deve ser 3 para solicitar confirmação

Cenário: Deny precede allow em cadeia
Dado um comando composto com um segmento allow e outro deny
Quando o hook calcular a decisão
Então nenhum rewrite deve ser auto-autorizado
```

## Rastreabilidade de Código

| Arquivo | Cobertura |
|---|---|
| `src/hooks/permissions.rs` | hosts, regras e precedência 🟢 |
| `src/hooks/rewrite_cmd.rs` | bridge e protocolo de exit codes 🟢 |
| `src/hooks/hook_cmd.rs` | handlers JSON e limite de stdin 🟢 |
