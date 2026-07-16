# Parser de Saida, Tarefas de Implementacao

> Sequencia para reconstruir `src/parser/` preservando os contratos observados.

## Pre-requisitos

- [ ] 🟢 Disponibilizar configuracao de limites com `passthrough_max_chars`.
- [ ] 🟢 Disponibilizar serializacao/deserializacao para os tipos canonicos usados por parsers de ferramentas.

## Tarefas

- [ ] T-01, Modelar `ParseResult<T>` com os tres tiers e implementar consulta, mapeamento e coleta de avisos. Confianca: 🟢
  - Origem no legado: `src/parser/mod.rs`.
  - Criterio de pronto: `Full`, `Degraded` e `Passthrough` preservam os valores e semanticas observados.
- [ ] T-02, Definir `OutputParser` com tipo associado, `parse` e a politica de `parse_with_tier`. Confianca: 🟢
  - Origem no legado: `src/parser/mod.rs`.
  - Criterio de pronto: resultado acima de `max_tier` retorna passthrough truncado do input original.
- [ ] T-03, Implementar truncamento por caracteres com marcador e limite obtido da configuracao. Confianca: 🟢
  - Origem no legado: `src/parser/mod.rs`, `src/core/config.rs`.
  - Criterio de pronto: entradas curtas sao preservadas e entradas longas mantem UTF-8 valido com contagens no marcador.
- [ ] T-04, Implementar extracao de objeto JSON por marcador e balanceamento de chaves sensivel a strings e escapes. Confianca: 🟢
  - Origem no legado: `src/parser/mod.rs`.
  - Criterio de pronto: objetos com prefixos, braces aninhadas e valores multibyte sao extraidos integralmente.
- [ ] T-05, Modelar `TestResult`, `TestFailure`, `DependencyState` e `Dependency` com os campos serializaveis observados. Confianca: 🟢
  - Origem no legado: `src/parser/types.rs`.
  - Criterio de pronto: parsers de teste e dependencias podem preencher todos os campos canonicos.
- [ ] T-06, Integrar avisos de degradacao e passthrough nos consumidores de comandos. Confianca: 🟢
  - Origem no legado: `src/parser/mod.rs`, `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs`.
  - Criterio de pronto: o modo detalhado informa degradacao e o fallback mostra texto limitado.

## Tarefas de Teste

- [ ] TT-01, Testar tier, `is_ok`, `map` e avisos para as tres variantes de `ParseResult`. Confianca: 🟢 `src/parser/mod.rs`
- [ ] TT-02, Testar truncamento ASCII e multibyte, incluindo Thai e emoji. Confianca: 🟢 `src/parser/mod.rs`
- [ ] TT-03, Testar extracao JSON limpa, com prefixos, strings contendo braces, escapes e valores CJK/emoji. Confianca: 🟢 `src/parser/mod.rs`
- [ ] TT-04, Testar cada parser consumidor nos tres tiers com fixtures representativas de versoes de ferramentas. Confianca: 🟡 `src/parser/README.md`
- [ ] TT-05, Medir frequencia de degradacao em uso real antes de adicionar alerta operacional. Confianca: 🔴

## Ordem Sugerida

1. T-01 a T-03 estabelecem o resultado, o contrato e o limite de seguranca.
2. T-04 e T-05 criam a base para parsers concretos e dados canonicos.
3. T-06 e os testes fecham a integracao dos consumidores.

## Lacunas Pendentes (🔴)

- Definir quais sinais de degradacao devem ser persistidos ou alertados em producao.
