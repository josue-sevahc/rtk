# Fluxograma — Modulo `scripts`

## Instalador Remoto

```mermaid
flowchart TD
    A[install.sh] --> B[detect_os]
    B --> C[detect_arch]
    C --> D[get_target]
    D --> E{RTK_VERSION definido?}
    E -- sim --> F[usar versao pinada]
    E -- nao --> G[resolver latest release]
    F --> H[baixar archive e checksums]
    G --> H
    H --> I{RTK_SKIP_CHECKSUM=1?}
    I -- nao --> J[verificar SHA-256]
    I -- sim --> K[warning bypass checksum]
    J --> L{checksum ok?}
    L -- nao --> M[erro e abortar]
    L -- sim --> N[validar paths do tar]
    K --> N
    N --> O{path traversal?}
    O -- sim --> M
    O -- nao --> P[extrair e instalar]
    P --> Q[verificar binario e PATH]
```

## Smoke Test Local

```mermaid
flowchart TD
    A[test-all.sh] --> B{rtk no PATH?}
    B -- nao --> C[exit 1]
    B -- sim --> D{dentro de git repo?}
    D -- nao --> C
    D -- sim --> E[executar secoes de comandos]
    E --> F[assert_ok / assert_contains / assert_fails]
    F --> G[contar PASS FAIL SKIP]
    G --> H[exit numero de falhas]
```

## Benchmark Em VM

```mermaid
flowchart TD
    A[run.ts] --> B[vmEnsureReady]
    B --> C{VM existe?}
    C -- nao --> D[multipass launch + cloud-init]
    C -- sim --> E[start se necessario]
    D --> F[wait ready]
    E --> F
    F --> G[transfer source tar]
    G --> H[cargo build --release]
    H --> I[fases de teste]
    I --> J[gerar report]
    J --> K{falhas == 0?}
    K -- sim --> L[READY FOR RELEASE]
    K -- nao --> M[NOT READY]
```

## Guard De Testes Inline

```mermaid
flowchart TD
    A[check-test-presence.sh] --> B[git diff base...HEAD]
    B --> C[filtrar src/cmds/*_cmd.rs]
    C --> D{ha arquivos?}
    D -- nao --> E[OK]
    D -- sim --> F[grep cfg(test)]
    F --> G{todos tem testes?}
    G -- sim --> E
    G -- nao --> H[FAIL exit 1]
```
