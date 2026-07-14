# ADR 006 - Fallback distingue meta-comandos de ferramentas externas

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: commit `9cc4937`, `src/main.rs`

## Contexto

O CLI usa fallback para interceptar ferramentas externas sem exigir que o usuario memorize todos os subcomandos RTK. O mesmo fallback seria perigoso para uso invalido de meta-comandos do proprio RTK.

## Decisao

Quando o primeiro token e meta-comando RTK, erro de parse permanece erro de CLI. Apenas comandos externos desconhecidos podem seguir para filtro TOML ou passthrough.

## Consequencias

Erros de sintaxe em comandos RTK nao executam inesperadamente binarios externos. A compatibilidade com ferramentas externas permanece.
