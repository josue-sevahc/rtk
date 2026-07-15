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
    "hybrid_traceability_checks": [],
    "last_completed_file": "_reversa_sdd/modulo/requirements.md",
    "next_file": "_reversa_sdd/modulo/design.md",
    "auto_advance": {
      "enabled": false,
      "scope": null,
      "unit": null
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
| `auto_advance` | object | Autorização temporária de avanço, limitada à unit atual |

Cada item de `generation_plan` contém no mínimo:

- `order`: inteiro positivo, único e crescente;
- `path`: caminho do arquivo dentro do output do Reversa;
- `unit`: identificador da unit, ou `global` para artefatos globais;
- `kind`: `canonical`, `optional` ou `global`;
- `status`: `pending`, `completed`, `preserved` ou `skipped`.

Arquivos preexistentes usam `preserved` e nunca são sobrescritos. Uma visão parcial como `active_batch` pode existir por compatibilidade, mas nunca substitui `generation_plan`. Em estado legado sem o plano integral, o Writer deve reconstruí-lo e persistir uma revisão completa antes de gerar outro arquivo.

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

### Invariante de `auto_advance`

`auto_advance.enabled: true` somente é válido com `scope: "unit"` e `unit` igual à unit atual. A autorização termina no último arquivo dessa unit; nesse checkpoint, o Writer grava `enabled: false`, `scope: null` e `unit: null` antes de qualquer arquivo da unit seguinte. Autorizações de unit nunca alcançam globais.

## Onde NÃO escrever

A decisão de organização das specs (granularidade, pastas customizadas, sugestão original do Scout, timestamp da escolha) **não** vai no `state.json`. Ela é persistida em `.reversa/config.toml`, seção `[specs]`, conforme `references/step-03-specs-organization.md`. O `state.json` é estado runtime, o `config.toml` é decisão de longo prazo.
