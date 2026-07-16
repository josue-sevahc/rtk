# Analíticos

> Contrato operacional do módulo `src/analytics/`. As afirmações marcadas 🟢 foram extraídas do código legado.

## Visão Geral

O módulo expõe dashboards somente de leitura sobre economia de tokens, custo estimado e adoção do RTK. Ele consulta o tracking SQLite e históricos locais do Claude Code, sem registrar métricas nem alterar o banco de tracking. 🟢

## Responsabilidades

- Exibir economia acumulada e por período com `rtk gain`. 🟢
- Correlacionar gastos do Claude Code com tokens poupados em `rtk cc-economics`. 🟢
- Medir a adoção de comandos cobertos pelo RTK em sessões recentes com `rtk session`. 🟢
- Degradar de forma legível quando não há dados locais ou quando `ccusage` não está disponível. 🟢

## Regras de Negócio

- O módulo analítico não escreve no banco de tracking. 🟢
- `gain --reset` só apaga dados após confirmação; `--yes` suprime a pergunta. 🟢
- Escopo `--project` usa o diretório atual canonizado; sem ele a consulta é global. 🟢
- Estimativas de economia são aproximações baseadas em tokens registrados/estimados, não em tokenização exata. 🟢

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Consultar e apresentar o resumo de tracking por comando. | Must | Com dados, `gain` mostra contagens, tokens e percentuais. |
| RF-02 | Oferecer visões diária, semanal, mensal, histórico, gráfico e exportações JSON/CSV. | Must | Cada flag seleciona a visão/exportação correspondente. |
| RF-03 | Exibir alertas de hooks, filtros não confiáveis e bypass `RTK_DISABLED`. | Should | Alertas vão para stderr sem invalidar o relatório. |
| RF-04 | Disponibilizar relatórios de economia e adoção por sessão. | Must | Os subcomandos retornam tabela ou mensagem de ausência de dados. |

## Requisitos Não Funcionais

| Tipo | Requisito inferido | Evidência | Confiança |
|---|---|---|---|
| Segurança | Reset requer confirmação interativa por padrão. | `src/analytics/gain.rs` | 🟢 |
| Confiabilidade | Falhas de ferramenta externa podem degradar para dados RTK locais. | `src/analytics/ccusage.rs` | 🟢 |
| Privacidade | Analytics lê dados locais; o módulo não persiste métricas. | `src/analytics/README.md` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: exibir ganhos existentes
  Dado que o tracking possui comandos registrados
  Quando o usuário executa `rtk gain`
  Então recebe KPIs e uma tabela por comando

Cenário: não haver tracking
  Dado que o tracking está vazio
  Quando o usuário executa `rtk gain`
  Então recebe orientação para executar comandos RTK

Cenário: reset cancelado
  Dado que o usuário executa `rtk gain --reset`
  Quando não confirma a operação
  Então os dados permanecem intactos
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Consulta read-only ao tracking | Must | É a base dos três relatórios. |
| Exportação estruturada | Must | Contrato CLI explícito. |
| Alertas operacionais | Should | Ajuda a explicar perda silenciosa de savings. |

## Rastreabilidade de Código

| Arquivo | Cobertura | Confiança |
|---|---|---|
| `src/analytics/README.md` | Limites e responsabilidade do módulo | 🟢 |
| `src/analytics/mod.rs` | Registro dos submódulos | 🟢 |
| `src/analytics/gain.rs` | Dashboard de economia | 🟢 |
| `src/analytics/cc_economics.rs` e `ccusage.rs` | Economia monetária | 🟢 |
| `src/analytics/session_cmd.rs` | Adoção por sessão | 🟢 |
