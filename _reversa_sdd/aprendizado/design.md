# Aprendizado, Design Técnico

> Design reconstruído de `src/learn/`. 🟢 confirmado no código; 🟡 inferido; 🔴 lacuna.

## Interface

| Símbolo | Entrada | Saída | Observação |
|---|---|---|---|
| `learn::run` | projeto, escopo, dias, formato, escrita, limiares | `Result<()>` | Orquestra `rtk learn`. 🟢 |
| `is_command_error` | flag de erro, saída | `bool` | Separa erro real de sinal insuficiente ou rejeição humana. 🟢 |
| `classify_error` | saída | `ErrorType` | Aplica catálogo de regex e fallback. 🟢 |
| `find_corrections` | `&[CommandExecution]` | `Vec<CorrectionPair>` | Busca pares em janela fixa. 🟢 |
| `deduplicate_corrections` | pares | `Vec<CorrectionRule>` | Agrupa e ordena recomendações. 🟢 |

## Fluxo Principal

1. A CLI encaminha as flags de `Learn` para `learn::run`. 🟢 `src/main.rs`
2. A unit resolve o filtro de projeto: todos, explícito ou diretório corrente codificado. 🟢 `src/learn/mod.rs`
3. `ClaudeProvider` localiza sessões dentro do período; sem sessões, a execução encerra com mensagem. 🟢 `src/learn/mod.rs`, `src/discover/provider.rs`
4. Cada sessão fornece comandos; falhas individuais são ignoradas, e somente comandos com `output_content` entram na análise. 🟢 `src/learn/mod.rs`
5. O detector encontra pares erro-correção, aplica o limiar de confiança externo, deduplica e filtra pela recorrência. 🟢 `src/learn/mod.rs`, `src/learn/detector.rs`
6. O resultado é serializado como JSON ou formatado para terminal; a escrita local ocorre somente no caminho textual com regras existentes. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Fluxos Alternativos

- **Sessões vazias:** imprime mensagem específica e retorna `Ok(())`. 🟢 `src/learn/mod.rs`
- **Sessão malformada:** ignora a sessão e segue com as demais. 🟢 `src/learn/mod.rs`
- **Sem pares elegíveis:** informa que não houve correções e retorna com sucesso. 🟢 `src/learn/mod.rs`
- **Formato diferente de `json`:** usa relatório textual. 🟢 `src/learn/mod.rs`
- **Erro ao escrever regras:** é propagado por `anyhow::Result`. 🟢 `src/learn/report.rs`

## Estruturas e Algoritmos

- `CommandExecution` contém comando, flag de erro e saída; é a entrada mínima para a detecção. 🟢 `src/learn/detector.rs`
- `ErrorType` reconhece flag desconhecida, comando ausente, sintaxe, caminho, argumento ausente, permissão ou categoria genérica. 🟢 `src/learn/detector.rs`
- A similaridade exige o mesmo comando-base e soma `0.5` pela base a uma similaridade Jaccard dos argumentos. 🟢 `src/learn/detector.rs`
- A busca olha no máximo três comandos posteriores; candidato sem erro recebe bônus de `0.2`, limitado a `1.0`. 🟢 `src/learn/detector.rs`
- Deduplicação usa `(base_command, error_type, diff_token)`, conserva o exemplo mais confiante e ordena por ocorrências decrescentes. 🟢 `src/learn/detector.rs`

## Dependências

- `discover::provider::{ClaudeProvider, SessionProvider}` para sessões e comandos extraídos. 🟢 `src/learn/mod.rs`
- `regex` e `lazy_static` para reconhecimento de mensagens de erro. 🟢 `src/learn/detector.rs`
- `serde_json` para o formato de saída estruturado. 🟢 `src/learn/mod.rs`
- `std::fs` e `std::path` para persistência opcional das recomendações. 🟢 `src/learn/report.rs`

## Decisões de Design Identificadas

| Decisão | Evidência | Confiança |
|---|---|---|
| Reutilizar o provider de descoberta evita duplicar o parsing de JSONL. | `src/learn/mod.rs`, `src/discover/provider.rs` | 🟢 |
| Tratar heurísticas de correção como recomendações, não como alteração automática de comando. | `src/learn/detector.rs`, `src/learn/report.rs` | 🟢 |
| Remover ciclos TDD e explorações de caminho reduz falsos positivos. | `src/learn/detector.rs` | 🟢 |
| Não ordenar explicitamente comandos entre arquivos pode afetar a sequência global. | `src/learn/mod.rs` | 🟡 |

## Estado Interno e Observabilidade

- A execução acumula `Vec<CommandExecution>`, pares filtrados e regras apenas em memória. 🟢 `src/learn/mod.rs`
- O JSON expõe `sessions_scanned`, `total_corrections` e campos de cada regra; o texto informa regras, correções, sessões e dias. 🟢 `src/learn/mod.rs`, `src/learn/report.rs`

## Riscos e Lacunas

- 🔴 As regexes e o limiar interno de confiança não foram calibrados com dados de produção.
- 🔴 A dupla filtragem por confiança interna e por flag CLI precisa de validação de comportamento esperado em casos limítrofes.
