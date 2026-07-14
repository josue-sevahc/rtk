# ADR 001 - Saida filtrada nunca piora a saida bruta

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: commit `861a46d` (`2026-06-23`), `src/core/guard.rs`, `src/core/runner.rs`

## Contexto

Filtros, cabecalhos e hints podiam produzir mais tokens que a saida original, contrariando a proposta de economia e transparencia.

## Decisao

Comparar a estimativa de tokens da saida composta com a saida bruta. Quando a filtrada for maior, emitir a bruta. A mesma representacao emitida deve ser a rastreada; saida bruta vazia nao recebe mensagem sintetica.

## Consequencias

O limite de economia deixa de ser apenas meta de filtro e passa a ser invariante transversal. Filtros podem perder mensagens cosmeticas, mas preservam fidelidade e exit code.
