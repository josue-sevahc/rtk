# Reescrita e Classificação

> Subunit de `descoberta` reconstruída de `src/discover/registry.rs`, `lexer.rs` e `rules.rs`.

## Visão Geral

Esta subunit converte um segmento shell em `Supported`, `Unsupported` ou `Ignored` e, quando seguro, propõe seu equivalente RTK. 🟢 Ela é compartilhada pelo scan histórico e por `rtk rewrite`, portanto classificação e rewrite precisam permanecer coerentes. 🟢 `src/discover/README.md`, `src/hooks/rewrite_cmd.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Normalizar prefixos env/sudo, binários absolutos e variações Git, Composer e golangci-lint antes do match. 🟢 | Must | Variações equivalentes alcançam a mesma regra. |
| RF-02 | Selecionar a regra mais específica do catálogo e retornar equivalente, categoria, percentual e status. 🟢 | Must | Match múltiplo escolhe a última regra aplicável. |
| RF-03 | Reescrever chains preservando operadores e somente o lado esquerdo de pipes compatíveis. 🟢 | Must | Segmento inseguro ou incompatível permanece cru. |
| RF-04 | Remover e reaplicar redirects finais somente quando isso não altera a segurança. 🟢 | Must | Redirect de arquivo em leitura não vira operação RTK. |
| RF-05 | Aceitar filtros TOML somente quando habilitados e correspondentes, sem reescrever comandos RTK reservados. 🟢 | Should | Filtro ausente, desabilitado ou não correspondente não altera a entrada. |

## Regras de Negócio

- Já iniciar com `rtk` evita rewrite de comando simples; em cadeia, segmentos posteriores ainda são considerados. 🟢 `registry.rs`
- `cat`, `head` e `tail` com redirect ou flags incompatíveis não são promovidos a leitura RTK. 🟢 `registry.rs`
- `head`/`tail` de um arquivo e linha simples mapeiam para limites de `rtk read`; múltiplos arquivos passam adiante. 🟢 `registry.rs`
- `gh` com `--json`, `--jq` ou `--template` não é reescrito para preservar saída estruturada. 🟢 `registry.rs`
- Profundidade de prefixos transparentes é limitada a 10. 🟢 `registry.rs`
- Substituições de comando/processo e redirects com alvo de arquivo são não atestáveis. 🟢 `lexer.rs`

## Critérios de Aceitação

```gherkin
Cenário: Reescrever cadeia suportada
  Dado `git status && rg termo src`
  Quando o registry reescrever a cadeia
  Então cada segmento suportado recebe equivalente RTK
  E o operador `&&` é preservado

Cenário: Preservar saída estruturada do GitHub
  Dado `gh pr list --json title`
  Quando o registry for avaliado
  Então nenhuma proposta de rewrite é produzida
```

## Lacunas

- 🔴 O catálogo é estático e seus percentuais não foram calibrados com telemetria de produção.
- 🔴 Construções Bash fora do lexer parcial devem ser tratadas como incompatíveis até validação explícita.
