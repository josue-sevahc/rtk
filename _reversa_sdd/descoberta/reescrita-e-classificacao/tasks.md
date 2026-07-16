# Reescrita e Classificação, Tarefas de Implementação

## Tarefas

- [ ] T-01, Implementar tokens shell e detecção de construções não atestáveis. Confiança: 🟢
  - Origem: `src/discover/lexer.rs`.
  - Pronto quando: aspas, escapes, pipes, operadores e redirects são distinguidos sem parser Bash total.
- [ ] T-02, Modelar `RtkRule`, status e catálogo ordenado de regras. Confiança: 🟢
  - Origem: `src/discover/rules.rs`.
  - Pronto quando: regras mais específicas podem sobrescrever categoria, percentual e status.
- [ ] T-03, Implementar normalizações antes da classificação. Confiança: 🟢
  - Origem: `src/discover/registry.rs`.
  - Pronto quando: env/sudo, paths absolutos e wrappers observados alcançam a mesma regra-base.
- [ ] T-04, Implementar rewrite simples, composto e por filtros customizados. Confiança: 🟢
  - Origem: `src/discover/registry.rs`.
  - Pronto quando: operadores são preservados e `None` representa nenhuma mudança segura.
- [ ] T-05, Conectar o resultado à ponte de hooks sem transportar decisão de permissão. Confiança: 🟢
  - Origem: `src/hooks/rewrite_cmd.rs`, ADR 003.
  - Pronto quando: o host continua responsável por Allow/Ask/Deny.

## Testes

- [ ] TT-01, Cobrir regras conflitantes, comandos ignorados, wrappers e percentuais por subcomando. Confiança: 🟢
- [ ] TT-02, Cobrir chains, pipes, redirects, heredoc, substituições, `RTK_DISABLED` e limite de prefixos. Confiança: 🟢
- [ ] TT-03, Executar matriz de regressão contra shells e plataformas suportadas. Confiança: 🔴
