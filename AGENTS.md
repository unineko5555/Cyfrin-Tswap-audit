# Repository Guidelines

## Project Structure & Module Organization
- `src/`: Solidity sources (`PoolFactory.sol`, `TSwapPool.sol`).
- `test/`: Foundry tests
  - `unit/` and `unit/invariant/` (fuzz + invariants).
- `script/`: Deploy and utility scripts (e.g., `DeployTSwap.t.sol`).
- `lib/`: External deps installed by Foundry (e.g., OZ, forge-std).
- `out/`, `cache/`: Build artifacts (ignored in git).
- Root: `foundry.toml`, `Makefile`, `slither.config.json`, `README.md`.

## Build, Test, and Development Commands
- `make build` / `forge build`: Compile contracts to `out/`.
- `make test` / `forge test -vv`: Run unit, fuzz, and invariant tests.
- `make format` / `forge fmt`: Auto-format code per `foundry.toml`.
- `make coverage-report` / `forge coverage`: Generate coverage (debug report saved to `coverage-report.txt`).
- `make anvil`: Start local chain with a deterministic mnemonic.
- `make slither`: Run Slither static analysis using `slither.config.json`.
- `make update`: Update libraries in `lib/`.

## Coding Style & Naming Conventions
- Use `forge fmt` before committing. Config highlights: 4-space tabs, line length 120, double quotes, bracket spacing enabled.
- Solidity naming: contracts/types `PascalCase`, functions/variables `camelCase`, constants/immutables `UPPER_SNAKE_CASE`.
- Files: one contract per file; file name matches main contract (e.g., `TSwapPool.sol`).
- Keep external/public interfaces stable; document breaking changes in PR description.

## Testing Guidelines
- Framework: Foundry (`forge-std`). Tests live under `test/unit` and `test/unit/invariant`.
- Write positive, negative, and edge-case tests; prefer invariant tests where applicable.
- Use deterministic fuzzing (seed set in `foundry.toml`); reproduce with `forge test -vvvv`.
- Naming: test files `ContractName.t.sol`; test funcs `test...`, setup in `setUp()`.

## Commit & Pull Request Guidelines
- Commits: short, imperative subject lines (e.g., "fix: handle zero liquidity"). Keep focused and atomic.
- PRs: include purpose, summary of changes, risks, and testing evidence (commands + outputs). Link issues where relevant and update `README.md` if behavior changes.
- Prefer small, reviewable PRs. Add screenshots or logs when helpful.

## Security & Configuration Tips
- Do not commit secrets. `.env`, `out/`, `cache/`, and broadcast logs are ignored; keep it that way.
- Use `make slither` before security-related PRs. For local validation, run `make anvil` and scripts in `script/`.
- Solidity version is pinned to `0.8.20` in `foundry.toml`; avoid upgrading without coordination.
