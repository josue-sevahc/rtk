# Análise de Histórico, Tarefas de Implementação

## Tarefas

- [ ] T-01, Implementar resolução de diretório Claude, codificação de projeto e busca de JSONL sem symlinks. Confiança: 🟢
  - Origem: `src/discover/provider.rs`.
  - Pronto quando: ausência de diretório e filtros de escopo são distinguíveis.
- [ ] T-02, Implementar parser de uma passagem para usos Bash e resultados por id. Confiança: 🟢
  - Origem: `src/discover/provider.rs`.
  - Pronto quando: resultados opcionais, preview limitado e sequência são preservados.
- [ ] T-03, Integrar extração ao agregador resiliente por sessão. Confiança: 🟢
  - Origem: `src/discover/mod.rs`.
  - Pronto quando: erro em arquivo incrementa diagnóstico e o lote continua.
- [ ] T-04, Implementar relatório texto/JSON, integrações e avisos de bypass. Confiança: 🟢
  - Origem: `src/discover/report.rs`, `src/discover/mod.rs`.
  - Pronto quando: campos e ordenação são idênticos entre modelo e formatos.

## Testes

- [ ] TT-01, Cobrir codificação de caminhos, filtros temporais, symlink e diretórios ausentes. Confiança: 🟢
- [ ] TT-02, Cobrir linhas inválidas, tool use sem resultado, resultado de erro e múltiplas sessões. Confiança: 🟢
- [ ] TT-03, Testar texto e JSON para relatório vazio, oportunidades, bypass e integrações detectadas. Confiança: 🟢
- [ ] TT-04, Medir memória e tempo com corpus representativo de sessões grandes. Confiança: 🔴
