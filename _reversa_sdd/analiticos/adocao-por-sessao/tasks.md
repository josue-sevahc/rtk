# Adoção por Sessão, Tarefas de Implementação

## Pré-requisitos

- [ ] Implementar provider para descobrir/extrair JSONL de sessões Claude. 🟢
- [ ] Disponibilizar parser de cadeia e registry de comandos suportados. 🟢

## Tarefas

- [ ] T-01 — Descobrir, filtrar, ordenar e limitar sessões candidatas.
  - Origem no legado: `src/analytics/session_cmd.rs`
  - Critério de pronto: subagents ficam fora e apenas as 10 mais recentes são processadas.
  - Confiança: 🟢
- [ ] T-02 — Implementar classificação por partes de cadeia.
  - Origem no legado: `src/analytics/session_cmd.rs`, `src/discover/registry.rs`
  - Critério de pronto: `rtk`, comandos suportados e não suportados produzem contagem correta.
  - Confiança: 🟢
- [ ] T-03 — Renderizar o resumo por sessão e média global.
  - Origem no legado: `src/analytics/session_cmd.rs`
  - Critério de pronto: ausência de sessões/comandos recebe mensagem legível.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01 — Testar barra de 0%, 50% e 100%.
- [ ] TT-02 — Testar explícitos, reescritos, mistos e cadeias encadeadas.
- [ ] TT-03 — Testar JSONL de sessão com tool_use/tool_result.

## Ordem Sugerida

1. Provider e fixtures; 2. classificação; 3. tabela e agregação.

## Lacunas Pendentes (🔴)

Nenhuma.
