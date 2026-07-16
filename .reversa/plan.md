# Plano de Exploração — rtk

> Criado pelo Reversa em 2026-07-14
> Marque cada tarefa com ✅ quando concluída.
> Você pode editar este plano antes de iniciar: adicione, remova ou reordene tarefas conforme necessário.

---

## Fase 1: Reconhecimento 🔍

- ✅ **Scout** — Mapeamento de estrutura de pastas e tecnologias
- ✅ **Scout** — Análise de dependências e gerenciadores de pacotes
- ✅ **Scout** — Identificação de entry points, CI/CD e configurações

## Decisão de organização das specs 🗂️

> Entre o Scout e o Arqueólogo, o Reversa pergunta como você quer organizar as specs (por módulo, caso de uso, endpoint, híbrida, por features ou customizada). A escolha fica persistida em `.reversa/config.toml` na seção `[specs]` e não será reperguntada em execuções futuras. Para reapresentar o menu, remova manualmente a seção.

## Fase 2: Escavação 🏗️

> O Reversa preenche esta seção com os módulos reais após o Scout concluir o reconhecimento.

- ✅ **Archaeologist** — Análise do módulo `main`
- ✅ **Archaeologist** — Análise do módulo `cmds`
- ✅ **Archaeologist** — Análise do módulo `core`
- ✅ **Archaeologist** — Análise do módulo `hooks`
- ✅ **Archaeologist** — Análise do módulo `analytics`
- ✅ **Archaeologist** — Análise do módulo `discover`
- ✅ **Archaeologist** — Análise do módulo `learn`
- ✅ **Archaeologist** — Análise do módulo `parser`
- ✅ **Archaeologist** — Análise do módulo `filters`
- ✅ **Archaeologist** — Análise do módulo `openclaw`
- ✅ **Archaeologist** — Análise do módulo `docs`
- ✅ **Archaeologist** — Análise do módulo `scripts`

## Fase 3: Interpretação 🧠

- ✅ **Detetive** — Arqueologia Git e ADRs retroativos
- ✅ **Detetive** — Regras de negócio implícitas e máquinas de estado
- ✅ **Detetive** — Matriz de permissões (RBAC/ACL)
- ✅ **Arquiteto** — Diagramas C4 (Contexto, Containers, Componentes)
- ✅ **Arquiteto** — ERD completo e integrações externas
- ✅ **Arquiteto** — Spec Impact Matrix

## Fase 4: Geração 📝

- [ ] **Redator** — Specs SDD por componente
- [ ] **Redator** — OpenAPI (se aplicável)
- [ ] **Redator** — User Stories (se aplicável)
- [ ] **Redator** — Code/Spec Matrix

### Controle Incremental do Redator

> Organização configurada: `hybrid`. Cada módulo recebe os três arquivos canônicos; casos de uso aninhados só são criados quando acrescentam um contrato distinto, sem duplicar uma unit já documentada em outro módulo.

- ✅ `entrada-cli/` — unit do módulo `main` (3 arquivos canônicos)
- ✅ `entrada-cli/roteamento-e-fallback/` — caso de uso do roteamento da CLI (3 arquivos canônicos)
- ✅ `wrappers-comandos/` — unit do módulo `cmds` (3 arquivos canônicos)
- ✅ `wrappers-comandos/execucao-filtrada/` — contrato compartilhado de execução filtrada (3 arquivos canônicos)
- ✅ `nucleo/` — unit do módulo `core` (3 arquivos canônicos)
- ✅ `nucleo/pipeline-de-filtros-toml/` — registry trust-gated, compilação e aplicação da DSL (3 arquivos canônicos).
- ✅ `nucleo/tracking-e-telemetria/` — tracking SQLite, agregações, retenção e ping opt-in (3 arquivos canônicos).
- ✅ `integracoes-de-agentes/` — unit do módulo `hooks` (3 arquivos canônicos).
- ✅ `integracoes-de-agentes/protocolo-de-hooks-e-permissoes/` — contrato de instalação, integridade e permissões de hooks (3 arquivos canônicos).
- ✅ `analiticos/` — unit do módulo `analytics` (3 arquivos canônicos).
- ✅ `analiticos/relatorios-de-economia/` — correlação read-only entre tracking RTK e custos Claude Code (3 arquivos canônicos).
- ✅ `analiticos/adocao-por-sessao/` — medição de cobertura RTK em sessões Claude Code (3 arquivos canônicos).
- ✅ `descoberta/` — unit do módulo `discover` (3 arquivos canônicos).
- ✅ `descoberta/reescrita-e-classificacao/` — classificação shell e proposta de rewrite seguro (3 arquivos canônicos).
- ✅ `descoberta/analise-de-historico/` — leitura de sessões Claude Code e relatório de oportunidades (3 arquivos canônicos).

> Não criar `nucleo/execucao-compartilhada/`: sua superfície operacional já está coberta por `wrappers-comandos/execucao-filtrada/`, que rastreia `core::runner`, `core::stream`, `core::guard`, `core::tee` e `core::tracking`. Duplicá-la reduziria a qualidade da rastreabilidade. 🟢

> Após decidir os dois subunits candidatos, o Redator deve persistir a lista ordenada de arquivos pendentes em `.reversa/state.json` antes de voltar a gerar. O plano anterior registrou apenas o total de 78 arquivos e não preservou essa lista.

## Fase 5: Revisão ✅

- [ ] **Revisor** — Revisão cruzada de specs
- [ ] **Revisor** — Resolução de lacunas com o usuário
- [ ] **Revisor** — Relatório de confiança final

---

## Agentes Independentes

> Execute estes agentes quando os recursos estiverem disponíveis — podem rodar em qualquer fase.

- [ ] **Visor** — Análise de interface via screenshots
- [ ] **Data Master** — Análise completa do banco de dados
- [ ] **Design System** — Extração de tokens de design
- [ ] **Tracer** — Análise dinâmica (requer sistema acessível)

---

## Próximo passo

Após o Time de Descoberta concluir e o `_reversa_sdd/` estar populado, você pode disparar um dos fluxos seguintes:

- `/reversa-migrate`: orquestrador do **Time de Migração** (Paradigm Advisor → Curator → Strategist → Designer → Screen Translator → Inspector). Gera as specs do sistema novo. Saída em `_reversa_sdd/migration/` e `_reversa_sdd/screens/`.
- `/reversa-reconstructor`: gera plano bottom-up para reimplementar o software a partir das specs do legado (uma tarefa por sessão).
