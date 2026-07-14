# C4 - Componentes Internos

```mermaid
flowchart LR
    input[Argumentos CLI ou payload de hook] --> main[main.rs\nparse, fallback e dispatch]
    main --> registry[discover::registry\nclassificacao e rewrite]
    main --> permissions[hooks::permissions\nveredito allow/ask/deny]
    permissions --> hookcmd[hooks::hook_cmd\nadaptadores por host]
    main --> wrappers[cmds::*\nwrappers por ecossistema]
    registry --> toml[core::toml_filter\nfiltros declarativos]
    wrappers --> runner[core::runner e stream\nexecucao e filtragem]
    toml --> runner
    runner --> guard[core::guard\nnever_worse]
    guard --> tracking[core::tracking\nSQLite e metricas]
    main --> analytics[analytics::*\ngain, sessao e economia]
    main --> learn[learn::*\ncorrecoes recorrentes]
    main --> integrity[hooks::trust e integrity\nconfianca e hashes]
```

| Componente | Responsabilidade | Dependencias principais |
|---|---|---|
| `main` | Entrada, flags, fallback e dispatch. | todos os modulos top-level. |
| `discover::registry` | Decide se e como um comando pode virar `rtk ...`. | lexer, regras e configuracao de hooks. |
| `hooks::permissions` / `hook_cmd` | Preserva permissoes do host e adapta protocolos de hook. | regras de host, JSON, registry. |
| `cmds::*` | Filtros especializados por ferramenta/ecossistema. | runner, parsers e utilitarios. |
| `core::runner` / `stream` / `guard` | Execucao, captura, filtragem, tee, exit code e guard de tamanho. | processos filhos e tracking. |
| `core::tracking` | Schema SQLite, retencao e agregacoes. | `rusqlite`, configuracao. |
| `analytics::*` / `learn::*` | Relatorios, descoberta de uso e regras extraidas. | tracking e historicos dos hosts. |
| `hooks::trust` / `integrity` | Trust por hash e verificacao de hooks. | SHA-256, filesystem e config. |

🟢 O `runner` e a rota compartilhada dominante para wrappers; 🟡 a dependencia de `core` em `hooks` merece refatoracao ou uma fronteira documentada.
