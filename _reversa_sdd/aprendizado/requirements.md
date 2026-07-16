# Aprendizado

> Contrato operacional reconstruído de `src/learn/`. Afirmações 🟢 foram confirmadas no código legado; 🟡 são inferências; 🔴 exigem validação humana.

## Visão Geral

Esta unit implementa `rtk learn`: analisa o histórico de sessões Claude Code para reconhecer erros de CLI que foram seguidos por uma correção provável. 🟢 `src/learn/mod.rs` Ela transforma pares observados em regras consolidadas, sem executar ou alterar os comandos originais. 🟢 `src/learn/detector.rs`

## Responsabilidades

- Localizar sessões Claude no projeto atual, em projeto informado ou em todos os projetos. 🟢 `src/learn/mod.rs`, `src/discover/provider.rs`
- Extrair apenas comandos Bash com conteúdo de saída e ignorar sessões que não possam ser lidas. 🟢 `src/learn/mod.rs`
- Distinguir erro de CLI de cancelamento humano, exploração de caminho e ciclo TDD. 🟢 `src/learn/detector.rs`
- Encontrar correções próximas, calcular confiança e deduplicar os pares em regras recorrentes. 🟢 `src/learn/detector.rs`
- Entregar recomendações em texto ou JSON e delegar a escrita de regras locais quando solicitada. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Regras de Negócio

- Sem `--all`, o escopo padrão é o diretório corrente codificado como identificador de projeto Claude; `--project` substitui esse filtro. 🟢 `src/learn/mod.rs`
- A ausência de sessões ou de correções não é erro: o comando informa o resultado e termina com sucesso. 🟢 `src/learn/mod.rs`
- Um erro elegível exige `is_error=true`, conteúdo indicativo de falha e não pode representar rejeição ou cancelamento pelo usuário. 🟢 `src/learn/detector.rs`
- A correção candidata deve estar entre os três comandos subsequentes, compartilhar a base e alcançar confiança mínima interna de `0.6`. 🟢 `src/learn/detector.rs`
- Mudanças que parecem apenas troca de caminho e comandos idênticos repetidos não formam recomendação. 🟢 `src/learn/detector.rs`
- Regras são agrupadas por comando-base, tipo de erro e token diferencial; o exemplo de maior confiança é preservado. 🟢 `src/learn/detector.rs`
- A ordem cronológica entre sessões diferentes não é explicitamente reordenada antes da detecção. 🟡 `src/learn/mod.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Resolver o escopo de sessões a partir de `--all`, `--project` ou diretório corrente. 🟢 | Must | Cada combinação seleciona o filtro esperado e respeita `--since`. |
| RF-02 | Extrair comandos com saída e continuar quando uma sessão estiver malformada. 🟢 | Must | Uma sessão ilegível não impede a análise das demais. |
| RF-03 | Detectar pares erro-correção por janela, similaridade e confiança. 🟢 | Must | Um erro seguido de correção compatível em até três comandos produz um par. |
| RF-04 | Excluir cancelamentos humanos, erros de compilação/teste, explorações de caminho e repetições idênticas. 🟢 | Must | Esses padrões não aparecem nas recomendações. |
| RF-05 | Deduplicar pares e aplicar `--min-confidence` e `--min-occurrences`. 🟢 | Must | O relatório contém somente regras que atendem aos limiares informados. |
| RF-06 | Exibir recomendações em texto ou JSON e permitir a geração opcional do arquivo de regras. 🟢 | Should | `--format json` retorna o objeto esperado; texto pode solicitar a escrita local. |

## Requisitos Não Funcionais

| Tipo | Requisito inferido | Evidência no código | Confiança |
|---|---|---|---|
| Resiliência | Falhas de extração por sessão são isoladas e não interrompem a análise global. | `src/learn/mod.rs` | 🟢 |
| Compatibilidade | A classificação é heurística baseada em regex e similaridade textual, não em semântica completa de shell. | `src/learn/detector.rs` | 🟢 |
| Auditabilidade | A saída JSON inclui contagem de sessões, correções e campos de cada regra. | `src/learn/mod.rs` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: Detectar uma correção de CLI
  Dado uma sessão com `git commit --ammend` marcado como erro
  E `git commit --amend` como comando subsequente bem-sucedido
  Quando `rtk learn` analisar a sessão
  Então uma recomendação de correção é incluída no relatório

Cenário: Ignorar uma tentativa que não é correção de CLI
  Dado um comando com saída de erro de compilação ou cancelamento do usuário
  Quando `rtk learn` analisar a sessão
  Então o comando não gera recomendação
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|---|---|---|
| Descobrir, extrair e detectar pares de correção | Must | É o caminho crítico de `rtk learn`. 🟢 |
| Filtrar falsos positivos | Must | Evita transformar atividade normal em regra indevida. 🟢 |
| Consolidar e filtrar regras | Must | Define o contrato útil do relatório. 🟢 |
| Formato JSON e escrita local de regras | Should | São saídas importantes, mas dependem do núcleo de detecção. 🟢 |

## Rastreabilidade de Código

| Arquivo | Cobertura | Confiança |
|---|---|---|
| `src/learn/mod.rs` | Orquestra escopo, extração, filtros e saída. | 🟢 |
| `src/learn/detector.rs` | Classificação, similaridade, detecção e deduplicação. | 🟢 |
| `src/learn/report.rs` | Relatório textual e arquivo de recomendações. | 🟢 |
| `src/discover/provider.rs` | Descoberta de sessões e extração de comandos. | 🟢 |

## Lacunas

- 🔴 A qualidade dos limiares e das heurísticas não foi validada sobre histórico real em volume.
- 🔴 Não há evidência estática de ordenação cronológica global entre arquivos de sessão.
