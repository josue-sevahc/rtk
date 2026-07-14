# ADR 004 - Integridade de hook e gate operacional

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: `src/hooks/integrity.rs`, historico de `fix(security)`

## Contexto

O hook legado pode produzir decisoes que autoaprovam comandos reescritos. Alteracao externa dele cria vetor de injecao de comandos.

## Decisao

Persistir SHA-256 no momento da instalacao e verificar antes de comandos operacionais. Divergencia bloqueia execucao e nao tem bypass por ambiente. Ausencia de baseline permanece tolerada por compatibilidade; script ausente torna a verificacao inaplicavel no modelo de hook binario nativo.

## Consequencias

Hooks legados adulterados interrompem o uso ate reinstalacao. Instalacoes antigas nao sao quebradas de imediato, ao custo de uma janela sem atestacao.
