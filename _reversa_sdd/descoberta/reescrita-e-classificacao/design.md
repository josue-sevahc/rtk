# Reescrita e Classificação, Design Técnico

## Interface

| Símbolo | Retorno | Papel |
|---|---|---|
| `classify_command` | `Classification` | Normaliza e escolhe regra. 🟢 |
| `rewrite_command` | `Option<String>` | Coordena exclusões, shell e rewrite composto. 🟢 |
| `split_command_chain` | segmentos | Separa para o scan histórico. 🟢 |
| `split_for_permissions` | segmentos | Separa para avaliação de permissões. 🟢 |
| `contains_unattestable_construct` | booleano | Detecta sintaxe que não deve ser autoautorizada. 🟢 |

## Fluxo de Reescrita

1. Colapsar continuação de linha e recusar heredoc ou aritmética shell. 🟢
2. Aplicar exclusões configuradas e verificar `RTK_DISABLED`. 🟢
3. Dividir cadeias por operadores e recursivamente separar prefixos transparentes. 🟢
4. Remover redirect final, procurar regra mais específica e aplicar o prefixo RTK correspondente. 🟢
5. Reaplicar redirect elegível; se nenhum segmento mudou, retornar `None`. 🟢

## Dados e Decisões

- `RtkRule` contém regex, comando RTK, prefixos, categoria, economia padrão e exceções por subcomando/status. 🟢 `rules.rs`
- `RegexSet` reduz a seleção inicial e o último índice de match resolve a especificidade do catálogo. 🟢 `registry.rs`
- O lexer mantém aspas e escapes como parte do argumento, evitando usar `split_whitespace` como parser de shell. 🟢 `lexer.rs`
- Reescrita por filtro TOML é fallback após regras internas e depende da camada de confiança externa. 🟢 `registry.rs`, `core/toml_filter.rs`

## Riscos

- 🔴 A regra de manter `find` e `fd` crus antes de pipe depende de compatibilidade de consumidores como `xargs`; ampliar essa lista requer testes de integração.
- 🟡 O limite de 10 prefixos parece ser proteção contra recursão ou entrada patológica. `registry.rs`
