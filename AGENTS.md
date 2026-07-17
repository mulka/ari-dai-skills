# AGENTS.md

## Cursor Cloud specific instructions

### What this repo is
`dai-skills` is a **markdown skills library** for AI IDEs plus a small amount of executable code. There is **no `package.json`** and the Node code is **zero-dependency** (Node.js built-ins only). Node 18+ is required (the VM ships v22).

- **Primary app:** `skills/sdd-conductor/sdd-conductor.mjs` — the SDD lifecycle enforcement CLI. Run any command with `node skills/sdd-conductor/sdd-conductor.mjs <command>` (run with no args for the command list).
- **Installer:** `bash install.sh <skill|--all> --project <dir> [--ide claude|cursor|copilot] [--copy]` copies/symlinks skill files into a target project. See `README.md`.
- **Only automated test suite:** `skills/all-dai-sdd/test_verify_gates.py` (Python). Run with `python3 -m pytest skills/all-dai-sdd/test_verify_gates.py`. It needs `pytest`, `pyyaml`, and `z3-solver` (installed by the update script).

### Running local sdd-conductor gates offline (non-obvious)
Local, network-free gates such as `gate tiptap-html <file>` (the page publish gate referenced in `CLAUDE.md`) and `gate no-mocks <file>` still require two things before they will run:

1. A `.sdd-state.json` at the git root with a `currentInitiative` and matching `initiatives` entry. `node sdd-conductor.mjs init` creates this but needs a real API key + network; for offline/local gate checks you can write a minimal stub, e.g.:
   ```json
   { "version": "1.2.3", "currentInitiative": "x", "initiatives": { "x": { "dsId": "ds_local", "dsUri": "local", "planModeId": "pm_local", "initiative": "x", "statusGroups": {} } } }
   ```
   `.sdd-state.json` is not gitignored — do not commit the stub.
2. `DATASPHERES_API_KEY` must be present in `~/.dataspheres.env` or a repo-root `.env` file. **A bare exported shell env var is NOT picked up** unless that key already exists in one of those files (`loadEnv()` only overlays `process.env` onto keys read from a file). The offline gates make no network call, so a dummy value (e.g. `dsk_dummy`) is fine.

### API-backed commands
Commands like `init`, `drive`, `sync`, `start`, `complete`, `validate`, `dashboard-check`, and the `research/pages/newsletters/...` skill workflows call the Dataspheres REST API and require a **real** `DATASPHERES_API_KEY` (get one at https://dataspheres.ai/app/developers?tab=keys) plus network access. Without a real key these will fail at the network/auth step — that is expected, not an environment problem.

### Known pre-existing issues (not environment problems)
- `python3 -m pytest skills/all-dai-sdd/test_verify_gates.py` → **3 failed, 5 passed**. The source `verify_gates.py` now enforces 14 rules (added RULE-9/10/11/12 for Origin Prompts / Search Results / Codebase Context) and reports "14 rules verified", but `test_verify_gates.py` still expects "8 rules" and builds fixtures without the new required sections. This is stale test/code drift in the repo, not a setup issue.
- `.github/workflows/ci.yml` is **stale**: it runs `ruff`/`mypy`/`pytest` against a Python package at `dai/` and `tests/` that was removed from the repo. Those paths no longer exist, so that CI job does not reflect the current codebase.
