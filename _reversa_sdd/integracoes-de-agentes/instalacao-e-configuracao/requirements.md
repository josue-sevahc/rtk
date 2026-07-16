# Instalação e Configuração de Integrações

> Contrato extraído de `src/hooks/init.rs`, `src/hooks/integrity.rs` e `src/hooks/hook_check.rs`. 🟢 confirmado; 🔴 lacuna.

## Visão Geral

Implementa `rtk init` e a remoção de integrações: detecta o agente e escopo, cria ou atualiza seus arquivos de configuração e instala instruções, plugins ou hooks sem destruir conteúdo preexistente. 🟢 `src/hooks/init.rs`

## Responsabilidades

- Instalar artefatos específicos de Claude, Codex, Cursor, Gemini, Droid, Copilot, Pi, Hermes e OpenCode. 🟢 `src/hooks/init.rs`
- Aplicar patches idempotentes, com `dry_run`, confirmação ou auto-patch. 🟢 `src/hooks/init.rs`
- Criar templates locais/globais de filtros TOML quando aplicável. 🟢 `src/hooks/init.rs`
- Registrar e verificar hash SHA-256 de hooks legados; diagnosticar ausente ou desatualizado. 🟢 `src/hooks/integrity.rs`, `src/hooks/hook_check.rs`

## Regras de Negócio

- `write_if_changed` não grava se o conteúdo final for igual ao atual; `dry_run` não grava nada. 🟢 `src/hooks/init.rs`
- Escritas usam arquivo temporário no mesmo diretório seguido de persist/rename. 🟢 `src/hooks/init.rs`
- O hash sidecar segue formato compatível com `sha256sum` e é read-only em Unix como barreira auxiliar. 🟢 `src/hooks/integrity.rs`
- Hook nativo binário registrado em `settings.json` é atual; coexistência com script legado indica estado desatualizado. 🟢 `src/hooks/hook_check.rs`

## Requisitos Funcionais

| ID | Requisito | Prioridade | Critério de Aceite |
|---|---|---|---|
| RF-01 | Instalar/remover a configuração do agente e escopo selecionados. 🟢 | Must | Arquivos esperados existem após init e são removidos somente no uninstall correspondente. |
| RF-02 | Preservar conteúdo não RTK e não duplicar patch já presente. 🟢 | Must | Segunda execução produz `AlreadyPresent` ou nenhuma alteração. |
| RF-03 | Suportar `dry_run`, modo de confirmação e modo automático. 🟢 | Should | Dry-run lista ações sem escrita; modo skip informa a ação manual. |
| RF-04 | Proteger e diagnosticar hook legado por baseline SHA-256 e versão. 🟢 | Must | Hash divergente é identificado; ausência/obsolescência é diagnosticada sem bloquear startup. |

## Critérios de Aceitação

```gherkin
Cenário: Instalação idempotente
Dado uma configuração já contendo o hook RTK
Quando rtk init for executado novamente
Então nenhum hook duplicado deve ser inserido
E o conteúdo não RTK deve permanecer inalterado

Cenário: Simulação segura
Dado rtk init em modo dry-run
Quando a instalação for avaliada
Então as ações planejadas devem ser exibidas
E nenhum arquivo deve ser alterado
```

## Rastreabilidade de Código

| Arquivo | Cobertura |
|---|---|
| `src/hooks/init.rs` | seleção de host, patch, escrita atômica e templates 🟢 |
| `src/hooks/integrity.rs` | baseline e verificação SHA-256 🟢 |
| `src/hooks/hook_check.rs` | status e aviso rate-limited 🟢 |
