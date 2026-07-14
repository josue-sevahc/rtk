# Spec Impact Matrix - RTK

Matriz de impacto entre capacidades arquiteturais e componentes observados.

Legenda: **P** primario, **S** secundario, **-** sem impacto direto.

| Capacidade/Regra | main | cmds/core runner | hooks/discover | tracking/analytics | learn | OpenClaw | Externos |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Parse, fallback e dispatch de CLI | P | S | S | S | - | - | S |
| Compactar sem piorar a saida bruta | S | P | S | S | - | - | P |
| Preservar exit code e sinais | P | P | - | S | - | - | P |
| Rewrite transparente de comandos | S | S | P | - | - | P | S |
| Preservar permissao do host | - | - | P | - | - | P | P |
| Trust de filtros TOML | S | P | P | - | - | - | - |
| Integridade de hooks | P | - | P | - | - | - | S |
| Tracking e relatorios de economia | S | P | S | P | - | - | - |
| Descobrir comandos nao filtrados | - | - | P | S | - | - | S |
| Aprender correcoes recorrentes | - | - | S | S | P | - | S |
| Telemetria opt-in | P | S | S | P | - | - | P |
| Suporte a nova ferramenta externa | S | P | P | S | - | S | P |

## Impactos de mudanca

| Mudanca proposta | Componentes que devem ser avaliados | Risco principal |
|---|---|---|
| Novo wrapper/filtro | `main`, `cmds`, `core::runner`, fixtures/testes, registry e filtros TOML | perda de fidelidade, piora de tokens ou exit code mascarado. |
| Novo host de agente | `hooks::init`, `permissions`, `hook_cmd`, docs e testes de payload | escalonamento de permissao ou quebra de protocolo JSON. |
| Alterar regra de rewrite | `discover::lexer`, `registry`, `permissions`, adaptadores e OpenClaw | rewrite de construto nao atestavel ou bypass de deny. |
| Alterar schema de tracking | `core::tracking`, analytics, telemetria e documentacao | migracao retrocompativel e agregacoes incorretas. |
| Alterar telemetria | config, init, `telemetry`, comando de controle e docs | violacao de consentimento/privacidade. |
| Alterar filtro TOML | `toml_filter`, trust, build.rs e testes inline | ocultar output relevante ou habilitar conteudo nao confiavel. |

## Itens de maior impacto

- 🟢 `core::runner` e `core::guard` atravessam praticamente todos os wrappers; qualquer mudanca exige verificar fidelidade, tracking e exit code.
- 🟢 `hooks::permissions` e `hooks::hook_cmd` sao fronteira de seguranca para todos os hosts integrados.
- 🟢 `discover::registry` conecta reconhecimento de comando aos adaptadores; uma regra nova repercute em CLI, hooks e OpenClaw.
- 🔴 A matriz e estatica; nao substitui testes de compatibilidade contra as versoes reais das ferramentas e hosts.
