# C4 - Contexto do Sistema

## Diagrama

```mermaid
flowchart LR
    dev[Desenvolvedor]
    agent[Agente de coding]
    host[Hosts de agentes\nClaude, Cursor, Gemini, Copilot, Droid]
    rtk[RTK\nCLI local para compactar saida de comandos]
    tools[Ferramentas externas\nGit, build, teste, lint, cloud e sistema]
    openclaw[OpenClaw]
    telemetry[Endpoint de telemetria\nopcional]

    dev -->|executa e configura| rtk
    agent -->|solicita comandos| host
    host -->|hook JSON/configuracao| rtk
    rtk -->|stdout compacto e exit code| host
    rtk -->|executa como processo filho| tools
    openclaw -->|plugin chama rtk rewrite| rtk
    rtk -.->|HTTP JSON, somente com consentimento| telemetry
```

## Relacionamentos

| Origem | Destino | Relacao | Confianca |
|---|---|---|---|
| Desenvolvedor | RTK | Opera CLI, instala hooks e administra configuracao/trust. | 🟢 |
| Hosts de agentes | RTK | Executam handlers ou permitem rewrite de comando. | 🟢 |
| RTK | Ferramentas externas | Executa e comprime output sem substituir a ferramenta. | 🟢 |
| OpenClaw | RTK | Plugin intercepta `exec` e invoca `rtk rewrite`. | 🟢 |
| RTK | Endpoint de telemetria | Envia ping opcional e assincrono. | 🟢 |

🔴 A compatibilidade concreta depende das versoes externas de cada host e ferramenta.
