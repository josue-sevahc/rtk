# Análise de Histórico

> Subunit de `descoberta` reconstruída de `src/discover/provider.rs`, `mod.rs` e `report.rs`.

## Visão Geral

Esta subunit examina o histórico de sessões Claude Code para revelar comandos que já possuem equivalente RTK e oportunidades ainda não cobertas. 🟢 O scan não mede diretamente Cursor, Hermes ou Copilot: quando seus artefatos são detectados, o relatório direciona a análise para `rtk gain`. 🟢 `src/discover/report.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de aceite |
|---|---|---|---|
| RF-01 | Resolver o diretório Claude e localizar JSONL sem seguir symlinks. 🟢 | Must | Diretório ausente retorna coleção vazia; caminho inválido retorna erro contextualizado. |
| RF-02 | Filtrar sessões por projeto codificado e data de modificação. 🟢 | Must | `--project`, diretório corrente, `--all` e `--since` alteram apenas a seleção esperada. |
| RF-03 | Extrair `tool_use` Bash e associar `tool_result` por id, preservando índice na sessão. 🟢 | Must | Comando sem resultado ainda aparece com campos opcionais vazios. |
| RF-04 | Continuar o scan quando uma sessão falhar e registrar a contagem de falhas. 🟢 | Must | Uma sessão ilegível não impede o relatório das demais. |
| RF-05 | Exibir oportunidades suportadas, não suportadas, bypasses e adoção já RTK em texto ou JSON. 🟢 | Must | A resposta mantém os campos de `DiscoverReport`. |

## Regras de Negócio

- Linhas JSONL são pré-filtradas para `Bash` ou `tool_result` antes do parse estrutural. 🟢 `provider.rs`
- Preview de resultado é limitado a cerca de 1000 caracteres; tamanho completo alimenta a estimativa. 🟢 `provider.rs`
- Comandos que já começam com `rtk ` incrementam adoção, mas comandos ignorados de outra natureza não. 🟢 `mod.rs`
- Erros de parsing são mostrados no texto somente em verbose. 🟢 `report.rs`

## Critérios de Aceitação

```gherkin
Cenário: Sessão parcialmente inválida
  Dado um JSONL com uma linha malformada e uma invocação Bash válida
  Quando o provider extrair comandos
  Então a invocação válida permanece disponível
  E a análise não falha por causa da linha malformada

Cenário: Projeto padrão
  Dado nenhum `--project` e nenhum `--all`
  Quando `rtk discover` executar
  Então o diretório corrente é codificado para selecionar sessões Claude do projeto
```

## Lacunas

- 🔴 O formato real de JSONL de versões futuras do Claude Code pode divergir da heurística atual.
- 🔴 Não há evidência de benchmark para volumes muito altos de sessões.
