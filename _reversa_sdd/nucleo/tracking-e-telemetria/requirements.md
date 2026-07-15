# Tracking e Telemetria

> Contrato operacional de `core::tracking`, `core::telemetry` e `core::telemetry_cmd`. 🟢 Confirmado no codigo legado; 🟡 inferido; 🔴 lacuna.

## Visao Geral

O subunit registra execucoes e economia estimada em SQLite local, expõe agregacoes para analytics e realiza telemetria anonima apenas por opt-in. 🟢 `src/core/tracking.rs:249`, `src/core/telemetry.rs:22`

## Responsabilidades

- Criar e migrar o banco local de comandos e falhas de parse. 🟢 `src/core/tracking.rs:249-326`
- Registrar tokens, economia, duracao e caminho canonico do projeto; remover historico vencido. 🟢 `src/core/tracking.rs:402-449`
- Registrar execucoes passthrough com tokens zero para nao contaminar as estatisticas. 🟢 `src/core/tracking.rs:1372-1397`
- Enviar ping somente quando endpoint, configuracao e consentimento permitirem. 🟢 `src/core/telemetry.rs:22-68`
- Expor `status`, `enable`, `disable` e `forget`, incluindo apagamento local e pedido de erasure remoto. 🟢 `src/core/telemetry_cmd.rs:5-18`, `src/core/telemetry_cmd.rs:109-182`

## Regras de Negocio

- WAL e `busy_timeout=5000` sao tentativas nao fatais para acesso concorrente. 🟢 `src/core/tracking.rs:255-261`
- Migracoes de `exec_time_ms` e `project_path` sao idempotentes por erro ignorado; `NULL` antigo de projeto e normalizado. 🟢 `src/core/tracking.rs:281-308`
- Economia e `input_tokens - output_tokens` com subtracao saturada; registros acima de 90 dias sao removidos. 🟢 `src/core/tracking.rs:410-449`, `src/core/constants.rs:6`
- Consultas por projeto usam igualdade ou `GLOB` com separador de caminho, sem os curingas de `LIKE`. 🟢 `src/core/tracking.rs:51-61`
- Telemetria exige URL compilada, ausencia de `RTK_TELEMETRY_DISABLED=1`, config habilitada e consentimento `Some(true)`. 🟢 `src/core/telemetry.rs:22-48`
- O marcador e tocado antes do envio e bloqueia novo ping por 23 horas; a requisicao ocorre em thread. 🟢 `src/core/telemetry.rs:50-68`
- `enable` exige terminal interativo e consentimento explicito; `forget` desabilita, apaga salt/marker/banco e tenta erasure remoto. 🟢 `src/core/telemetry_cmd.rs:63-100`, `src/core/telemetry_cmd.rs:109-182`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|------------|-------------------|
| RF-01 | Inicializar SQLite com tabelas, indices e migracoes compatíveis. 🟢 | Must | Dado banco novo ou antigo, quando o tracker abrir, entao `commands` e `parse_failures` ficam disponiveis com colunas de duracao e projeto. |
| RF-02 | Registrar execucoes com tokens, economia, duracao e projeto, removendo registros antigos. 🟢 | Must | Dado uma execucao, quando registrada, entao seus campos podem ser consultados e entradas com mais de 90 dias sao removidas. |
| RF-03 | Registrar passthrough com tokens zero e tempo decorrido. 🟢 | Should | Dado uma execucao sem captura, quando terminar, entao existe registro temporal que nao reduz a taxa de economia. |
| RF-04 | Avaliar telemetria de modo fail-closed e nao bloqueante. 🟢 | Must | Dado qualquer pre-requisito ausente, quando `maybe_ping` rodar, entao nao ha envio; dado todos presentes, o CLI nao espera a rede. |
| RF-05 | Permitir consultar, consentir, desabilitar e esquecer telemetria. 🟢 | Should | Dado `forget`, quando concluir, entao consentimento e dados locais sao removidos e erasure remoto e tentado. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia | Confianca |
|------|--------------------|-----------|-----------|
| Disponibilidade | Falhas de tracking e telemetria nao devem derrubar a execucao do comando. | `src/core/tracking.rs:1361-1369`, `src/core/telemetry.rs:20-68` | 🟢 |
| Concorrencia | SQLite tenta WAL e espera ocupacao por ate 5 segundos, sem tornar incompatibilidade fatal. | `src/core/tracking.rs:255-261` | 🟢 |
| Privacidade | Coleta remota requer consentimento explicito, opt-out e limite de frequencia. | `src/core/telemetry.rs:22-68` | 🟢 |
| Privacidade | Esquecimento limpa identificadores e banco local antes de tentar a requisicao remota. | `src/core/telemetry_cmd.rs:109-157` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Registrar execucao filtrada
  Dado uma saida bruta e uma saida exibida
  Quando TimedExecution registra a execucao
  Entao o banco guarda tokens, economia, duracao e projeto atual

Cenario: Bloquear ping sem consentimento
  Dado consentimento ausente ou negado
  Quando maybe_ping e chamado
  Entao nenhum envio de telemetria e iniciado

Cenario: Esquecer dados de telemetria
  Dado telemetria previamente habilitada
  Quando o usuario executa telemetry forget
  Entao consentimento, salt, marcador e banco local sao removidos
  E uma requisicao de erasure remoto e tentada quando houver endpoint
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Tracking persistente e retencao | Must | Suporta `gain`, analytics e a medicao de valor do produto. 🟢 |
| Telemetria fail-closed e nao bloqueante | Must | Limite de privacidade e disponibilidade do CLI. 🟢 |
| Registro de passthrough e controles do usuario | Should | Mantem observabilidade sem alterar o caminho principal. 🟢 |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/core/tracking.rs` | `Tracker::new`, `Tracker::record`, `cleanup_old` | 🟢 |
| `src/core/tracking.rs` | `TimedExecution::track`, `track_passthrough` | 🟢 |
| `src/core/telemetry.rs` | `maybe_ping`, `send_ping` | 🟢 |
| `src/core/telemetry_cmd.rs` | `TelemetrySubcommand`, `run_forget` | 🟢 |
