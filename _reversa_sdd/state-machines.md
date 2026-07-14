# Maquinas de Estado - RTK

Projeto: `rtk`  
Fase: Interpretacao (Detetive)

## Escala de confianca

- 🟢 **CONFIRMADO** - transicao codificada.
- 🟡 **INFERIDO** - composicao de fluxos confirmados.
- 🔴 **LACUNA** - sem validacao em execucao.

## Confianca de filtros customizados

```mermaid
stateDiagram-v2
    [*] --> Ausente
    Ausente --> NaoConfiavel: arquivo encontrado sem entrada valida
    NaoConfiavel --> Revisao: rtk trust
    Revisao --> NaoConfiavel: TOML invalido ou usuario recusa
    Revisao --> Confiavel: hash SHA-256 aprovado
    Confiavel --> Carregado: hash atual confere e UTF-8 valido
    Confiavel --> ConteudoAlterado: hash diverge
    ConteudoAlterado --> Revisao
    Carregado --> NaoConfiavel: rtk untrust
    NaoConfiavel --> Carregado: override CI valido
```

🟢 A transicao `Confiavel -> ConteudoAlterado` impede carga do conteudo alterado. O override so vale para CI detectado e continua exigindo leitura bem-sucedida.

## Integridade do hook legado

```mermaid
stateDiagram-v2
    [*] --> NaoInstalado
    NaoInstalado --> SemBaseline: script existe, hash ausente
    NaoInstalado --> Verificado: instalacao grava hash
    SemBaseline --> Verificado: rtk init reestabelece baseline
    Verificado --> Adulterado: hash atual diverge
    Verificado --> HashOrfao: script removido, hash permanece
    Adulterado --> Verificado: rtk init reestabelece baseline
    HashOrfao --> NaoInstalado: limpeza ou reinstalacao
```

🟢 Em comandos operacionais, `Adulterado` encerra com codigo 1; `SemBaseline` e `HashOrfao` permitem continuidade com comportamento de compatibilidade. Hooks binarios nativos nao possuem script a verificar.

## Decisao de permissao e rewrite

```mermaid
stateDiagram-v2
    [*] --> AvaliarRegras
    AvaliarRegras --> PreservarNativo: deny corresponde
    AvaliarRegras --> Deferir: construto nao atestavel
    AvaliarRegras --> ProcurarRewrite: ask, allow ou default
    ProcurarRewrite --> Deferir: sem rewrite ou rewrite identico
    ProcurarRewrite --> ReescreverComAllow: allow e host suporta decisao
    ProcurarRewrite --> ReescreverComAsk: ask ou default
    PreservarNativo --> [*]
    Deferir --> [*]
    ReescreverComAllow --> [*]
    ReescreverComAsk --> [*]
```

🟢 `Deny` tem precedencia maxima. 🟢 Para cadeias compostas, todos os segmentos devem corresponder a allow para chegar a `ReescreverComAllow`. 🟢 Para Droid, allow nao e emitido: um deny/block faz `PreservarNativo`; demais rewrites nao carregam decisao de permissao.

## Elegibilidade e envio de telemetria

```mermaid
stateDiagram-v2
    [*] --> Desabilitada
    Desabilitada --> Elegivel: endpoint compilado, consentimento=true e enabled=true
    Elegivel --> Desabilitada: opt-out por env/config ou erro de config
    Elegivel --> AguardarJanela: marker menor que 23h
    Elegivel --> Marcada: janela disponivel
    Marcada --> EnvioAssincrono: marker gravado antes do envio
    EnvioAssincrono --> Elegivel: sucesso ou falha silenciosa
    AguardarJanela --> Elegivel: janela expira
```

🟢 O fluxo e fire-and-forget; falha no envio nao altera o fluxo principal do CLI. 🔴 A entrega HTTP e o endpoint nao foram exercitados nesta analise.
