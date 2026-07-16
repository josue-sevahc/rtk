# Integrações de Agentes, Tarefas de Implementação

> Sequência para reconstruir `src/hooks/` preservando os contratos observados.

## Pré-requisitos

- [ ] 🟢 Disponibilizar configuração, acesso ao filesystem do usuário, JSON, SHA-256 e escrita atômica. Origem: `src/hooks/init.rs`, `src/hooks/integrity.rs`.
- [ ] 🟢 Implementar ou integrar o registry de classificação/reescrita de `discover`. Origem: `src/hooks/rewrite_cmd.rs`.

## Tarefas

- [ ] T-01, Modelar hosts, formatos de hook, decisões, permissões e status de integridade.
  - Origem no legado: `src/hooks/mod.rs`, `src/hooks/constants.rs`, `src/hooks/permissions.rs`, `src/hooks/integrity.rs`.
  - Critério de pronto: tipos representam `Allow`, `Ask`, `Deny`, estados de integridade e diferenças de protocolo sem usar strings soltas.
  - Confiança: 🟢

- [ ] T-02, Implementar instalação, desinstalação, patch idempotente, backup e escrita atômica por host.
  - Origem no legado: `src/hooks/init.rs`.
  - Critério de pronto: duas execuções de instalação não duplicam configuração e falha de escrita não deixa arquivo parcialmente gravado.
  - Confiança: 🟢

- [ ] T-03, Implementar avaliação de permissões composta com precedência `Deny > Ask > Allow > Default` e recusa de construtos não atestáveis.
  - Origem no legado: `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: um único segmento negado impede auto-allow; construto ambíguo exige confirmação.
  - Confiança: 🟢

- [ ] T-04, Implementar `rtk rewrite`, seus quatro códigos de saída e delegação ao registry de rewrite.
  - Origem no legado: `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: rewrite permitido, passthrough, deny e ask produzem respectivamente códigos 0, 1, 2 e 3.
  - Confiança: 🟢

- [ ] T-05, Implementar handlers JSON de Claude, Cursor, Gemini, Copilot e Droid, com limite de stdin e fallback não bloqueante.
  - Origem no legado: `src/hooks/hook_cmd.rs`.
  - Critério de pronto: cada payload válido retorna o shape exigido; input inválido não corrompe stdout nem bloqueia o host.
  - Confiança: 🟢

- [ ] T-06, Implementar baseline SHA-256, verificação runtime e proteção de hook adulterado.
  - Origem no legado: `src/hooks/integrity.rs`.
  - Critério de pronto: baseline válido verifica, divergência vira `Tampered` e somente esse estado bloqueia o caminho protegido.
  - Confiança: 🟢

- [ ] T-07, Implementar trust store de filtros TOML e aprovação explícita com análise de risco.
  - Origem no legado: `src/hooks/trust.rs`.
  - Critério de pronto: conteúdo externo só é devolvido para estado confiável/override válido; hash alterado volta a impedir carregamento.
  - Confiança: 🟢

- [ ] T-08, Implementar auditoria opt-in e aviso rate-limited de integração ausente ou obsoleta.
  - Origem no legado: `src/hooks/hook_audit_cmd.rs`, `src/hooks/hook_check.rs`.
  - Critério de pronto: auditoria sanitiza delimitadores e aviso não se repete mais de uma vez ao dia.
  - Confiança: 🟢

## Tarefas de Teste

- [ ] TT-01, Testar idempotência, backup e rollback de patch por host.
  - Origem no legado: `src/hooks/init.rs`.
  - Critério de pronto: fixtures preservam conteúdo não RTK e não produzem segunda inserção.
  - Confiança: 🟢
- [ ] TT-02, Testar decisões de permissão e todos os códigos de `rtk rewrite`.
  - Origem no legado: `src/hooks/permissions.rs`, `src/hooks/rewrite_cmd.rs`.
  - Critério de pronto: casos de allow, ask, deny, passthrough e shell ambíguo estão cobertos.
  - Confiança: 🟢
- [ ] TT-03, Testar payloads válidos, vazios e inválidos para todos os hosts suportados.
  - Origem no legado: `src/hooks/hook_cmd.rs`.
  - Critério de pronto: cada host recebe JSON válido ou fallback seguro sem saída adicional.
  - Confiança: 🟢
- [ ] TT-04, Testar hash íntegro, adulterado, ausente e órfão, além de trust por hash alterado.
  - Origem no legado: `src/hooks/integrity.rs`, `src/hooks/trust.rs`.
  - Critério de pronto: somente adulteração bloqueia e filtro alterado não volta a ser aceito automaticamente.
  - Confiança: 🟢

## Ordem Sugerida

1. Implementar T-01 a T-04 para fixar os contratos de decisão e bridge CLI.
2. Implementar T-02 e T-05 por host, usando os contratos já estáveis.
3. Adicionar T-06 a T-08 como barreiras e diagnóstico antes da validação de integração real.

## Lacunas Pendentes (🔴)

- 🔴 Executar matriz de compatibilidade real por versão de host e sistema operacional.
- 🔴 Confirmar se todos os formatos de configuração de terceiros permanecem estáveis.
