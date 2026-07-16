# Aprendizado, Tarefas de Implementação

> Sequência para reconstruir `src/learn/` preservando os contratos observados.

## Pré-requisitos

- [ ] 🟢 Disponibilizar leitura de sessões Claude e extração de comandos com saída.
- [ ] 🟢 Disponibilizar regex, serialização JSON e acesso a filesystem para a saída opcional.

## Tarefas

- [ ] T-01, Modelar comandos, pares de correção, regras e tipos de erro. Confiança: 🟢
  - Origem no legado: `src/learn/detector.rs`.
  - Critério de pronto: tipos preservam comando, saída, confiança, ocorrência, base e exemplo.
- [ ] T-02, Implementar resolução de escopo, descoberta de sessões e tolerância a sessões malformadas. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`, `src/discover/provider.rs`.
  - Critério de pronto: todos os modos de projeto e `--since` são encaminhados, sem abortar por um arquivo inválido.
- [ ] T-03, Implementar classificação de erro real e categorias por regex. Confiança: 🟢
  - Origem no legado: `src/learn/detector.rs`.
  - Critério de pronto: cancelamentos humanos e saídas sem indicador de erro não entram no fluxo.
- [ ] T-04, Implementar janela de correção, similaridade, exclusões e cálculo de confiança. Confiança: 🟢
  - Origem no legado: `src/learn/detector.rs`.
  - Critério de pronto: pares fora da janela, apenas-path, idênticos e TDD são rejeitados.
- [ ] T-05, Deduplicar pares e aplicar os limiares de confiança e recorrência. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`, `src/learn/detector.rs`.
  - Critério de pronto: grupos conservam o melhor exemplo e saem ordenados por ocorrência.
- [ ] T-06, Integrar saídas textual e JSON, delegando a persistência opcional de regras. Confiança: 🟢
  - Origem no legado: `src/learn/mod.rs`, `src/learn/report.rs`.
  - Critério de pronto: ausências de sessão/correção têm saída bem-sucedida e formatos retornam conteúdo equivalente.

## Tarefas de Teste

- [ ] TT-01, Testar reconhecimento de erro real, rejeição de usuário e classificação por categoria. Confiança: 🟢 `src/learn/detector.rs`
- [ ] TT-02, Testar janela de três comandos, similaridade, ciclos TDD e exploração de caminhos. Confiança: 🟢 `src/learn/detector.rs`
- [ ] TT-03, Testar deduplicação, ordenação e preservação do exemplo mais confiante. Confiança: 🟢 `src/learn/detector.rs`
- [ ] TT-04, Testar relatório vazio, texto com recorrência, JSON e persistência de regras. Confiança: 🟢 `src/learn/report.rs`, `src/learn/mod.rs`
- [ ] TT-05, Validar o detector contra histórico real e medir falsos positivos. Confiança: 🔴

## Ordem Sugerida

1. T-01 a T-03 estabelecem os dados e o filtro de entradas.
2. T-04 e T-05 materializam a heurística de recomendação.
3. T-06 e as tarefas de teste fecham o contrato de consumo.

## Decisoes e Lacunas

- 🟢 Os limiares atuais devem ser preservados como perfil versionado `legacy-v1`; recalibracoes futuras exigem dataset, benchmark, falsos positivos, economia e protecao `never_worse`. Decisao do usuario em 2026-07-16.

- Calibrar a heurística com histórico real antes de tratar as recomendações como regra de equipe.
- 🟢 O perfil `legacy-v1` preserva a ordenacao observada sem prometer cronologia global; uma alteracao futura exige dataset e benchmark. Decisao do usuario em 2026-07-16.
