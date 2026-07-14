# Copilot Instructions for rtk

**rtk (Rust Token Killer)** is a CLI proxy that filters and compresses command outputs before they reach an LLM context, saving 60-90% of tokens. It wraps common tools (`git`, `cargo`, `grep`, `pnpm`, `go`, etc.) and outputs condensed summaries instead of raw output.

## Using rtk in this session

**Always prefix commands with `rtk` when running shell commands** — this reduces token consumption for every operation you perform.

```bash
# Instead of:              Use:
git status                 rtk git status
git log -10                rtk git log -10
cargo test                 rtk cargo test
cargo clippy --all-targets rtk cargo clippy --all-targets
grep -r "pattern" src/     rtk grep -r "pattern" src/
```

**rtk meta-commands** (always use these directly, no prefix needed):
```bash
rtk gain              # Show token savings analytics
rtk gain --history    # Full command history with per-command savings
rtk discover          # Scan session history for missed rtk opportunities
rtk proxy <cmd>       # Run a command raw (no filtering) but still track it
```

**Verify rtk is installed before starting:**
```bash
rtk --version   # Should print: rtk X.Y.Z
rtk gain        # Should show a dashboard (not "command not found")
```

> Name collision: `rtk gain` failing means you have `reachingforthejack/rtk` (Rust Type Kit) installed instead. Run `which rtk` to check.

## Build, Test & Lint

```bash
cargo build                    # Development build
cargo test                     # All tests
cargo test test_name           # Single test
cargo test module::tests::     # Module tests
cargo test -- --nocapture      # With stdout

# Pre-commit gate (must all pass before any PR)
cargo fmt --all --check && cargo clippy --all-targets && cargo test

bash scripts/test-all.sh       # Smoke tests (requires installed binary)
```

PRs target the **`develop`** branch, not `main`. All commits require a DCO sign-off (`git commit -s`).

## Architecture

rtk routes CLI commands via a Clap `Commands` enum in `main.rs` to specialized filter modules in `src/cmds/*/`, each executing the underlying command and compressing output. Token savings are tracked in SQLite via `src/core/tracking.rs`.

For full details see [ARCHITECTURE.md](../docs/contributing/ARCHITECTURE.md) and [docs/contributing/TECHNICAL.md](../docs/contributing/TECHNICAL.md). Module responsibilities are documented in each folder's `README.md` and each file's `//!` doc header.

## Key Conventions

- **Error handling**: `anyhow::Result` with `.context("description")?` — no bare `?`, no `unwrap()` in production. Filters must fall back to raw command on error.
- **Regex**: Always `lazy_static!`, never compile inside a function body.
- **Testing**: Unit tests inside modules (`#[cfg(test)] mod tests`). Fixtures in `tests/fixtures/`. Token savings assertions with `count_tokens()`.
- **Exit codes**: Preserve the underlying command's exit code via `std::process::exit(code)`.
- **Performance**: Startup <10ms (no async runtime), binary <5MB stripped.
- **Branch naming**: `fix(scope):`, `feat(scope):`, `chore(scope):` where scope is the affected component.

For the full contribution workflow, design philosophy, and new-filter checklist, see [CONTRIBUTING.md](../CONTRIBUTING.md).


---

# Reversa

> Framework de Engenharia Reversa instalado neste projeto.

## Como usar

Use o fluxo adequado no chat:

- `/reversa` — descobrir e documentar um sistema existente
- `/reversa-new` — criar PRD e specs para um projeto novo
- `/reversa-forward` — implementar ou evoluir código a partir das specs
- `/reversa-migrate` — planejar a migração de um sistema legado
- `/reversa-docs` — gerar o mini-site visual da documentação
- `/reversa-agents-help` — consultar o catálogo completo de agentes

## Comportamento ao ativar

Quando o usuário digitar `/reversa` ou a palavra `reversa` sozinha em uma mensagem:

1. Ative o skill `reversa` disponível em `.agents/skills/reversa/SKILL.md`
2. Leia o SKILL.md na íntegra e siga exatamente as instruções do Reversa

## Regra não-negociável

Nunca apague, modifique ou sobrescreva arquivos pré-existentes do projeto legado.
O Reversa escreve apenas em `.reversa/`, `_reversa_sdd/`, `_reversa_docs/` e `_reversa_forward/`.
