# Instalação e Configuração de Integrações, Tarefas de Implementação

## Tarefas

- [ ] T-01, Modelar alvo de agente, escopo, `PatchMode`, `FilterTrust`, `PatchResult` e `InitContext`.
  - Origem no legado: `src/hooks/init.rs`.
  - Critério de pronto: flags inválidas e combinações não suportadas são rejeitadas antes de qualquer escrita.
  - Confiança: 🟢
- [ ] T-02, Implementar resolução de diretórios e artefatos de instalação por host.
  - Origem no legado: `src/hooks/init.rs`, `src/hooks/constants.rs`.
  - Critério de pronto: cada host recebe somente seu settings, instrução, plugin ou script esperado.
  - Confiança: 🟢
- [ ] T-03, Implementar patch idempotente, backup, `dry_run` e escrita atômica.
  - Origem no legado: `src/hooks/init.rs`.
  - Critério de pronto: conteúdo não RTK é preservado, segunda instalação não duplica e falha não deixa arquivo parcial.
  - Confiança: 🟢
- [ ] T-04, Criar e remover baseline SHA-256 e classificar todos os estados de integridade.
  - Origem no legado: `src/hooks/integrity.rs`.
  - Critério de pronto: hash íntegro, adulterado, ausente e órfão recebem status distintos; somente adulterado bloqueia o caminho protegido.
  - Confiança: 🟢
- [ ] T-05, Implementar diagnóstico de hook binário/script e aviso diário.
  - Origem no legado: `src/hooks/hook_check.rs`.
  - Critério de pronto: hook nativo atualizado não alerta; ausência ou obsolescência alerta no máximo uma vez por dia.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar patch com JSON/Markdown preexistente, repetição e dry-run.
  - Origem no legado: `src/hooks/init.rs`.
  - Critério de pronto: fixtures provam preservação e idempotência.
  - Confiança: 🟢
- [ ] TT-02, Testar integridade, formato de hash e permissões Unix quando disponíveis.
  - Origem no legado: `src/hooks/integrity.rs`.
  - Critério de pronto: divergência é detectada e baseline inválido é rejeitado.
  - Confiança: 🟢

## Lacunas Pendentes (🔴)

- 🔴 Validar a matriz real de sistemas operacionais e versões de agentes suportadas.
