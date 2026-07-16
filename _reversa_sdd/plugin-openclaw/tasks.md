# Plugin OpenClaw, Tarefas de Implementação

> Sequência para reconstruir `openclaw/` preservando o contrato do adaptador e a autoridade de permissão do CLI Rust.

## Pré-requisitos

- [ ] Disponibilizar Node.js e uma API de plugin compatível com o contrato `before_tool_call`. 🟡 Origem: `openclaw/index.ts`.
- [ ] Disponibilizar o binário `rtk` no `PATH`, com o subcomando `rewrite` e seu protocolo de exit codes. 🟢 Origem: `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`.
- [ ] Implementar ou integrar a política de permissões e o registry de rewrite no lado do CLI. 🟢 Origem: `src/hooks/rewrite_cmd.rs`.

## Tarefas

- [ ] T-01, Publicar o manifesto npm e o manifesto OpenClaw com identidade, arquivos distribuídos e schema de `enabled`/`verbose`.
  - Origem no legado: `openclaw/package.json`, `openclaw/openclaw.plugin.json`.
  - Critério de pronto: o host reconhece `rtk-rewrite`, expõe as duas opções booleanas e o pacote inclui entrada, manifesto e README.
  - Confiança: 🟢

- [ ] T-02, Implementar a verificação cacheada do binário usando `which rtk` e desabilitar o registro quando ela falhar.
  - Origem no legado: `openclaw/index.ts:15`.
  - Critério de pronto: duas chamadas de registro no mesmo processo não repetem a consulta ao `PATH`; ausência do binário produz warning e não instala handler.
  - Confiança: 🟢

- [ ] T-03, Implementar o adaptador síncrono de `rtk rewrite`, passando o comando inteiro como um argumento e limitando a execução a 2 s.
  - Origem no legado: `openclaw/index.ts:33`.
  - Critério de pronto: sucesso retorna stdout normalizado; timeout, exit 1 e falhas inesperadas não lançam erro para o host.
  - Confiança: 🟢

- [ ] T-04, Mapear os exits 0, 1, 2 e 3 para, respectivamente, rewrite automático, passthrough, deny e rewrite sujeito a aprovação.
  - Origem no legado: `openclaw/index.ts:40`, `src/hooks/rewrite_cmd.rs:11`.
  - Critério de pronto: cada código produz a tupla ou efeito previsto, e stdout vazio/igual ao comando nunca gera rewrite.
  - Confiança: 🟢

- [ ] T-05, Registrar `before_tool_call` com prioridade 10 e filtrar eventos para `toolName === "exec"` e comando string.
  - Origem no legado: `openclaw/index.ts:81`.
  - Critério de pronto: eventos de outras ferramentas e parâmetros inválidos retornam `undefined` sem alteração.
  - Confiança: 🟢

- [ ] T-06, Aplicar rewrites retornando cópia de `params` com apenas `command` modificado; bloquear denies com razão estável.
  - Origem no legado: `openclaw/index.ts:94`.
  - Critério de pronto: campos adicionais em `params` sobrevivem ao rewrite e exit 2 retorna `block: true` com `RTK deny rule matched`.
  - Confiança: 🟢

- [ ] T-07, Construir `requireApproval` para rewrites `ask`, limitado a `allow-once` e `deny`, com expiração que nega.
  - Origem no legado: `openclaw/index.ts:130`.
  - Critério de pronto: exit 3 com stdout válido adiciona título, descrição contendo original e rewrite, severidade `info` e `timeoutBehavior: "deny"`.
  - Confiança: 🟢

- [ ] T-08, Adicionar logs opt-in para registro, decisões e resolução de aprovação.
  - Origem no legado: `openclaw/index.ts:90`.
  - Critério de pronto: `verbose: false` não emite logs de decisão; `verbose: true` torna cada transição relevante rastreável no console.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar habilitação, ausência de binário e cache de disponibilidade.
  - Origem no legado: `openclaw/index.ts`.
  - Critério de pronto: o handler só é registrado na combinação habilitada/com binário; a consulta ocorre no máximo uma vez por processo.
  - Confiança: 🔴 Não há suite de testes legada em `openclaw/`.

- [ ] TT-02, Testar o mapeamento completo de exit code, incluindo stdout vazio, stdout idêntico, erro desconhecido e timeout.
  - Origem no legado: `openclaw/index.ts`, `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: exits 0/1/2/3 e falhas produzem exatamente os efeitos documentados, sem exceções não tratadas.
  - Confiança: 🟢

- [ ] TT-03, Testar eventos inelegíveis, preservação de parâmetros e bloqueio de deny.
  - Origem no legado: `openclaw/index.ts`.
  - Critério de pronto: somente `exec` com comando string é alterado; deny impede a execução.
  - Confiança: 🟢

- [ ] TT-04, Testar a solicitação de aprovação contra uma instância ou mock fiel da API OpenClaw.
  - Origem no legado: `openclaw/index.ts`.
  - Critério de pronto: `allow-once`, `deny` e timeout são aceitos pelo host conforme o contrato e não há `allow-always` implícito.
  - Confiança: 🔴 A compatibilidade real não foi exercitada no legado analisado.

## Ordem Sugerida

1. Implementar T-01 a T-04 para estabilizar publicação, descoberta de binário e protocolo com o CLI.
2. Implementar T-05 e T-06 para ligar o protocolo ao ciclo de ferramentas do OpenClaw.
3. Implementar T-07 e T-08, então executar TT-01 a TT-04 em mock e host real.

## Lacunas Pendentes (🔴)

- 🔴 Validar versões suportadas da API OpenClaw, sobretudo o contrato de `api.on` e `requireApproval`.
- 🔴 Criar cobertura automatizada no subdiretório `openclaw/`; o legado não contém testes locais para esse adaptador.
- 🔴 Confirmar que degradar erro de subprocesso para passthrough é a política operacional desejada em produção.

