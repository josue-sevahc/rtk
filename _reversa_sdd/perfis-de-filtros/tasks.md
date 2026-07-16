# Perfis de Filtros, Tarefas de Implementacao

> Sequencia para reconstruir o modulo declarativo de perfis em `src/filters/`.

## Pre-requisitos

- [ ] 🟢 Disponibilizar o schema TOML e o compilador de regexes descritos em `nucleo/pipeline-de-filtros-toml`.
- [ ] 🟢 Disponibilizar o processo de build capaz de gerar um artefato em `OUT_DIR`.
- [ ] 🟢 Certificar inicialmente apenas combinacoes cobertas por fixture ou teste no baseline Linux x86_64, Bash/Zsh e UTF-8; demais versoes e locales permanecem experimentais. Decisao do usuario em 2026-07-16.

## Tarefas

- [ ] T-01, Definir o formato de perfil TOML com `match_command`, descricao e acoes da DSL. Confianca: 🟢
  - Origem no legado: `src/filters/README.md`, `src/core/toml_filter.rs:80-108`.
  - Criterio de pronto: uma definicao valida e uma definicao com campo desconhecido sao distinguidas pelo parser.

- [ ] T-02, Criar perfis isolados por ferramenta ou subcomando, nomeando o arquivo pelo comando coberto. Confianca: 🟢
  - Origem no legado: `src/filters/README.md`, `src/filters/*.toml`.
  - Criterio de pronto: cada novo perfil pode ser localizado pelo nome de arquivo e contem ao menos uma transformacao declarativa.

- [ ] T-03, Implementar perfis representativos para logs de instalacao, monitoramento, linters e ferramentas de infraestrutura. Confianca: 🟢
  - Origem no legado: `src/filters/brew-install.toml`, `src/filters/systemctl-status.toml`, `src/filters/shellcheck.toml`, `src/filters/terraform-plan.toml`.
  - Criterio de pronto: cada familia preserva uma saida operacional reconhecivel apos remocao de ruido.

- [ ] T-04, Associar testes inline de entrada e saida a cada perfil. Confianca: 🟢
  - Origem no legado: `src/filters/*.toml`, `src/core/toml_filter.rs:657-799`.
  - Criterio de pronto: todo perfil aparece como testado e as comparacoes literais passam.

- [ ] T-05, Revisar perfis que se sobrepoem para evitar que o primeiro casamento oculte um contrato mais especifico. Confianca: 🟡
  - Origem no legado: `build.rs:20-23`, `src/core/toml_filter.rs:481-486`.
  - Criterio de pronto: comandos representativos escolhem o perfil pretendido em ordem deterministica.

## Tarefas de Teste

- [ ] TT-01, Validar desserializacao, `schema_version` e regexes de todos os perfis embutidos. Confianca: 🟢 `src/core/toml_filter.rs:1237-1243`
- [ ] TT-02, Executar todos os exemplos inline e reprovar qualquer perfil sem teste. Confianca: 🟢 `src/core/toml_filter.rs:657-799`, `src/core/toml_filter.rs:1909-1930`
- [ ] TT-03, Testar perfis com `strip_ansi`, `replace`, `match_output`, selecao de linhas, limites e `on_empty`. Confianca: 🟢 `src/core/toml_filter.rs:515-628`
- [ ] TT-04, Verificar a fixture de cada familia contra uma execucao real da ferramenta e registrar diferencas de versao ou locale. Confianca: 🔴

## Tarefas de Migracao de Dados

- [ ] Nao aplicavel. 🟢 Os perfis sao fontes TOML versionadas e o catalogo e gerado durante o build.

## Ordem Sugerida

1. Implementar T-01 antes de criar perfis para manter o contrato da DSL verificavel.
2. Executar T-02 a T-04 em conjunto para que cada perfil nasca com testes.
3. Concluir T-05 depois que o catalogo tiver cobertura representativa e ordem deterministica.

## Lacunas Pendentes (🔴)

- Validar a compatibilidade de perfis com ferramentas externas reais antes de prometer equivalencia de saida.
- 🟢 Os valores atuais formam o perfil versionado `legacy-v1`; novos perfis exigem dataset, benchmark, metricas de falsos positivos/economia e protecao `never_worse`. Decisao do usuario em 2026-07-16.
