# C4 - Containers

```mermaid
flowchart TB
    subgraph dispositivo[Dispositivo do usuario]
        subgraph rtk[RTK]
            cli[CLI e dispatcher\nRust + Clap]
            engine[Execucao, streaming e filtros\nRust]
            hooks[Hooks e descoberta\nRust]
            analytics[Analytics e learn\nRust]
        end
        db[(SQLite history.db)]
        config[config.toml, filtros e trust store]
        plugin[Plugin OpenClaw\nTypeScript]
    end

    host[Host de agente] -->|JSON stdin/stdout| hooks
    plugin -->|subprocesso rtk rewrite| cli
    cli --> engine
    cli --> hooks
    cli --> analytics
    hooks --> engine
    engine -->|metricas| db
    analytics -->|consultas| db
    cli -->|carrega| config
    hooks -->|le/grava| config
    engine -->|processos filhos| tools[Ferramentas externas]
    cli -.->|HTTP opcional| telemetry[Endpoint de telemetria]
```

| Container | Tecnologias | Comunicacao | Confianca |
|---|---|---|---|
| CLI e dispatcher | Rust, Clap | chamadas de modulo e processos filhos | 🟢 |
| Motor de execucao | Rust, `std::process`, parsers/TOML | stdout/stderr, streaming e tracking | 🟢 |
| Hooks e descoberta | Rust, JSON, SHA-256 | stdin/stdout, arquivos de settings e regras | 🟢 |
| Analytics e learn | Rust, SQLite | queries e historicos locais | 🟢 |
| SQLite | `rusqlite` bundled | arquivo local, WAL/busy timeout tentados | 🟢 |
| Plugin OpenClaw | TypeScript | API OpenClaw e subprocesso | 🟢 |

🟡 O limite entre `core` e `hooks` e permeavel por chamadas diretas de `core` para `hooks`; isso indica acoplamento transversal, nao um container adicional.
