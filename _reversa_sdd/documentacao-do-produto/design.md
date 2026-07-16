# Documentacao do Produto, Design Tecnico

> Implementacao documental da superficie publica do RTK. As afirmacoes tecnicas carregam marcador de confianca.

## Interface

| Simbolo | Assinatura | Retorno | Observacao |
|---------|------------|---------|------------|
| Guia de instalacao | canais de distribuicao + comandos de verificacao | ambiente RTK verificavel | Explica a colisao de nome e o uso de `rtk gain`. 🟢 |
| Guia de inicio rapido | `rtk init` ou `rtk init --global` | integracao configurada | Inclui opcao `--dry-run` para previsualizacao. 🟢 |
| Matriz de agentes | agente + tier de integracao | instrucao de setup por agente | Hooks, plugins e regras sao distinguidos por capacidade de reescrita. 🟢 |
| Referencia de configuracao | TOML, variaveis de ambiente e comandos | comportamento configuravel | Cobre tracking, display, filtros, tee, telemetria e hooks. 🟢 |
| Politica de trust | `rtk trust` / `rtk untrust` | filtro autorizado ou revogado | A confianca e vinculada ao SHA-256 do conteudo. 🟢 |

## Fluxo Principal

1. O indice em `docs/guide/index.md` apresenta RTK como proxy entre o agente e ferramentas de desenvolvimento e encaminha para os guias de inicio. 🟢
2. O leitor segue `installation.md`, escolhe curl, Homebrew, Cargo ou binario pre-compilado e valida a instalacao por `rtk --version` e `rtk gain`. 🟢
3. O guia de inicio rapido orienta `rtk init` no projeto ou `rtk init --global`, com `--dry-run` quando a intencao e apenas inspecionar alteracoes. 🟢
4. A pagina de agentes associa o host ao mecanismo de integracao; hooks e plugins chamam `rtk rewrite` para decidir a reescrita antes da execucao. 🟢
5. O leitor usa comandos normalmente; RTK filtra saidas reconhecidas, rastreia uso e disponibiliza `gain`, `discover` e `session` como superficies de analise. 🟢
6. A configuracao oferece excecoes por comando, tee para saidas completas e trust explicito para filtros customizados. 🟢

## Fluxos Alternativos

- **Binario homonimo detectado:** se `rtk --version` funciona, mas `rtk gain` nao, o guia instrui remover o pacote errado e reinstalar a origem correta. 🟢
- **Previsualizacao de inicializacao:** `rtk init --dry-run` imprime as alteracoes previstas e nao grava arquivos nem solicita consentimento de telemetria. 🟢
- **Comando sem filtro reconhecido:** a documentacao orienta passthrough por `rtk proxy`, mantendo a saida inalterada e o uso rastreado. 🟢
- **Filtro sem trust ou alterado:** o arquivo e ignorado na rota de comando ate o usuario revisar e confiar novamente. 🟢
- **Falha de hook ou filtro:** o contrato tecnico prescreve falha aberta; o comando ou a saida bruta continuam disponiveis. 🟢

## Dependencias

- `src/core`, `src/cmds` e `src/filters`: fornecem os comportamentos de proxy, filtragem e configuracao descritos publicamente. 🟢
- `src/hooks`, `hooks/` e `openclaw/`: sustentam os adaptadores por agente apresentados na matriz de integracoes. 🟢
- `src/analytics` e `src/discover`: sustentam os comandos de economia, descoberta e sessao citados nos guias. 🟢
- `README.md` e `CONTRIBUTING.md`: complementam a superficie de onboarding e as regras de contribuicao. 🟢
- Infraestrutura de publicacao do site de documentacao: `docs/guide/` alimenta o repositorio `rtk-ai/rtk-website` pelo pipeline `prepare-docs.mjs`, que publica a navegacao via Starlight. 🟢 `.github/docs-pipeline-contract.md:3`

## Decisoes de Design Identificadas

| Decisao | Evidencia no codigo | Confianca |
|---------|---------------------|-----------|
| Organizar a experiencia por jornada: instalar, iniciar, configurar, medir e diagnosticar. | `docs/guide/index.md:35` | 🟢 |
| Centralizar a decisao de reescrita em `rtk rewrite`; adaptadores apenas traduzem o protocolo de cada agente. | `docs/guide/getting-started/supported-agents.md` | 🟢 |
| Manter transparencia: a saida filtrada deve ser subconjunto util da saida real, sem formato artificial padrao. | `CONTRIBUTING.md` | 🟢 |
| Exigir trust explicito e sensivel ao conteudo para filtros que podem alterar o que o agente enxerga. | `docs/guide/getting-started/configuration.md:139` | 🟢 |
| Priorizar telemetria opt-in com dados agregados e opt-out configuravel. | `docs/guide/getting-started/configuration.md:113` | 🟢 |

## Estado Interno

A unit e majoritariamente declarativa: paginas Markdown com front matter de titulo, descricao e ordem lateral compoem a documentacao do guia. 🟢 A configuracao apresentada ao usuario cobre estado persistido de tracking, tee, telemetria e regras de hooks, mas a implementacao desses estados pertence a outras units. 🟢

## Observabilidade

- `rtk gain`, `rtk gain --daily` e `rtk gain --weekly` sao apresentados como visualizacoes do ganho acumulado. 🟢
- `rtk discover` identifica comandos executados sem RTK e `rtk session` mostra adocao por sessao. 🟢
- O tee registra o caminho para a saida bruta em falhas, permitindo recuperacao sem nova execucao. 🟢

## Riscos e Lacunas

- 🟢 O contrato de integracao com o site esta documentado em `.github/docs-pipeline-contract.md`; a implementacao de `prepare-docs.mjs` pertence ao repositorio externo `rtk-ai/rtk-website`.
- 🟡 Alguns exemplos de versao, cobertura percentual e lista de agentes podem ficar defasados se nao forem atualizados junto com releases.
- 🟡 Existem referencias historicas a `CLAUDE.md` em scripts de diagnostico, enquanto guias mais recentes cobrem outros agentes; a consistencia integral entre todas as paginas requer revisao editorial.
