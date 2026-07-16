# Relatório de evolução do Writer — plano persistente e avanço limitado por unit

Data da análise: 2026-07-14  
Escopo: instalação do Reversa presente no projeto `rtk`  
Natureza: especificação de mudança para aplicação posterior no repositório-fonte do Reversa

## 1. Resultado da análise

O local primário da alteração é o contrato operacional do Writer:

- `.agents/skills/reversa-writer/SKILL.md`
- `.claude/skills/reversa-writer/SKILL.md` (espelho equivalente)

A mudança também afeta o contrato geral de persistência e o override do modo autônomo:

- `.agents/skills/reversa/references/state-schema.md`
- `.claude/skills/reversa/references/state-schema.md` (espelho equivalente)
- `.agents/skills/reversa-autonomous/SKILL.md`
- `.claude/skills/reversa-autonomous/SKILL.md` (espelho equivalente)

O manifesto de distribuição contém hashes dessas cópias e deverá ser regenerado pelo mecanismo oficial de empacotamento/instalação do Reversa:

- `.reversa/_config/files-manifest.json` em uma instalação gerada; não deve ser editado manualmente como fonte da regra.

Não foi encontrado código executável específico do Writer nesta instalação. O comportamento é definido pelos arquivos `SKILL.md`; por isso, somente alterar o código Rust do projeto hospedeiro não produziria efeito.

## 2. Diagnóstico do comportamento atual

O Writer atualmente:

1. monta e apresenta um plano, mas não exige que a lista integral seja persistida antes da primeira escrita;
2. salva `redator_progress` após cada arquivo, sem schema ou invariantes documentados;
3. informa o próximo item no chat, mas não proíbe `next_file: null` com itens pendentes;
4. define a estrutura híbrida, mas não obriga uma verificação de duplicidade antes de criar cada subunit;
5. avança um arquivo por confirmação, enquanto o modo autônomo substitui genericamente os handoffs por avanço imediato, sem fronteira explícita de unit.

O estado observado nesta instalação confirma a lacuna: `plan_total_files` registra 78, mas não existe um `generation_plan` integral com os 78 itens. Há apenas `completed_files`, `next_file` e um `active_batch` parcial. Portanto, uma retomada não consegue provar qual é a fila completa originalmente aprovada.

## 3. Alterações normativas propostas

### 3.1 Persistência obrigatória antes do primeiro arquivo

No `Passo 1, Montar o plano` do Writer, inserir uma etapa obrigatória entre a resolução do plano e a geração:

> Antes de criar qualquer arquivo de spec, persista atomicamente em `.reversa/state.json#redator_progress.generation_plan` a lista completa, ordenada e aprovada de arquivos de todas as units e dos globais. A persistência deve ocorrer antes da criação de diretórios de unit e antes da escrita do primeiro arquivo. `plan_total_files` deve ser igual ao tamanho de `generation_plan`.

Regras complementares:

- cada item precisa ter, no mínimo, `order`, `path`, `unit`, `kind` e `status`;
- `kind` deve distinguir `canonical`, `optional` e `global`;
- o plano aprovado começa com status `pending`, exceto arquivos preexistentes, que devem ser registrados como `preserved` sem serem sobrescritos;
- qualquer ajuste posterior exige persistir uma nova revisão integral do plano antes de continuar; não é permitido manter apenas um lote parcial em `active_batch`;
- em retomada de uma execução antiga sem `generation_plan`, o Writer deve reconstruir e persistir o plano completo antes de gerar outro arquivo.

Schema de referência:

```json
{
  "redator_progress": {
    "plan_revision": 1,
    "plan_total_files": 6,
    "generation_plan": [
      {
        "order": 1,
        "path": "_reversa_sdd/modulo/requirements.md",
        "unit": "modulo",
        "kind": "canonical",
        "status": "completed"
      },
      {
        "order": 2,
        "path": "_reversa_sdd/modulo/design.md",
        "unit": "modulo",
        "kind": "canonical",
        "status": "pending"
      }
    ],
    "last_completed_file": "_reversa_sdd/modulo/requirements.md",
    "next_file": "_reversa_sdd/modulo/design.md",
    "auto_advance": {
      "enabled": false,
      "scope": null,
      "unit": null
    }
  }
}
```

### 3.2 Invariante de `next_file`

Adicionar ao `Passo 2, Gerar um arquivo por vez` e ao schema de estado:

```text
pending := itens de generation_plan cujo status é pending

se pending não está vazio:
  next_file DEVE ser pending[0].path
se pending está vazio:
  next_file DEVE ser null
```

Consequências operacionais:

- é proibido salvar `next_file: null` enquanto houver qualquer item `pending`;
- após cada escrita ou preservação, atualizar o status do item e recalcular `next_file` na mesma gravação atômica;
- antes de retomar, validar a invariante; se estiver violada, recomputar `next_file` a partir do plano, registrar a correção e só então prosseguir;
- `next_file` não pode apontar para item `completed`, `preserved`, `skipped` ou inexistente;
- globais pendentes também impedem `next_file: null`.

### 3.3 Verificação de duplicidade antes de subunits híbridos

Expandir a seção `Caso hybrid` com uma barreira anterior à criação da subunit:

1. montar a identidade de rastreabilidade candidata usando caminhos relativos normalizados do legado e, quando disponíveis, símbolos, rotas, comandos ou fluxos;
2. comparar a identidade e o contrato comportamental pretendido com todas as units e subunits já planejadas, concluídas ou preservadas;
3. classificar o resultado como `unique`, `partial_overlap` ou `duplicate`;
4. persistir o resultado no item lógico da subunit antes de adicionar seus arquivos a `generation_plan`;
5. somente criar a subunit quando o resultado for `unique`, ou `partial_overlap` acompanhado de uma justificativa explícita de contrato distinto;
6. para `duplicate`, não criar pasta nem arquivos; registrar `duplicate_of` e a evidência da cobertura existente.

Modelo mínimo do registro:

```json
{
  "unit": "nucleo/execucao-compartilhada",
  "traceability_check": {
    "result": "duplicate",
    "duplicate_of": "wrappers-comandos/execucao-filtrada",
    "legacy_refs": ["src/core/runner.rs", "src/core/stream.rs"],
    "reason": "O contrato operacional e as referências já estão cobertos pela unit existente."
  }
}
```

Critério de decisão: compartilhar um arquivo do legado, isoladamente, não torna uma subunit duplicada. A duplicidade ocorre quando a cobertura das referências relevantes e o contrato comportamental proposto já estão integralmente representados. Sobreposição parcial é aceita apenas com fronteira comportamental documentada.

### 3.4 `auto_advance` exige limite explícito

Adicionar uma seção `Avanço automático limitado` ao Writer:

- `auto_advance` só pode ser habilitado com `scope: "unit"` e `unit` preenchida com a unit atual;
- comandos como “continue sem confirmação”, “pode seguir automaticamente” ou equivalentes habilitam avanço apenas até concluir a unit atual;
- um pedido sem limite explícito deve ser interpretado pelo limite seguro padrão “até concluir esta unit”, e o Writer deve informar esse limite antes de avançar;
- ao concluir a unit-alvo, o Writer deve desabilitar `auto_advance`, persistir o checkpoint e parar antes do primeiro arquivo da próxima unit;
- o limite não pode incluir duas units, “até o fim do plano” ou globais; para uma nova unit é necessária nova autorização;
- `auto_advance.enabled: true` com `scope` ou `unit` ausente é estado inválido e não autoriza geração.

Estado válido durante o avanço:

```json
{
  "auto_advance": {
    "enabled": true,
    "scope": "unit",
    "unit": "nucleo/tracking-e-telemetria"
  }
}
```

### 3.5 Fronteira dos comandos de continuação

No fluxo normal, distinguir:

- `CONTINUAR`: autoriza apenas o próximo arquivo;
- `continue sem confirmação` e equivalentes: autorizam os arquivos restantes da unit atual;
- ao terminar a unit atual: salvar estado, desabilitar o avanço automático e solicitar nova confirmação;
- se o comando for recebido no último arquivo de uma unit, ele autoriza somente esse arquivo e não o primeiro arquivo da unit seguinte;
- globais formam uma fronteira própria e não são alcançados por autorização concedida a uma unit.

## 4. Ajuste obrigatório no modo autônomo

O `reversa-autonomous` hoje declara que responde automaticamente a cada handoff e prossegue para a próxima tarefa do plano. Para não contradizer o novo contrato, substituir o override genérico por uma regra explícita:

> Durante a execução do Writer, o avanço sem confirmação é limitado à unit registrada em `redator_progress.auto_advance.unit`. Ao concluir essa unit, o Writer persiste o checkpoint e o orquestrador só pode habilitar a unit seguinte com um novo limite explícito. O modo autônomo pode conceder essa autorização unit a unit, mas nunca manter uma autorização ilimitada atravessando fronteiras de unit ou alcançando globais.

Para o modo autônomo continuar realmente sem supervisão, o próprio orquestrador pode renovar a autorização após validar o checkpoint da unit concluída. Cada renovação deve:

1. nomear a próxima unit;
2. persistir `auto_advance` com essa unit;
3. nunca agrupar várias units em uma única autorização;
4. tratar globais separadamente após todas as units.

Assim, “autônomo” continua sem intervenção humana, mas cada concessão permanece delimitada e auditável.

## 5. Arquivos a alterar no repositório-fonte

### Obrigatórios

1. `.agents/skills/reversa-writer/SKILL.md`
   - persistência integral do plano antes da primeira escrita;
   - schema e invariantes de `redator_progress`;
   - deduplicação híbrida;
   - semântica de `CONTINUAR` e avanço limitado;
   - retomada e reparo de estado legado.
2. `.claude/skills/reversa-writer/SKILL.md`
   - manter conteúdo semanticamente idêntico ao arquivo em `.agents`.
3. `.agents/skills/reversa/references/state-schema.md`
   - documentar `redator_progress`, `generation_plan`, `next_file`, `auto_advance` e seus invariantes.
4. `.claude/skills/reversa/references/state-schema.md`
   - espelho do schema acima.
5. `.agents/skills/reversa-autonomous/SKILL.md`
   - limitar a automação do Writer por unit e separar globais.
6. `.claude/skills/reversa-autonomous/SKILL.md`
   - espelho do ajuste autônomo.

### Gerados ou condicionais

7. Manifesto/checksums de distribuição
   - regenerar pelo processo oficial após alterar os skills; em instalações atuais ele aparece como `.reversa/_config/files-manifest.json`.
8. Teste de contrato do Writer
   - adicionar no diretório de testes do repositório-fonte conforme a convenção existente. Se ainda não houver harness para skills Markdown, criar um teste de validação que carregue um fixture de `state.json` e verifique as invariantes abaixo.
9. Changelog/versão do skill
   - incrementar `metadata.version` do Writer e do orquestrador autônomo conforme a política de release do Reversa; registrar a mudança no changelog do repositório-fonte.

Não é necessário alterar `.reversa/plan.md` de projetos já analisados para definir a regra global. Esse arquivo é estado de uma execução, não a fonte do comportamento do Writer.

## 6. Cenários mínimos de teste e aceite

1. **Plano antes do arquivo**: com saída vazia, nenhuma pasta ou spec é criada antes de `generation_plan` completo existir em `state.json`.
2. **Plano completo**: `plan_total_files == generation_plan.length`, incluindo opcionais e globais aplicáveis.
3. **Próximo pendente**: com três itens e o primeiro concluído, `next_file` aponta para o segundo.
4. **Nulo proibido**: uma tentativa de salvar `next_file: null` com item pendente é rejeitada ou reparada antes de gerar.
5. **Fim real**: `next_file` só se torna `null` quando não há itens pendentes.
6. **Retomada legada**: estado com apenas `plan_total_files` e `active_batch` é migrado/reconstruído antes de nova geração.
7. **Duplicidade híbrida total**: candidato já integralmente coberto não cria diretório nem itens de arquivo e registra `duplicate_of`.
8. **Sobreposição parcial**: candidato que compartilha arquivos, mas possui contrato distinto, é aceito somente com justificativa persistida.
9. **Auto sem escopo**: `enabled: true` sem unit não permite avançar.
10. **Auto por unit**: “continue sem confirmação” gera somente os arquivos restantes da unit atual.
11. **Último arquivo**: autorização recebida no último arquivo não atravessa para a próxima unit.
12. **Globais isolados**: autorização de unit não alcança `openapi`, `user-stories` ou `traceability`.
13. **Modo autônomo**: o orquestrador renova a autorização unit a unit, com checkpoint persistido entre elas.
14. **Non-destructive**: arquivo preexistente recebe status `preserved` e nunca é sobrescrito.

## 7. Ordem recomendada de implementação

1. definir o schema e as invariantes em `state-schema.md`;
2. atualizar o Writer para produzir, validar e reparar esse estado;
3. adicionar a barreira de deduplicação na enumeração híbrida;
4. implementar a semântica de autorização por unit;
5. alinhar o modo autônomo;
6. executar os testes de contrato;
7. sincronizar os espelhos `.agents`/`.claude`, atualizar versões e regenerar manifestos.

## 8. Fora de escopo desta entrega

Nenhum skill instalado, arquivo legado ou estado atual foi modificado por esta análise. Este documento é o único artefato criado e serve como handoff para uma alteração futura no repositório-fonte do Reversa.

## 9. Implementação local aprovada

Em 2026-07-14, após aprovação explícita do usuário, as mudanças deste relatório foram aplicadas à instalação local. A afirmação da seção 8 descreve somente a entrega inicial de análise e não o estado posterior à aprovação.

Arquivos de comportamento alterados:

- `.agents/skills/reversa-writer/SKILL.md` e seu espelho `.claude/skills/reversa-writer/SKILL.md`;
- `.agents/skills/reversa/references/state-schema.md` e seu espelho `.claude/skills/reversa/references/state-schema.md`;
- `.agents/skills/reversa-autonomous/SKILL.md` e seu espelho `.claude/skills/reversa-autonomous/SKILL.md`.

Metadados atualizados:

- Writer: versão `1.2.0` para `1.3.0`;
- Reversa Autonomous: versão `1.0.0` para `1.1.0`;
- `.reversa/_config/files-manifest.json`: hashes SHA-256 das seis cópias recalculados.

Nenhum arquivo do projeto legado, spec já gerada em `_reversa_sdd/`, plano de execução ou estado runtime em `.reversa/state.json` foi alterado por essa implementação. Estados antigos sem `generation_plan` serão reconstruídos pelo Writer antes da próxima geração, conforme o novo contrato.

## 10. Correção do limite de avanço — comando `loop`

Data da correção: 2026-07-15 (America/Bahia)
Commit analisado: `91ac8c52bda84000bf57d4a814ae34b0fc8d8e9e`

Esta seção substitui, para implementação futura no repositório-fonte, a semântica de avanço descrita nas seções 3.4, 3.5 e 4 e nos cenários 9 a 13.

### 10.1 Falha identificada

A implementação anterior tratava `unit` como limite plano. Em granularidade híbrida, a unit principal e cada subunit tinham identificadores distintos, por exemplo `analiticos`, `analiticos/relatorios-de-economia` e `analiticos/adocao-por-sessao`. Como não havia relação persistida entre elas, “continuar sem confirmação” podia encerrar após uma unit ou subunit isolada.

Também permaneceram prompts antigos que ofereciam somente `CONTINUAR`, portanto os modos não eram apresentados de forma consistente após cada arquivo.

### 10.2 Semântica corrigida

Os comandos canônicos do Writer são:

- `clear`: salva o checkpoint, desabilita loop ativo e encerra a conversa sem gerar outro arquivo;
- `continuar`: gera exatamente o arquivo apontado por `next_file`, salva e volta a apresentar os comandos;
- `loop`: processa todos os itens pendentes da unit principal alvo e de todas as suas subunits descendentes, sem confirmações intermediárias.

“Continuar sem confirmação” permanece apenas como alias retrocompatível de `loop`.

O loop nunca atravessa para outra unit principal nem alcança globais. Ao concluir a árvore, salva o checkpoint, desabilita-se e apresenta novamente os três comandos.

### 10.3 Hierarquia persistida

Cada item de `generation_plan` passa a incluir:

- `root_unit`: identificador da unit principal;
- `parent_unit`: pai imediato da subunit, ou `null` na raiz;
- `unit`: identificador integral da unit ou subunit do item.

Planos antigos sem esses campos devem ser migrados antes do próximo arquivo ou antes de aceitar `loop`, com incremento de `plan_revision`.

### 10.4 Loop finito e verificável

Ao iniciar `loop`, `auto_advance` persiste `mode: "loop"`, `scope: "unit_tree"`, `root_unit`, a lista finita `target_orders`, `max_iterations`, `completed_iterations` e `last_progress_order`.

Cada iteração deve reduzir em um a quantidade de alvos pendentes. O loop termina com sucesso somente quando nenhuma ordem de `target_orders` permanece pendente. Ele interrompe com checkpoint em erro irrecuperável, risco non-destructive, estouro de contexto ou estagnação.

### 10.5 Estado local migrado

O plano local foi migrado da revisão 1 para a revisão 2, preservando os 78 itens, seus status e `next_file`. Para o próximo alvo `analiticos`, o limite calculado contém nove arquivos:

- unit principal `analiticos`;
- subunit `analiticos/relatorios-de-economia`;
- subunit `analiticos/adocao-por-sessao`.

### 10.6 Arquivos adicionais afetados

Além dos arquivos listados na seção 9, a correção altera:

- `.agents/skills/reversa/SKILL.md` e `.claude/skills/reversa/SKILL.md`, para o orquestrador não substituir o menu do Writer;
- `.agents/skills/reversa/references/step-02-resume.md` e seu espelho `.claude`, para retomar automaticamente um loop válido interrompido por contexto;
- `.reversa/state.json`, migrado para a hierarquia explícita do plano atual.

Versões locais após a correção:

- Reversa Writer: `1.4.0`;
- Reversa Autonomous: `1.2.0`;
- Reversa orchestrator: `1.1.0`.

### 10.7 Aceite corrigido

1. Após cada arquivo fora de loop, a resposta mostra `clear`, `continuar` e `loop`.
2. `continuar` processa somente `next_file`.
3. `loop` em `analiticos` processa as ordens 31 a 39 e para antes da ordem 40.
4. Concluir a última tarefa de uma subunit não encerra o loop enquanto houver outra subunit da mesma `root_unit` pendente.
5. O loop não inclui globais nem itens de outra `root_unit`.
6. Uma iteração sem redução de pendências dispara estagnação e checkpoint.
7. `clear` preserva `next_file` e desabilita o loop.
8. Estouro de contexto preserva um loop válido para retomada automática.
