# ADR 003 - Permissoes do host prevalecem sobre rewrite

Status: Aceita (reconstruida)  
Confianca: 🟢 CONFIRMADO  
Evidencia: commits `40c9dbc`, `952245d`, `3d40742`; `src/hooks/permissions.rs`, `src/hooks/hook_cmd.rs`

## Contexto

Reescrever `git status` para `rtk git status` pode alterar o alvo dos padroes de permissao do agente. Em comandos compostos, um allow parcial podia elevar uma cadeia inteira.

## Decisao

Aplicar precedencia `Deny > Ask > Allow > Default`; default pede confirmacao. Exigir allow em todos os segmentos da cadeia e deferir construtos nao atestaveis. Para Droid, nao replicar allowlists: deny/block explicitos fazem RTK preservar o comando original e os demais rewrites nao carregam decisao de permissao.

## Consequencias

O RTK aceita menos autoaprovações em troca de nao ampliar privilegios do host. Adaptadores podem ter semanticas distintas, mas nao devem elevar a decisao original.
