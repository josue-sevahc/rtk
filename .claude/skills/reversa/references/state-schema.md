# Schema — .reversa/state.json

Este arquivo persiste o estado completo da análise entre sessões. O Reversa lê e escreve neste arquivo.

## Estrutura completa

```json
{
  "version": "1.0.0",
  "project": "nome-do-projeto",
  "user_name": "Nome do Usuário",
  "chat_language": "pt-br",
  "doc_language": "Português",
  "answer_mode": "chat",
  "doc_level": null,
  "output_folder": "_reversa_sdd",
  "phase": "reconhecimento",
  "completed": ["reconhecimento"],
  "pending": ["escavacao", "interpretacao", "geracao", "revisao"],
  "engines": ["claude-code"],
  "agents": ["reversa", "reversa-scout", "reversa-archaeologist"],
  "checkpoints": {
    "scout": {
      "completed_at": "2026-04-26T10:00:00Z",
      "files": [
        "_reversa_sdd/inventory.md",
        "_reversa_sdd/dependencies.md",
        ".reversa/context/surface.json"
      ]
    },
    "archaeologist": {
      "completed_at": "2026-04-26T11:00:00Z",
      "modules_analyzed": ["auth", "orders", "payments"],
      "files": [
        "_reversa_sdd/code-analysis.md",
        "_reversa_sdd/data-dictionary.md",
        ".reversa/context/modules.json"
      ]
    }
  },
  "redator_progress": {
    "plan_revision": 1,
    "plan_total_files": 2,
    "generation_plan": [
      {
        "order": 1,
        "path": "_reversa_sdd/modulo/requirements.md",
        "unit": "modulo",
        "root_unit": "modulo",
        "parent_unit": null,
        "kind": "canonical",
        "status": "completed"
      },
      {
        "order": 2,
        "path": "_reversa_sdd/modulo/design.md",
        "unit": "modulo",
        "root_unit": "modulo",
        "parent_unit": null,
        "kind": "canonical",
        "status": "pending"
      }
    ],
    "hybrid_traceability_checks": [],
    "last_completed_file": "_reversa_sdd/modulo/requirements.md",
    "next_file": "_reversa_sdd/modulo/design.md",
    "auto_advance": {
      "enabled": false,
      "mode": null,
      "scope": null,
      "root_unit": null,
      "target_orders": [],
      "max_iterations": 0,
      "completed_iterations": 0,
      "last_progress_order": null
    }
  },
  "created_files": [
    "CLAUDE.md",
    ".agents/skills/reversa/SKILL.md",
    ".reversa/state.json",
    ".reversa/plan.md"
  ]
}
```

## Campos

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `version` | string | Versão do Reversa instalada |
| `project` | string | Nome do projeto legado |
| `user_name` | string | Nome do usuário (para interações) |
| `chat_language` | string | Idioma das interações (ex: pt-br, en-us) |
| `doc_language` | string | Idioma das specs geradas (ex: Português, English) |
| `answer_mode` | string | Como o usuário responde às lacunas: `chat` ou `file` |
| `doc_level` | string \| null | Volume de documentação gerada: `essencial`, `completo` ou `detalhado`. Começa `null` — obrigatório preencher via escolha do usuário após o Scout. |
| `output_folder` | string | Pasta de saída das specs (padrão: `_reversa_sdd`) |
| `phase` | string \| null | Fase atual. `null` = não iniciado |
| `completed` | string[] | Fases concluídas |
| `pending` | string[] | Fases pendentes |
| `checkpoints` | object | Registro de conclusão de cada agente |
| `redator_progress` | object | Plano integral e estado retomável do Writer |
| `engines` | string[] | Engines configuradas (ex: `["claude-code", "codex"]`) |
| `agents` | string[] | Agentes instalados |
| `created_files` | string[] | Todos os arquivos criados pelo Reversa (para uninstall seguro) |

## Fases válidas

`reconhecimento` → `escavacao` → `interpretacao` → `geracao` → `revisao`

## Regra ao escrever

Nunca remova campos existentes. Apenas adicione ou atualize.

## Contrato de `redator_progress`

O Writer persiste `redator_progress.generation_plan` completo e aprovado **antes** de criar qualquer diretório de unit ou arquivo de spec. A atualização do plano, do status do item e de `next_file` deve ser atômica.

| Campo | Tipo | Regra |
|-------|------|-------|
| `plan_revision` | integer | Começa em 1 e é incrementado sempre que um plano ajustado é aprovado |
| `plan_total_files` | integer | Deve ser igual a `generation_plan.length` |
| `generation_plan` | object[] | Lista integral e ordenada de arquivos de units, opcionais e globais |
| `hybrid_traceability_checks` | object[] | Resultados `unique`, `partial_overlap` ou `duplicate` avaliados antes de planejar subunits híbridas |
| `last_completed_file` | string \| null | Último arquivo efetivamente concluído; não substitui o plano |
| `next_file` | string \| null | Caminho do primeiro item pendente; só pode ser `null` quando não houver pendências |
| `auto_advance` | object | Estado do comando `loop`, limitado à árvore de uma unit principal |

Cada item de `generation_plan` contém no mínimo:

- `order`: inteiro positivo, único e crescente;
- `path`: caminho do arquivo dentro do output do Reversa;
- `unit`: identificador da unit ou subunit, ou `global` para artefatos globais;
- `root_unit`: unit principal que contém o item e todas as suas subunits; `null` para globais;
- `parent_unit`: pai imediato da subunit; `null` para a unit principal e globais;
- `kind`: `canonical`, `optional` ou `global`;
- `status`: `pending`, `completed`, `preserved` ou `skipped`.

Arquivos preexistentes usam `preserved` e nunca são sobrescritos. Uma visão parcial como `active_batch` pode existir por compatibilidade, mas nunca substitui `generation_plan`. Em estado legado sem o plano integral ou sem `root_unit`/`parent_unit`, o Writer deve reconstruir a hierarquia e persistir uma revisão completa antes de gerar outro arquivo ou aceitar `loop`.

### Invariante de `next_file`

```text
pending := generation_plan, em ordem, filtrado por status == pending

se pending não está vazio:
  next_file DEVE ser pending[0].path
se pending está vazio:
  next_file DEVE ser null
```

É inválido salvar `next_file: null` com qualquer item pendente, inclusive global. Também é inválido apontar `next_file` para item `completed`, `preserved`, `skipped` ou inexistente. Na retomada, o Writer deve validar essa invariante, reparar `next_file` a partir do plano e persistir a correção antes de continuar.

### Duplicidade de subunits híbridas

Antes de adicionar arquivos de uma subunit híbrida ao plano, registre em `hybrid_traceability_checks` sua identidade normalizada (`legacy_refs` e, quando disponíveis, símbolos, rotas, comandos e fluxos), o resultado e a justificativa. `partial_overlap` exige uma fronteira comportamental distinta em `reason`. `duplicate` exige `duplicate_of` e não pode criar diretório ou itens em `generation_plan`.

### Invariantes do comando `loop`

`auto_advance.enabled: true` somente é válido quando todos estes campos satisfazem o contrato:

- `mode == "loop"`;
- `scope == "unit_tree"`;
- `root_unit` identifica uma unit principal existente;
- `target_orders` contém exatamente as ordens que estavam `pending` nessa árvore ao ativar o loop;
- `max_iterations == target_orders.length`;
- `0 <= completed_iterations <= max_iterations`;
- `last_progress_order` é `null` antes da primeira iteração ou pertence a `target_orders`.

A cada iteração, exatamente um alvo deve sair de `pending`, `completed_iterations` deve aumentar e `next_file` deve continuar obedecendo sua invariante global. Se a quantidade de alvos pendentes não diminuir, se o próximo alvo sair de `target_orders` ou se o limite for ultrapassado, o loop está estagnado e deve parar com checkpoint.

O sucesso ocorre somente quando nenhuma ordem em `target_orders` permanece `pending`. Então o Writer desabilita o loop antes de tocar a próxima unit principal. Um loop nunca inclui ordens de outra `root_unit` nem globais.

O comando `clear` sempre grava `enabled: false` e preserva `next_file`. Em estouro involuntário de contexto, o loop válido permanece ativo para que a retomada continue sem nova confirmação.

## Onde NÃO escrever

A decisão de organização das specs (granularidade, pastas customizadas, sugestão original do Scout, timestamp da escolha) **não** vai no `state.json`. Ela é persistida em `.reversa/config.toml`, seção `[specs]`, conforme `references/step-03-specs-organization.md`. O `state.json` é estado runtime, o `config.toml` é decisão de longo prazo.
