# Formatacao Estruturada

> Subunit de `parser-de-saida` reconstruida de `src/parser/formatter.rs`. Afirmacoes 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 exigem validacao humana.

## Visao Geral

Esta subunit converte resultados canonicos de testes e dependencias em texto orientado a economia de tokens, com modos compacto, detalhado e ultracompacto. 🟢 `src/parser/formatter.rs` A selecao do modo vem do nivel numerico de verbosidade. 🟢 `src/parser/formatter.rs`

## Responsabilidades

- Mapear verbosidade `0`, `1` e maior ou igual a `2` para `Compact`, `Verbose` e `Ultra`. 🟢 `src/parser/formatter.rs`
- Padronizar a interface `TokenFormatter` e delegar o formato escolhido por `format`. 🟢 `src/parser/formatter.rs`
- Exibir resultado de testes sem esconder testes ignorados ou mensagens de falha relevantes. 🟢 `src/parser/formatter.rs`
- Diferenciar uma listagem simples de dependencias de uma consulta de atualizacoes. 🟢 `src/parser/formatter.rs`
- Limitar listagens compactas de dependencias por `CAP_INVENTORY`. 🟢 `src/parser/formatter.rs`, `src/core/truncate.rs`

## Regras de Negocio

- `Compact` e o modo para verbosidade `0`, `Verbose` para `1` e `Ultra` para qualquer valor superior. 🟢 `src/parser/formatter.rs`
- O resumo compacto de testes sempre mostra aprovados e falhos, acrescentando ignorados apenas quando o total e maior que zero. 🟢 `src/parser/formatter.rs`
- O modo compacto exibe no maximo cinco falhas, mas preserva todas as linhas da mensagem de cada falha exibida. 🟢 `src/parser/formatter.rs`
- O modo detalhado mostra todas as falhas, caminho de arquivo e no maximo tres linhas de stack trace quando existente. 🟢 `src/parser/formatter.rs`
- Uma lista de pacotes sem versao mais recente nao deve afirmar falsamente que todos estao atualizados. 🟢 `src/parser/formatter.rs`
- Para dependencias desatualizadas, o modo compacto mostra no maximo dez atualizacoes e indica quantas foram omitidas. 🟢 `src/parser/formatter.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de aceite |
|---|---|---|---|
| RF-01 | Selecionar o modo de formatacao a partir da verbosidade. 🟢 | Must | Valores 0, 1 e 2 retornam Compact, Verbose e Ultra. |
| RF-02 | Formatar resultados de testes nos tres modos previstos. 🟢 | Must | O resumo compacto, detalhes completos e siglas ultra seguem o contrato. |
| RF-03 | Preservar detalhes de falhas relevantes no modo compacto. 🟢 | Must | Diferencas expected/received e call log em mensagens entram na saida. |
| RF-04 | Formatar listagens e atualizacoes de dependencias sem falso positivo de atualizacao. 🟢 | Must | Lista simples exibe pacotes; lista vazia de desatualizados pode informar atualizacao. |
| RF-05 | Aplicar os limites de itens definidos para formatos compactos. 🟢 | Should | Falhas e dependencias alem do teto geram indicador de itens adicionais. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Usabilidade | O modo detalhado inclui contexto de arquivo e stack para falhas. | `src/parser/formatter.rs` | 🟢 |
| Eficiencia de tokens | Os modos compacto e ultra reduzem itens e usam sintaxe curta. | `src/parser/formatter.rs` | 🟢 |
| Corretude | Testes impedem que listagens simples virem falso "up-to-date" e que falhas compactas percam detalhes multiline. | `src/parser/formatter.rs` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Mostrar uma falha de teste compacta
  Dado um resultado de teste com mensagem contendo expected, received e call log
  Quando ele for formatado em Compact
  Entao a saida mostra os contadores
  E preserva todas as linhas da mensagem da falha exibida

Cenario: Listar pacotes sem informacao de atualizacao
  Dado um estado de dependencias sem latest_version e com pacotes
  Quando ele for formatado em Compact
  Entao a saida lista os pacotes
  E nao declara que todos estao atualizados

Cenario: Formatar em modo ultra
  Dado um resultado de testes
  Quando a verbosidade for maior que um
  Entao a saida usa os marcadores curtos de aprovado, falha e ignorado
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Formatar testes e dependencias | Must | E a entrega consumida pelos wrappers especializados. 🟢 |
| Preservar detalhes uteis de erro | Must | Evita esconder dados necessarios para diagnostico. 🟢 |
| Modo ultra | Should | Otimiza a saida quando ha maior verbosidade. 🟢 |
| Limites de itens | Should | Controla tamanho, sem impedir a entrega de resumo. 🟢 |

## Rastreabilidade de Codigo

| Arquivo | Cobertura | Confianca |
|---|---|---|
| `src/parser/formatter.rs` | Modos, trait e formatadores de testes/dependencias. | 🟢 |
| `src/parser/types.rs` | Dados de entrada para os formatadores. | 🟢 |
| `src/core/truncate.rs` | Constante `CAP_INVENTORY` usada no limite de listagem. | 🟢 |
| `src/cmds/js/vitest_cmd.rs`, `src/cmds/js/playwright_cmd.rs` | Selecionam modo e formatam testes. | 🟢 |
| `src/cmds/js/pnpm_cmd.rs` | Formata dependencias e aplica fluxos especializados. | 🟢 |

## Lacunas

- 🔴 Nao ha uma metrica no codigo que comprove a reducao de tokens anunciada no README do modulo.
