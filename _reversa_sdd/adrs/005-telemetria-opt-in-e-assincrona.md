# ADR 005 - Telemetria opt-in e assincrona

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: commit `6a5bc84`, `src/core/telemetry.rs`, `src/core/telemetry_cmd.rs`

## Contexto

Metricas de uso ajudam a evoluir filtros, mas enviam dados para fora do ambiente local.

## Decisao

Exigir endpoint compilado, consentimento explicito e configuracao habilitada. Respeitar `RTK_TELEMETRY_DISABLED=1`, limitar o ping a uma janela aproximada de 23 horas e enviar fora do caminho bloqueante do CLI. Disponibilizar disable e forget para controle local.

## Consequencias

Ausencia de consentimento impede coleta remota. O marker antecipado evita duplicidade, mas uma falha de rede pode consumir a janela sem entrega.
