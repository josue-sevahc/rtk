# Parser de Saida

> Contrato operacional reconstruido de `src/parser/`. Afirmacoes 🟢 foram confirmadas no codigo legado; 🟡 sao inferencias; 🔴 exigem validacao humana.

## Visao Geral

Esta unit oferece uma interface comum para transformar saidas brutas de ferramentas em dados canonicos, sem ocultar falhas de interpretacao. 🟢 `src/parser/mod.rs` Ela usa tres niveis de resultado: parse completo, parse degradado com avisos e passthrough truncado. 🟢 `src/parser/mod.rs`

## Responsabilidades

- Definir `OutputParser` e `ParseResult<T>` como contrato generico para parsers de ferramentas. 🟢 `src/parser/mod.rs`
- Preservar o nivel de degradacao ao consultar, mapear ou limitar o resultado. 🟢 `src/parser/mod.rs`
- Limitar com seguranca a saida que nao pode ser interpretada e identificar o truncamento. 🟢 `src/parser/mod.rs`
- Extrair um objeto JSON completo de saida com prefixos textuais conhecidos. 🟢 `src/parser/mod.rs`
- Expor dados de testes e dependencias que a subunit de formatacao converte em texto. 🟢 `src/parser/types.rs`, `src/parser/formatter.rs`

## Regras de Negocio

- Um resultado `Full` representa tier 1; `Degraded` representa tier 2 e carrega avisos; `Passthrough` representa tier 3 e carrega texto bruto. 🟢 `src/parser/mod.rs`
- `is_ok` aceita somente os tiers 1 e 2; `Passthrough` nao e um parse bem-sucedido. 🟢 `src/parser/mod.rs`
- `map` transforma apenas o dado dos tiers 1 e 2, preservando o tier e os avisos; o passthrough permanece intacto. 🟢 `src/parser/mod.rs`
- `parse_with_tier` devolve passthrough truncado quando o resultado do parser excede o tier maximo solicitado. 🟢 `src/parser/mod.rs`
- O limite de passthrough vem de `limits().passthrough_max_chars`; o padrao configurado e 2000 caracteres. 🟢 `src/parser/mod.rs`, `src/core/config.rs`
- A extracao JSON prioriza `"numTotalTests"`; sem esse marcador, procura uma linha cujo conteudo inicia com `{` e fecha o objeto por balanceamento de chaves, respeitando strings e escapes. 🟢 `src/parser/mod.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de aceite |
|---|---|---|---|
| RF-01 | Expor `OutputParser::parse` com tipo associado de saida e retorno `ParseResult`. 🟢 | Must | Um parser de ferramenta pode retornar `Full`, `Degraded` ou `Passthrough`. |
| RF-02 | Oferecer operacoes de consulta e transformacao que preservam a semantica do tier. 🟢 | Must | `tier`, `is_ok`, `warnings` e `map` retornam os valores esperados para os tres casos. |
| RF-03 | Forcar passthrough quando o tier permitido for menor que o resultado produzido. 🟢 | Should | `parse_with_tier` retorna texto truncado para resultado acima do limite. |
| RF-04 | Truncar saidas sem parse por quantidade de caracteres, mantendo UTF-8 valido e marcador explicito. 🟢 | Must | Texto maior que o limite termina com `[RTK:PASSTHROUGH]` e as contagens. |
| RF-05 | Extrair JSON completo de saida com banners ou mensagens antes do objeto. 🟢 | Should | Prefixos pnpm, dotenv e texto multibyte nao impedem encontrar o objeto JSON. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|---|---|---|---|
| Resiliencia | Falhas de parse degradam para texto limitado em vez de fabricar dados estruturados. | `src/parser/mod.rs` | 🟢 |
| Compatibilidade | Truncamento e balanceamento operam por caracteres/offsets UTF-8 e sao testados com texto Thai, CJK e emoji. | `src/parser/mod.rs` | 🟢 |
| Configurabilidade | O teto do passthrough e obtido da configuracao carregada em tempo de execucao. | `src/parser/mod.rs`, `src/core/config.rs` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Degradar uma saida que nao pode ser parseada
  Dado uma saida sem estrutura reconhecivel maior que o limite configurado
  Quando um parser a retorna como passthrough
  Entao a saida e truncada sem corromper caracteres UTF-8
  E contem o marcador [RTK:PASSTHROUGH]

Cenario: Extrair JSON depois de um banner
  Dado uma saida de teste com mensagens anteriores e um objeto JSON valido
  Quando extract_json_object for chamado
  Entao ele retorna apenas o objeto JSON completo

Cenario: Restringir o tier aceito
  Dado um parser cujo resultado e Degraded
  Quando parse_with_tier receber max_tier igual a 1
  Entao o retorno e Passthrough truncado
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Contrato de tiers e fallback | Must | Protege os consumidores contra dados falsos silenciosos. 🟢 |
| Truncamento seguro | Must | Limita a saida no caminho de falha. 🟢 |
| Extracao de JSON embutido | Should | Sustenta parsers que recebem banners de ferramentas. 🟢 |
| Restricao explicita de tier | Should | Serve a testes e diagnosticos, sem ser o caminho principal. 🟢 |

## Rastreabilidade de Codigo

| Arquivo | Cobertura | Confianca |
|---|---|---|
| `src/parser/mod.rs` | Resultado, trait, truncamento, avisos e extracao JSON. | 🟢 |
| `src/parser/types.rs` | Tipos canonicos de testes e dependencias. | 🟢 |
| `src/core/config.rs` | Valor configuravel do limite de passthrough. | 🟢 |
| `src/cmds/js/vitest_cmd.rs` | Consumidor de tiers e JSON extraido. | 🟢 |
| `src/cmds/js/playwright_cmd.rs`, `src/cmds/js/pnpm_cmd.rs` | Consumidores concretos do contrato de parser. | 🟢 |

## Lacunas

- 🔴 O README descreve tipos canonicos adicionais e uma migracao mais ampla, mas `src/parser/types.rs` implementa somente resultados de teste e estado de dependencias nesta revisao.
