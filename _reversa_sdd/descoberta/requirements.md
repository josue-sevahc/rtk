# Descoberta

> Contrato operacional reconstruído de `src/discover/`. Afirmações 🟢 foram confirmadas no código legado; 🟡 são inferências; 🔴 exigem validação humana.

## Visão Geral

Esta unit sustenta `rtk discover` e o registro compartilhado de reescrita consumido pelos hooks. Ela lê sessões Claude Code, identifica comandos que o RTK poderia otimizar e apresenta oportunidades; no caminho online, classifica e reescreve comandos shell sem assumir a autorização do host. 🟢 `src/discover/mod.rs`, `src/discover/README.md`

## Responsabilidades

- Descobrir arquivos JSONL de sessões Claude, extrair invocações Bash e associar seus resultados. 🟢 `src/discover/provider.rs`
- Dividir cadeias shell, classificar cada segmento como suportado, não suportado ou ignorado e estimar economia. 🟢 `src/discover/mod.rs`, `src/discover/registry.rs`
- Renderizar relatório em terminal ou JSON, incluindo integrações de agentes e bypasses `RTK_DISABLED`. 🟢 `src/discover/report.rs`
- Reescrever comandos conhecidos ou filtros TOML confiáveis, preservando construções e formatos não seguros. 🟢 `src/discover/registry.rs`
- Fornecer tokenização e divisão shell-aware para o registro e para as permissões de hooks. 🟢 `src/discover/lexer.rs`

## Regras de Negócio

- O escopo padrão de `rtk discover` é o diretório corrente codificado como projeto Claude; `--all` remove esse filtro. 🟢 `src/discover/mod.rs`
- `RTK_DISABLED=` só é contado como bypass quando o comando subjacente seria suportado. 🟢 `src/discover/mod.rs`
- O relatório agrega comandos suportados pelo equivalente RTK e não suportados pelo comando-base; ordena respectivamente por economia e frequência. 🟢 `src/discover/mod.rs`
- O estimate usa tamanho real de `tool_result / 4` quando presente, ou média estática por categoria/subcomando. 🟢 `src/discover/mod.rs`, `src/discover/registry.rs`
- Reescrita não altera heredoc, aritmética shell, substituições, redirecionamentos com alvo de arquivo, saída estruturada de `gh` ou segmentos incompatíveis em pipe. 🟢 `src/discover/registry.rs`, `src/discover/lexer.rs`
- A decisão de permitir ou confirmar rewrite permanece no host; esta unit apenas fornece classificação e proposta. 🟢 `src/hooks/rewrite_cmd.rs`, ADR 003

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Localizar sessões Claude por projeto e janela temporal. 🟢 | Must | Com `--all` e `--since`, somente JSONL elegíveis são varridos. |
| RF-02 | Extrair comandos Bash e resultado associado sem abortar o scan por linha inválida. 🟢 | Must | JSONL malformado é ignorado e o restante da sessão continua processado. |
| RF-03 | Classificar e agregar oportunidades de economia, comandos ignorados e não suportados. 🟢 | Must | O relatório contém contagens e agrupamentos ordenados conforme o contrato. |
| RF-04 | Expor relatório textual e JSON. 🟢 | Must | `--format json` serializa o relatório; o padrão mostra tabela legível. |
| RF-05 | Reescrever somente segmentos semanticamente suportados e devolver passthrough quando não houver mudança segura. 🟢 | Must | Construção não atestável ou regra ausente não gera comando RTK. |
| RF-06 | Informar integrações Cursor, Hermes e Copilot, distinguindo-as do scan de histórico Claude. 🟢 | Should | Quando detectadas, notas direcionam a medição para `rtk gain`. |

## Requisitos Não Funcionais

| Tipo | Requisito | Evidência | Confiança |
|---|---|---|---|
| Segurança | Parsing parcial deve falhar para passthrough, sem ampliar permissões. | `lexer.rs`, `registry.rs`, ADR 003 | 🟢 |
| Resiliência | Falha de leitura de uma sessão incrementa diagnóstico e não encerra o relatório inteiro. | `mod.rs` | 🟢 |
| Compatibilidade | JSONL e comandos shell são tratados por heurísticas, não por parser Bash completo. | `provider.rs`, `lexer.rs` | 🟢 |
| Observabilidade | O relatório mostra parse errors em verbose e bypasses RTK quando encontrados. | `report.rs`, `mod.rs` | 🟢 |

## Critérios de Aceitação

```gherkin
Cenário: Encontrar oportunidades em sessões Claude
  Dado uma sessão JSONL com comando Bash suportado e resultado associado
  Quando `rtk discover` executar para o projeto
  Então o equivalente RTK aparece em oportunidades perdidas
  E a economia estimada é acumulada no bucket correspondente

Cenário: Preservar shell não atestável
  Dado um comando com substituição de processo ou redirecionamento para arquivo
  Quando o registry avaliar a reescrita
  Então não é gerado rewrite automático
  E o host pode preservar o comando original
```

## Rastreabilidade de Código

| Arquivo | Cobertura | Confiança |
|---|---|---|
| `src/discover/mod.rs` | Orquestração do scan e agregação. | 🟢 |
| `src/discover/provider.rs` | Sessões Claude e extração JSONL. | 🟢 |
| `src/discover/registry.rs` | Classificação, normalização e rewrite. | 🟢 |
| `src/discover/lexer.rs` | Tokens, operadores e limites de segurança shell. | 🟢 |
| `src/discover/report.rs` | Modelo e renderização do relatório. | 🟢 |
| `src/discover/rules.rs` | Catálogo estático de regras. | 🟢 |

## Lacunas

- 🔴 O scan real de históricos extensos e corrompidos não foi executado nesta análise.
- 🔴 Percentuais em `rules.rs` são heurísticas estáticas e requerem validação empírica.
- 🔴 O lexer não pretende cobrir toda a gramática Bash.
