# Documentacao do Produto, Tarefas de Implementacao

> Sequencia para reimplementar a superficie documental do RTK sem depender do codigo original.

## Pre-requisitos

- [ ] A interface de CLI e os comandos `rtk init`, `rtk rewrite`, `rtk gain`, `rtk discover`, `rtk session`, `rtk trust` e `rtk untrust` estao definidos. 🟢
- [ ] As integracoes de agentes e seus mecanismos de hook, plugin ou rules file estao disponiveis. 🟢
- [ ] O contrato de configuracao para tracking, filtros, tee, telemetria e hooks esta documentado pela unit correspondente. 🟢
- [ ] O destino de publicacao e `rtk-ai/rtk-website`, via `prepare-docs.mjs` e Starlight. 🟢 Origem: `.github/docs-pipeline-contract.md`.

## Tarefas

- [ ] T-01, Criar o indice documental com proposta de valor, fluxograma textual do proxy e links para instalacao, inicio rapido, agentes, configuracao, analytics, troubleshooting e privacidade.
  - Origem no legado: `docs/guide/index.md:8`
  - Criterio de pronto: o leitor encontra a jornada inicial e os destinos de consulta a partir de uma unica pagina.
  - Confianca: 🟢

- [ ] T-02, Implementar guia de instalacao para curl, Homebrew, Cargo e binarios pre-compilados, incluindo alerta de colisao com Rust Type Kit e verificacao por `rtk gain`.
  - Origem no legado: `docs/guide/getting-started/installation.md:10`
  - Criterio de pronto: o guia permite confirmar a instalacao correta e recuperar-se do pacote homonimo.
  - Confianca: 🟢

- [ ] T-03, Documentar inicializacao local e global por `rtk init`, com previsualizacao por `--dry-run` e o comportamento de nao gravar alteracoes nesse modo.
  - Origem no legado: `docs/guide/getting-started/quick-start.md:20`
  - Criterio de pronto: cada modo explica escopo, reinicio do agente e resultado esperado.
  - Confianca: 🟢

- [ ] T-04, Publicar matriz de agentes e guias especificos, distinguindo reescrita transparente por hook ou plugin de integracoes baseadas em regras.
  - Origem no legado: `docs/guide/getting-started/supported-agents.md:8`
  - Criterio de pronto: cada agente suportado tem mecanismo e comando de instalacao ou limitacao explicitados.
  - Confianca: 🟢

- [ ] T-05, Escrever referencia de configuracao para secoes TOML, variaveis de ambiente, tee, exclusoes de hook e telemetria opt-in.
  - Origem no legado: `docs/guide/getting-started/configuration.md:10`
  - Criterio de pronto: cada opcao tem finalidade, valor padrao ou exemplo quando este existe no legado.
  - Confianca: 🟢

- [ ] T-06, Documentar o ciclo de vida de filtros customizados: local ou global, revisao por `rtk trust`, revogacao por `rtk untrust` e novo trust apos edicao.
  - Origem no legado: `docs/guide/getting-started/configuration.md:130`
  - Criterio de pronto: o leitor entende quando um filtro e ignorado e como habilita-lo com seguranca.
  - Confianca: 🟢

- [ ] T-07, Manter guia de contribuicao com transparencia de output, falha aberta, baixo overhead, fixtures reais, testes e atualizacao documental.
  - Origem no legado: `CONTRIBUTING.md`, `docs/contributing/TECHNICAL.md:59`
  - Criterio de pronto: uma contribuicao de filtro tem criterio de qualidade e rastreabilidade claros.
  - Confianca: 🟢

## Tarefas de Teste

- [ ] TT-01, Validar links internos e metadados de front matter para todas as paginas do guia.
  - Origem no legado: `docs/guide/index.md`, `scripts/validate-docs.sh`
  - Criterio de pronto: nenhuma rota ou referencia interna essencial permanece quebrada.
  - Confianca: 🟡

- [ ] TT-02, Verificar que o guia de instalacao diferencia corretamente o binario Token Killer do projeto homonimo usando `rtk gain`.
  - Origem no legado: `docs/guide/getting-started/installation.md:10`, `scripts/check-installation.sh:38`
  - Criterio de pronto: os dois resultados de verificacao e a acao de recuperacao estao documentados.
  - Confianca: 🟢

- [ ] TT-03, Testar editorialmente que filtros sem trust e filtros alterados exibem a instrucao de trust/re-trust, sem sugerir ativacao automatica.
  - Origem no legado: `docs/guide/getting-started/configuration.md:139`
  - Criterio de pronto: exemplos e texto preservam o requisito de confirmacao explicita.
  - Confianca: 🟢

- [ ] TT-04, Conferir que exemplos de agentes descrevem a falha aberta e nao prometem reescrita onde a integracao usa apenas rules file.
  - Origem no legado: `docs/guide/getting-started/supported-agents.md`
  - Criterio de pronto: as capacidades declaradas batem com a matriz de tiers.
  - Confianca: 🟢

## Tarefas de Migracao de Dados

Nao se aplica: a unit publica conhecimento em arquivos Markdown e nao possui migracao de dados persistidos propria. 🟢

## Ordem Sugerida

1. Construir o indice, instalacao e inicio rapido, pois definem a entrada de qualquer usuario. 🟢
2. Adicionar matriz de agentes e configuracao antes de analytics e troubleshooting, pois dependem dos contratos do binario e das integracoes. 🟢
3. Publicar politicas de trust, telemetria e contribuicao em conjunto com os mecanismos implementados, para evitar documentar comportamento inexistente. 🟢
4. Executar validacao de links e revisao de consistencia sempre que uma release alterar comandos, agentes ou defaults. 🟡

## Lacunas Pendentes (🔴)

- 🟢 `docs/guide/` alimenta `rtk-ai/rtk-website` via `prepare-docs.mjs` e Starlight. Evidencia: `.github/docs-pipeline-contract.md`.
- 🟢 A lista certificada inicial deve refletir apenas Linux x86_64, Claude Code e OpenClaw; outros agentes permanecem experimentais ate validacao executavel. Decisao do usuario em 2026-07-16.
