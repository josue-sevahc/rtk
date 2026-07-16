# Protocolo de Hooks e Permissões, Tarefas de Implementação

## Tarefas

- [ ] T-01, Implementar carregadores de regras por host e extrair somente escopos de shell pertinentes.
  - Origem no legado: `src/hooks/permissions.rs`.
  - Critério de pronto: regras de projeto/global obedecem ao escopo e fonte do host.
  - Confiança: 🟢
- [ ] T-02, Implementar segmentação de cadeia e precedência `Deny > Ask > Allow > Default`.
  - Origem no legado: `src/hooks/permissions.rs`.
  - Critério de pronto: allow exige correspondência de cada segmento não vazio; Default não autoriza automaticamente.
  - Confiança: 🟢
- [ ] T-03, Implementar bridge `rtk rewrite` e protocolo de exit codes.
  - Origem no legado: `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: 0, 1, 2 e 3 representam allow, passthrough, deny e ask sem ambiguidade.
  - Confiança: 🟢
- [ ] T-04, Implementar handlers de payload e respostas JSON sem poluir stdout.
  - Origem no legado: `src/hooks/hook_cmd.rs`.
  - Critério de pronto: handlers preservam metadados necessários e fazem defer seguro em input inválido/no-op.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar precedência, cadeia parcialmente autorizada, Default e construtos não atestáveis.
  - Origem no legado: `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: nenhum caso sem allow explícito retorna auto-allow.
  - Confiança: 🟢
- [ ] TT-02, Testar payloads de todos os hosts, JSON inválido e limite de 1 MiB.
  - Origem no legado: `src/hooks/hook_cmd.rs`.
  - Critério de pronto: resposta sempre respeita o protocolo ou preserva passthrough.
  - Confiança: 🟢

## Lacunas Pendentes (🔴)

- 🔴 Validar contratos contra versões reais dos hosts suportados.
