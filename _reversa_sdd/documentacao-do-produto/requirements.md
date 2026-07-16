# Documentacao do Produto

> Contrato operacional da documentacao que apresenta, orienta e governa o uso e a contribuicao no RTK. Todas as afirmacoes abaixo indicam a confianca da evidencia.

## Visao Geral

A unit publica a jornada de uso do RTK: instalar, inicializar a integracao com um agente, configurar comportamento, medir economia e diagnosticar problemas. 🟢 A mesma superficie documental registra contratos tecnicos, de telemetria e de contribuicao para manter o comportamento publico alinhado ao binario. 🟢

## Responsabilidades

- Publicar ponto de entrada, guias de instalacao, inicio rapido, agentes suportados, configuracao, analytics, troubleshooting e privacidade. 🟢
- Explicar que hooks e plugins delegam a decisao de reescrita para `rtk rewrite`, preservando o fluxo usual de comandos do agente. 🟢
- Documentar configuracoes de tracking, filtros, tee, telemetria e exclusoes de hook, incluindo o requisito de trust explicito para filtros customizados. 🟢
- Estabelecer praticas de contribuicao: fidelidade do output, falha aberta, baixo overhead, fixtures reais, testes e documentacao afetada. 🟢

## Regras de Negocio

- A documentacao de instalacao deve alertar que existe outro projeto chamado `rtk`; `rtk gain` e o teste operacional para identificar o Rust Token Killer. 🟢
- A jornada recomendada orienta instalar, inicializar o agente, executar comandos normalmente e consultar `rtk gain`, `rtk discover` ou `rtk session` conforme a necessidade. 🟢
- Um filtro customizado nao deve ser aplicado antes de o usuario executar um trust explicito; uma alteracao no conteudo exige novo trust. 🟢
- Telemetria depende de consentimento explicito, usa dados agregados e nao deve incluir codigo, caminhos de arquivos ou conteudo completo de comandos. 🟢
- Quando um hook ou filtro falha, o comportamento documentado deve preservar a execucao ou a saida bruta em vez de bloquear o usuario. 🟢
- O conjunto exato de paginas, exemplos e integracoes mantidas em cada release pode evoluir com o produto. 🟡

## Requisitos Funcionais

| ID | Requisito | Prioridade | Criterio de Aceite |
|----|-----------|------------|--------------------|
| RF-01 | A documentacao deve oferecer um indice com instalacao, inicio rapido, agentes suportados, configuracao, analytics, troubleshooting e privacidade. 🟢 | Must | Um leitor encontra links para esses destinos a partir de `docs/guide/index.md`. |
| RF-02 | A instalacao deve apresentar os canais curl, Homebrew, Cargo e binarios pre-compilados, alem da verificacao por `rtk --version` e `rtk gain`. 🟢 | Must | O guia `installation.md` contem os comandos e a advertencia de colisao de nome. |
| RF-03 | A documentacao deve explicar como integrar RTK aos agentes suportados e o papel de `rtk rewrite` no fluxo de interceptacao. 🟢 | Must | `supported-agents.md` traz matriz de tiers e instrucoes por agente. |
| RF-04 | A configuracao deve descrever opcoes de tracking, display, filtros, tee, telemetria e hooks, com variaveis de ambiente e exclusoes. 🟢 | Must | `configuration.md` possui estrutura TOML, tabela de variaveis e exemplos. |
| RF-05 | A documentacao deve especificar trust baseado no conteudo para filtros locais e globais. 🟢 | Must | O guia exige `rtk trust` e informa que edicoes requerem nova confianca. |
| RF-06 | A contribuicao deve informar criterios de qualidade para novos filtros, incluindo fixture real, testes e economia minima observada. 🟢 | Should | `CONTRIBUTING.md` e a documentacao tecnica descrevem o fluxo de contribuicao. |

## Requisitos Nao Funcionais

| Tipo | Requisito inferido | Evidencia no codigo | Confianca |
|------|--------------------|---------------------|-----------|
| Clareza | A jornada inicial deve permitir instalacao e ativacao sem exigir conhecimento da arquitetura interna. | `docs/guide/index.md:35`, `docs/guide/getting-started/quick-start.md:8` | 🟢 |
| Seguranca | A documentacao deve manter visivel a separacao entre o RTK correto e o pacote homonimo, e o trust deve ser explicito. | `docs/guide/getting-started/installation.md:10`, `docs/guide/getting-started/configuration.md:139` | 🟢 |
| Privacidade | A descricao de telemetria deve limitar o contrato a agregados e oferecer opt-out. | `docs/guide/getting-started/configuration.md:113` | 🟢 |
| Manutenibilidade | Mudancas de comportamento devem ter paginas afetadas atualizadas junto com contribuicoes. | `CONTRIBUTING.md`, `docs/contributing/TECHNICAL.md` | 🟢 |
| Disponibilidade | A orientacao publica deve refletir falha aberta para evitar bloquear comandos do agente. | `docs/contributing/TECHNICAL.md:59` | 🟢 |

## Criterios de Aceitacao

```gherkin
Cenario: Usuario valida uma instalacao existente
  Dado que o comando rtk esta disponivel no PATH
  Quando executa "rtk --version" e "rtk gain"
  Entao a documentacao permite confirmar que a instalacao e do Rust Token Killer

Cenario: Usuario tenta usar filtro customizado ainda nao confiado
  Dado um arquivo .rtk/filters.toml sem trust vigente
  Quando executa um comando elegivel
  Entao o guia informa que o filtro fica inativo ate "rtk trust"

Cenario: Falha durante reescrita de comando
  Dado que hook ou filtro encontra um erro
  Quando o agente executa um comando
  Entao o contrato documentado preserva comando ou saida bruta
```

## Prioridade (MoSCoW)

| Requisito | MoSCoW | Justificativa |
|-----------|--------|---------------|
| Instalacao e verificacao do binario correto. 🟢 | Must | Sem isso, a integracao pode apontar para o projeto homonimo. |
| Inicio rapido e integracao por agente. 🟢 | Must | E o caminho principal para o proxy funcionar de modo transparente. |
| Configuracao, trust e privacidade. 🟢 | Must | Afetam seguranca, comportamento e consentimento do usuario. |
| Guias de contribuicao e manutencao. 🟢 | Should | Mantem a qualidade e a coerencia das mudancas futuras. |
| Catalogo completo por integracao e exemplos adicionais. 🟡 | Could | Amplia cobertura, mas nao substitui a jornada central. |

## Rastreabilidade de Codigo

| Arquivo | Funcao / Classe | Cobertura |
|---------|-----------------|-----------|
| `docs/guide/index.md` | Indice e jornada do usuario | 🟢 |
| `docs/guide/getting-started/installation.md` | Instalacao e verificacao | 🟢 |
| `docs/guide/getting-started/quick-start.md` | Ativacao inicial | 🟢 |
| `docs/guide/getting-started/configuration.md` | Configuracao, tee, telemetria e trust | 🟢 |
| `docs/guide/getting-started/supported-agents.md` | Matriz e instrucao de integracoes | 🟢 |
| `docs/contributing/TECHNICAL.md` | Contratos tecnicos e qualidade | 🟢 |
| `docs/TELEMETRY.md` | Contrato de privacidade | 🟢 |
| `CONTRIBUTING.md` | Principios e processo de contribuicao | 🟢 |
