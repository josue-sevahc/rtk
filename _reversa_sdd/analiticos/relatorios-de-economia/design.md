# Relatórios de Economia, Design Técnico

## Interface

| Símbolo | Assinatura | Retorno | Observação |
|---|---|---|---|
| `cc_economics::run` | `daily, weekly, monthly, all, format, verbose` | `Result<()>` | Seleciona saída. |
| `ccusage::fetch` | `Granularity` | `Result<Option<Vec<CcusagePeriod>>>` | Integração opcional. |

## Fluxo Principal

1. Criar `Tracker` e escolher texto, JSON ou CSV. 🟢
2. Buscar períodos `ccusage` e dados equivalentes do tracking. 🟢
3. Inserir cada fonte em mapa por chave e calcular métricas por `PeriodEconomics`. 🟢
4. Ordenar rótulos e emitir tabela ou serialização. 🟢

## Fluxos Alternativos

- `ccusage` no PATH é preferido; caso contrário o código tenta `npx --yes ccusage`. 🟢
- Se ambos falham ou o subprocesso retorna erro, há aviso e `Ok(None)`. 🟢
- Campos insuficientes permanecem `None`, representados por travessão na saída textual. 🟢

## Dependências

- `core::tracking::{Tracker, DayStats, WeekStats, MonthStats}`. 🟢
- `core::stream::exec_capture` para subprocessos. 🟢
- `chrono`, `serde`, `anyhow` e `HashMap`. 🟢

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Tolerar formatos antigo e atual de ccusage via alias `period`. | `src/analytics/ccusage.rs` | 🟢 |
| Mostrar métricas legacy somente em verbose. | `src/analytics/cc_economics.rs` | 🟢 |
| Conservar períodos de apenas uma fonte no merge. | `merge_daily/weekly/monthly` | 🟢 |

## Riscos e Lacunas

- 🟡 As razões de preço codificadas são uma premissa de produto, não uma cotação em tempo real.
