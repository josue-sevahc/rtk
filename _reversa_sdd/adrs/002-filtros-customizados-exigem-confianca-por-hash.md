# ADR 002 - Filtros customizados exigem confianca por hash

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: commits `1130a7c`, `9cc4937`, `2d487cb`; `src/hooks/trust.rs`

## Contexto

Um `.rtk/filters.toml` presente em repositorio pode suprimir ou reescrever output que chega ao agente, inclusive resultados de scanners de seguranca.

## Decisao

Carregar filtros customizados somente apos aprovacao explicita. Guardar SHA-256 por caminho canonico; mudanca de conteudo exige nova aprovacao. Falhas de leitura ou validacao resultam em nao confiavel. O bypass por ambiente vale apenas em CI detectado.

## Consequencias

O usuario revisa a capacidade de transformacao antes de habilita-la. Pipelines podem optar por confianca explicita, mas o caminho interativo mantem postura fail-secure.
