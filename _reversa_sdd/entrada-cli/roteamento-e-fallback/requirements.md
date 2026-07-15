# Roteamento e Fallback

> Contrato operacional do caso de uso que classifica uma invocacao nao parseada pelo RTK e a encaminha de forma segura. As afirmacoes usam 🟢 para comportamento confirmado no codigo, 🟡 para inferencia sustentada por evidencias e 🔴 para lacuna de validacao.

## Visao Geral

🟢 Este caso de uso recebe uma falha comum do parser e decide se ela representa uso invalido de um comando interno ou uma ferramenta externa que o RTK pode filtrar ou delegar. Ele preserva o erro do parser para a superficie propria do RTK e mantém a compatibilidade com ferramentas externas via filtro TOML ou passthrough.

🟢 Quando filtra, o fluxo conserva o exit code e impede que a representacao emitida tenha mais tokens estimados que a saida bruta.

## Responsabilidades

- 🟢 Distinguir erros de uso de meta-comandos RTK de invocacoes de ferramentas externas desconhecidas pela CLI declarada.
- 🟢 Normalizar a ferramenta externa pelo basename para localizar filtros TOML mesmo quando ela for invocada por caminho absoluto.
- 🟢 Executar e filtrar a saida conforme o contrato do filtro, fornecendo recuperacao quando a transformacao for lossy.
- 🟢 Executar passthrough com streams herdados quando o filtro estiver desabilitado ou nao houver correspondencia.
- 🟢 Registrar tracking e falhas de parse sem duplicar a mensagem do Clap, preservando o codigo de saida observavel.

## Regras de Negocio

- 🟢 Sem argumentos apos o nome do binario, o erro do Clap encerra o processo e nenhum fallback e iniciado.
- 🟢 O primeiro token pertencente a `RTK_META_COMMANDS` sempre preserva o erro de parse; ele nunca e resolvido como executavel externo.
- 🟢 A lista de meta-comandos inclui superficies como `gain`, `discover`, `learn`, `init`, `config`, `proxy`, `run`, `hook`, `verify`, `trust`, `untrust`, `session`, `rewrite` e `telemetry`.
- 🟢 `RTK_NO_TOML` desabilita a busca de filtro e força o caminho passthrough.
- 🟢 Um filtro TOML pode solicitar que stderr seja capturado e mesclado a stdout antes da filtragem; sem essa opcao, stderr permanece visivel diretamente.
- 🟢 Quando ha perda de informacao, o fluxo tenta produzir hint de recuperacao; sem hint recuperavel, mostra a saida bruta inteira.
- 🟢 A saida composta de filtro e hint passa por `never_worse`, portanto nao pode custar mais tokens estimados que a referencia bruta.
- 🟢 Falha ao iniciar a ferramenta externa resulta em uma mensagem `[rtk: <erro>]`, registro de parse failure com falha e codigo `127`.

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|-----------|-------------------|
| RF-01 | 🟢 O caso de uso deve receber uma falha comum do parser e obter os argumentos crus do processo para classificacao. | Must | Dado um erro de parse sem ajuda ou versao, quando ha argumentos, entao o fluxo usa os argumentos apos o nome do binario para decidir o destino. |
| RF-02 | 🟢 O caso de uso deve falhar fechado para tokens da lista `RTK_META_COMMANDS`. | Must | Dado `rtk gain --flag-invalida`, quando o parser falha, entao o erro do parser e exibido/encerrado e nenhum processo `gain` e iniciado pelo sistema. |
| RF-03 | 🟢 O caso de uso deve permitir que token externo nao classificado siga para filtro TOML ou passthrough. | Must | Dado `rtk ferramenta-externa arg`, quando o parser falha e o token nao pertence aos meta-comandos, entao a ferramenta e considerada para execucao externa. |
| RF-04 | 🟢 A busca de filtro deve usar o basename do executavel e todos os argumentos, salvo quando `RTK_NO_TOML` estiver ativo. | Must | Dado `/usr/bin/make teste`, quando existe filtro para `make`, entao o filtro e localizado; dado `RTK_NO_TOML=1`, entao o mesmo comando segue sem busca de filtro. |
| RF-05 | 🟢 Com filtro localizado, o caso de uso deve capturar a saida definida pelo filtro, aplicar a transformacao e devolver o exit code real do processo filho. | Must | Dado filtro elegivel e filho que termina com codigo diferente de zero, quando o fluxo conclui, entao a saida segue as regras de filtro/recuperacao e o codigo devolvido coincide com o do filho. |
| RF-06 | 🟢 Uma filtragem com perda deve manter um caminho de recuperacao; se nao puder haver hint, deve emitir a referencia bruta. | Must | Dado `Lossiness::Tail` ou `Lossiness::Whole`, quando o hint e gerado, entao a saida inclui o mecanismo de recuperacao; dado hint indisponivel, entao a saida exibida e a bruta. |
| RF-07 | 🟢 Sem filtro ou com TOML desabilitado, o caso de uso deve executar o filho com stdin, stdout e stderr herdados e registrar passthrough. | Must | Dado um comando sem correspondencia TOML, quando ele e executado, entao os streams sao herdados, o tracking de passthrough ocorre e o exit code e preservado. |
| RF-08 | 🟢 Falhas de spawn devem ser registradas sem duplicar o erro de parse e retornar `127`. | Must | Dado executavel ausente, quando o fluxo tenta iniciá-lo, entao stderr recebe uma unica mensagem `[rtk: ...]`, `record_parse_failure_silent` recebe falha e o resultado e `127`. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Seguranca | 🟢 Sintaxe invalida de meta-comando nao pode provocar execucao de binario homonimo no `PATH`. | `src/main.rs:1259`, `src/core/constants.rs:10` | 🟢 |
| Compatibilidade | 🟢 O exit code externo e parte do contrato e deve ser devolvido tanto no caminho filtrado quanto em passthrough. | `src/main.rs:1299`, `src/main.rs:1380` | 🟢 |
| Fidelidade | 🟢 O output filtrado, inclusive hint, nao pode ultrapassar a saida bruta em tokens estimados. | `src/core/runner.rs:12`, `src/core/guard.rs` | 🟢 |
| Observabilidade | 🟢 Execucao filtrada e passthrough registram tracking; toda tentativa de fallback registra parse failure como sucesso ou falha. | `src/main.rs:1354`, `src/main.rs:1376` | 🟢 |
| Disponibilidade | 🟢 Falha ao criar processo externo e convertida em resposta local `127`, sem depender de novo parse ou servico remoto. | `src/main.rs:1358`, `src/main.rs:1381` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Preservar erro de meta-comando invalido
  Dado uma invocacao invalida cujo primeiro token esta em RTK_META_COMMANDS
  Quando o parser da CLI retorna erro comum
  Entao o erro do parser encerra o processo
  E nenhum executavel externo com o mesmo nome e iniciado

Cenario: Filtrar ferramenta externa invocada por caminho absoluto
  Dado uma ferramenta externa cujo basename possui filtro TOML elegivel
  Quando a CLI nao reconhece a invocacao
  Entao o fallback localiza o filtro pelo basename
  E aplica a filtragem sem alterar o exit code do processo filho
  E registra tracking e a falha de parse de forma silenciosa

Cenario: Recuperar informacao removida por filtro
  Dado uma filtragem que remove parte ou toda a saida
  Quando o processo externo conclui
  Entao o fluxo fornece hint de recuperacao quando possivel
  E quando nao for possivel, emite a saida bruta integral

Cenario: Delegar comando sem filtro
  Dado uma ferramenta externa sem filtro ou RTK_NO_TOML ativo
  Quando o parser da CLI nao reconhece a invocacao
  Entao o filho recebe stdin, stdout e stderr herdados
  E o codigo devolvido coincide com o status do filho

Cenario: Informar executavel externo ausente
  Dado uma ferramenta externa que nao pode ser iniciada
  Quando o fallback tenta executa-la
  Entao a falha e registrada uma vez
  E o resultado e codigo 127
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| 🟢 Bloqueio de fallback para meta-comandos | Must | E a barreira que impede execucao externa inesperada por erro de uso do RTK. |
| 🟢 Filtragem com preservacao de exit code e guard de saida | Must | Mantem a proposta de economia sem ocultar falha ou piorar a saida original. |
| 🟢 Passthrough com streams herdados | Must | Garante compatibilidade com ferramentas que nao possuem filtro. |
| 🟢 Tracking e parse failure silencioso | Should | Sustenta analitica e diagnostico sem alterar o resultado da ferramenta. |
| 🟢 Busca pelo basename | Should | Amplia compatibilidade com ferramentas chamadas por caminho absoluto. |

> 🟡 As prioridades foram inferidas pelo ADR reconstruido, pela posicao do fallback no caminho de erro e pelos invariantes transversais de fidelidade observados.

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `src/main.rs` | `run_fallback` | 🟢 |
| `src/core/constants.rs` | `RTK_META_COMMANDS` | 🟢 |
| `src/core/toml_filter.rs` | `toml_disabled`, `find_matching_filter`, `apply_filter_with_info` | 🟢 |
| `src/core/runner.rs` | `emit_guarded` | 🟢 |
| `src/core/guard.rs` | `never_worse` | 🟢 |
| `src/core/tee.rs` | hints de recuperacao para perda de output | 🟢 |
| `src/core/tracking.rs` | `TimedExecution`, `record_parse_failure_silent` | 🟢 |
| `_reversa_sdd/adrs/006-fallback-distingue-meta-comandos-de-ferramentas-externas.md` | Decisao retroativa de seguranca | 🟢 |
| `_reversa_sdd/adrs/001-saida-filtrada-nunca-piora-a-saida-bruta.md` | Invariante de fidelidade e economia | 🟢 |
