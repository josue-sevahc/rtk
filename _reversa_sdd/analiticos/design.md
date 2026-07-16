# Analíticos, Design Técnico

## Interface

| Símbolo | Assinatura | Retorno | Observação |
|---|---|---|---|
| `gain::run` | flags de escopo, período, formato e reset | `Result<()>` | Dashboard de tracking. |
| `cc_economics::run` | período, formato e verbose | `Result<()>` | Junta `ccusage` e tracking. |
| `session_cmd::run` | `verbose` | `Result<()>` | Analisa até 10 sessões Claude. |

## Fluxo Principal

1. O dispatcher CLI chama o submódulo em `src/analytics/mod.rs`. 🟢
2. A visão escolhida lê `Tracker` para obter métricas SQLite. 🟢
3. O módulo formata texto para terminal ou serializa JSON/CSV. 🟢
4. Nenhuma consulta analítica grava no tracking. 🟢

## Fluxos Alternativos

- **Tracking vazio:** imprime mensagem de orientação e encerra com sucesso. 🟢
- **`ccusage` ausente/falho:** retorna `None`, emite aviso e preserva a parte RTK do relatório. 🟢
- **Sessões inexistentes ou sem Bash:** imprime mensagem amigável. 🟢
- **Reset recusado:** imprime `Aborted.` sem alterar dados. 🟢

## Dependências

- `core::tracking::Tracker`: consultas de resumo, períodos, falhas e histórico. 🟢
- `core::display_helpers` e `core::utils`: tabelas, duração, tokens e moeda. 🟢
- `hooks::hook_check` e `hooks::trust`: avisos de eficácia. 🟢
- `discover::provider` e `discover::registry`: extração/classificação de comandos de sessões. 🟢
- Binário `ccusage` ou fallback `npx ccusage`: custo e tokens Claude Code. 🟢

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Separar apresentação analítica da escrita de métricas. | `src/analytics/README.md` | 🟢 |
| Manter saída legível em terminal e formatos de automação. | `src/analytics/gain.rs`, `cc_economics.rs` | 🟢 |
| Fazer integração com `ccusage` opcional. | `src/analytics/ccusage.rs` | 🟢 |

## Estado Interno

O módulo não mantém estado persistente próprio. `gain` pode resolver um caminho de projeto para filtrar consultas; relatórios usam estruturas temporárias como `SessionSummary` e `PeriodEconomics`. 🟢

## Observabilidade

Avisos operacionais e de ausência/falha de `ccusage` são enviados para stderr. A saída principal é escrita em stdout para permitir consumo por JSON/CSV. 🟢

## Riscos e Lacunas

- 🟢 Dados de `ccusage` são externos e seu contrato pode evoluir; aliases tratam formatos conhecidos.
- 🟡 A estimativa econômica depende de preços/razões codificados e pode precisar atualização.
