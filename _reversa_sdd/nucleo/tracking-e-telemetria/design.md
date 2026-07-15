# Tracking e Telemetria, Design Tecnico

> Reconstrucao de `src/core/tracking.rs`, `telemetry.rs` e `telemetry_cmd.rs`. 🟢 Confirmado no codigo legado; 🟡 inferido; 🔴 lacuna.

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| `Tracker::new` | `()` | `Result<Tracker>` | Abre SQLite, cria schema e aplica migracoes. 🟢 `src/core/tracking.rs:249` |
| `Tracker::record` | `(&str, &str, usize, usize, u64)` | `Result<()>` | Persiste contagens, economia e tempo. 🟢 `src/core/tracking.rs:402` |
| `TimedExecution::track` | `(&str, &str, &str, &str)` | `()` | Estima tokens e grava de modo best-effort. 🟢 `src/core/tracking.rs:1356` |
| `TimedExecution::track_passthrough` | `(&str, &str)` | `()` | Grava tempo com tokens zero. 🟢 `src/core/tracking.rs:1392` |
| `maybe_ping` | `()` | `()` | Agenda ping se todos os gates passarem. 🟢 `src/core/telemetry.rs:22` |
| `telemetry_cmd::run` | `(&TelemetrySubcommand)` | `Result<()>` | Despacha `status`, `enable`, `disable`, `forget`. 🟢 `src/core/telemetry_cmd.rs:12` |

## Fluxo Principal

1. Ao iniciar uma medicao, `TimedExecution` armazena `Instant`; ao terminar, estima tokens de entrada/saida e tenta abrir `Tracker`. 🟢 `src/core/tracking.rs:1326-1369`
2. `Tracker::new` cria diretorio, abre SQLite, tenta WAL/busy timeout, cria tabelas e aplica migracoes aditivas. 🟢 `src/core/tracking.rs:249-326`
3. `record` calcula economia saturada e percentual, insere timestamp, comandos, projeto e duracao, depois executa a retencao. 🟢 `src/core/tracking.rs:402-449`
4. `maybe_ping` retorna cedo sem URL, com opt-out, config indisponivel, consentimento ausente/negado, telemetria desabilitada ou marker recente. 🟢 `src/core/telemetry.rs:22-60`
5. Quando elegivel, toca o marker antes da thread e envia em segundo plano com dados anonimizados e estatisticas disponiveis. 🟢 `src/core/telemetry.rs:62-142`
6. Os subcomandos alteram consentimento; `forget` remove identificadores e banco local, depois tenta erasure remoto. 🟢 `src/core/telemetry_cmd.rs:63-182`

## Fluxos Alternativos

- **WAL indisponivel:** a configuracao de concorrencia e ignorada e o banco continua abrindo. 🟢 `src/core/tracking.rs:255-261`
- **Falha no tracker durante medicao:** `track` e `track_passthrough` ignoram a falha de gravacao. 🟢 `src/core/tracking.rs:1361-1369`, `src/core/tracking.rs:1395-1397`
- **Tracking indisponivel para telemetria:** o payload usa valores neutros em vez de bloquear o ping. 🟢 `src/core/telemetry.rs:79-106`
- **`enable` em entrada nao interativa:** o comando falha sem aceitar consentimento por pipe. 🟢 `src/core/telemetry_cmd.rs:63-100`
- **Erasure remoto indisponivel:** dados locais ja sao removidos e o usuario recebe orientacao de contato. 🟢 `src/core/telemetry_cmd.rs:143-157`

## Dependencias

- `rusqlite`, `chrono` e constantes de caminho: banco, datas e retencao. 🟢 `src/core/tracking.rs:32-64`
- `core::config`: configuracao e consentimento de telemetria. 🟢 `src/core/telemetry.rs:33-48`
- `sha2`, `ureq` e `serde_json`: hash de dispositivo e transporte do payload. 🟢 `src/core/telemetry.rs:8`, `src/core/telemetry.rs:108-150`
- `hooks::init`: persistencia da decisao de consentimento. 🟢 `src/core/telemetry_cmd.rs:92`, `src/core/telemetry_cmd.rs:104`

## Decisoes de Design Identificadas

| Decisao | Evidencia | Confianca |
|---------|-----------|-----------|
| Persistencia local usa SQLite com WAL e timeout como otimizacoes nao fatais. | `src/core/tracking.rs:255-261` | 🟢 |
| Retencao ocorre apos cada insercao e abrange comandos e falhas de parse. | `src/core/tracking.rs:435-449` | 🟢 |
| Telemetria falha fechada e fire-and-forget. | `src/core/telemetry.rs:20-68` | 🟢 |
| Marker e atualizado antes da rede para evitar duplo ping. | `src/core/telemetry.rs:50-68` | 🟢 |
| Esquecimento remove estado local antes de tentar coordenacao remota. | `src/core/telemetry_cmd.rs:109-157` | 🟢 |

## Estado Interno

- Tabela `commands` armazena timestamp, comandos, tokens, economia, percentual, duracao e projeto; `parse_failures` registra fallback. 🟢 `src/core/tracking.rs:263-324`
- `TimedExecution` mantem apenas o instante inicial. 🟢 `src/core/tracking.rs:1306-1329`
- Telemetria usa salt cacheado e arquivo marker para identidade pseudonima e controle de frequencia. 🟢 `src/core/telemetry.rs:12-18`, `src/core/telemetry.rs:50-68`

## Observabilidade

- As consultas de tracking alimentam ganho diario, semanal, mensal, por comando e por projeto. 🟢 `src/core/tracking.rs`
- O payload inclui versao, plataforma, economia e estatisticas agregadas quando o tracker estiver disponivel. 🟢 `src/core/telemetry.rs:71-142`
- `telemetry status` torna visiveis consentimento, habilitacao, override de ambiente e presenca de hash. 🟢 `src/core/telemetry_cmd.rs:21-60`

## Riscos e Lacunas

- 🔴 A concorrencia real entre multiplas instancias em sistemas de arquivos de rede nao foi exercitada.
- 🔴 O endpoint compilado, a retencao remota e o processamento do pedido de erasure nao foram validados estaticamente.
- 🟡 A coleta de metricas ampliadas pode evoluir sem alterar a estrutura local do banco, pois o payload admite valores neutros quando o tracker falha. `src/core/telemetry.rs:79-106`
