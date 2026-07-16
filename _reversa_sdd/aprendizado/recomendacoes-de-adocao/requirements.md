# Recomendações de Adoção

> Subunit de `aprendizado` reconstruída de `src/learn/mod.rs` e `src/learn/report.rs`. Afirmações 🟢 foram confirmadas no código legado; 🟡 são inferências; 🔴 exigem validação humana.

## Visão Geral

Esta subunit apresenta as correções detectadas para consumo humano ou automatizado e pode gravar um arquivo Markdown de orientação em `.claude/rules/cli-corrections.md`. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Responsabilidades

- Aplicar os limiares de confiança e ocorrência às correções encontradas. 🟢 `src/learn/mod.rs`
- Renderizar regras em relatório de terminal ou documento JSON. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`
- Gerar o arquivo Markdown local quando `--write-rules` é solicitado no modo textual e há regras. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Regras de Negócio

- O filtro de confiança ocorre antes da deduplicação; o de ocorrência ocorre depois. 🟢 `src/learn/mod.rs`
- O formato exatamente igual a `json` produz objeto estruturado; qualquer outro valor segue o relatório textual. 🟢 `src/learn/mod.rs`
- O arquivo de regras só é escrito se há regras e o caminho textual recebeu `--write-rules`. 🟢 `src/learn/mod.rs`
- O relatório textual vazio declara explicitamente que não houve correções. 🟢 `src/learn/report.rs`
- A escrita cria diretórios pais, agrupa por comando-base em ordem alfabética e registra recorrência quando maior que um. 🟢 `src/learn/report.rs`
- O arquivo alvo é regravado pela funcionalidade do produto quando a escrita é solicitada. 🟢 `src/learn/report.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Aplicar `min_confidence` antes de consolidar pares. 🟢 | Must | Pares abaixo do limiar não contribuem para regras. |
| RF-02 | Aplicar `min_occurrences` às regras consolidadas. 🟢 | Must | Regras pouco recorrentes são removidas do resultado final. |
| RF-03 | Expor sessões, total de correções e regras em JSON. 🟢 | Should | `--format json` retorna os campos previstos. |
| RF-04 | Exibir relatório textual legível, incluindo exemplo de erro. 🟢 | Should | Cada regra apresenta origem, destino, contagem e primeira linha do erro quando disponível. |
| RF-05 | Gravar recomendações Markdown sob solicitação explícita. 🟢 | Should | `--write-rules` cria ou atualiza o caminho esperado quando há regras. |

## Requisitos Não Funcionais

| Tipo | Requisito inferido | Evidência no código | Confiança |
|---|---|---|---|
| Usabilidade | O formato textual diferencia repetição com marcador visual e mostra um fragmento de erro. | `src/learn/report.rs` | 🟢 |
| Interoperabilidade | O JSON usa `serde_json` e expõe campos simples para consumidores externos. | `src/learn/mod.rs` | 🟢 |
| Persistência | A escrita cria diretórios ancestrais antes do arquivo de regras. | `src/learn/report.rs` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: Publicar recomendações em JSON
  Dado correções que atendem aos limiares configurados
  Quando `rtk learn --format json` executar
  Então a saída contém sessões analisadas, total de correções e regras

Cenário: Criar regras locais
  Dado pelo menos uma regra consolidada
  Quando `rtk learn --write-rules` executar em formato textual
  Então `.claude/rules/cli-corrections.md` é criado com grupos por comando-base
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Filtrar e consolidar recomendações elegíveis | Must | Define a recomendação que pode ser adotada. 🟢 |
| Saída textual | Should | É o canal padrão de consumo da CLI. 🟢 |
| JSON e arquivo Markdown | Should | São integrações úteis, derivadas do conjunto já detectado. 🟢 |

## Rastreabilidade de Código

| Arquivo | Cobertura | Confiança |
|---|---|---|
| `src/learn/mod.rs` | Filtros, escolha de formato e gatilho de escrita. | 🟢 |
| `src/learn/report.rs` | Relatório e arquivo Markdown. | 🟢 |

## Lacunas

- 🔴 Não há validação estática de como regras geradas são consumidas posteriormente pelo Claude Code.
