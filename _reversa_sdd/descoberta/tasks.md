# Descoberta, Tarefas de Implementação

> Sequência para reconstruir `src/discover/` preservando os contratos observados.

## Pré-requisitos

- [ ] 🟢 Disponibilizar CLI, acesso a diretórios do usuário, JSONL, regex e serialização JSON.
- [ ] 🟢 Definir a política de permissões do host antes de consumir propostas de rewrite.

## Tarefas

- [ ] T-01, Modelar sessões, comandos extraídos, classificação, status RTK e relatório. Confiança: 🟢
  - Origem: `provider.rs`, `registry.rs`, `report.rs`.
  - Pronto quando: os tipos representam output opcional, erro, sequência, buckets e integrações sem parse textual do relatório.
- [ ] T-02, Implementar descoberta e leitura tolerante de JSONL Claude. Confiança: 🟢
  - Origem: `provider.rs`.
  - Pronto quando: projetos, data de modificação, linhas inválidas e associações tool-use/tool-result forem cobertos.
- [ ] T-03, Implementar tokenização e split shell-aware para operadores, pipes, redirects e shellisms. Confiança: 🟢
  - Origem: `lexer.rs`.
  - Pronto quando: offsets, aspas e limites de construções não atestáveis forem preservados.
- [ ] T-04, Implementar classificação, normalizações e catálogo de regras com estimativas. Confiança: 🟢
  - Origem: `registry.rs`, `rules.rs`.
  - Pronto quando: comandos ignorados, suportados, não suportados e wrappers conhecidos produzirem a categoria correta.
- [ ] T-05, Implementar agregação de oportunidades, bypass `RTK_DISABLED`, ordenação e formatos texto/JSON. Confiança: 🟢
  - Origem: `mod.rs`, `report.rs`.
  - Pronto quando: buckets e percentuais ponderados forem reproduzíveis.
- [ ] T-06, Integrar proposta de rewrite segura ao registry, mantendo o host como autoridade de autorização. Confiança: 🟢
  - Origem: `registry.rs`, `hooks/rewrite_cmd.rs`.
  - Pronto quando: ausência de regra ou construção incerta resulta em passthrough.

## Tarefas de Teste

- [ ] TT-01, Testar JSONL válido, inválido, sem resultado e sessões ausentes. Confiança: 🟢
- [ ] TT-02, Testar classificação, normalizações de prefixos e cálculo ponderado de economia. Confiança: 🟢
- [ ] TT-03, Testar lexer e rewrite para aspas, chains, pipes, redirects, heredoc, substituições e `gh --json`. Confiança: 🟢
- [ ] TT-04, Executar scan controlado de histórico grande para medir desempenho e taxas reais de erro. Confiança: 🔴

## Ordem Sugerida

1. T-01 a T-03 fixam o modelo de dados e a fronteira shell.
2. T-04 e T-06 constroem classificação e rewrite sob a política de segurança.
3. T-05 entrega o comando observável; TT-01 a TT-04 validam o contrato.
