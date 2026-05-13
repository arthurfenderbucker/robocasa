# RoboCasa Fork Notes

This fork (`arthurfenderbucker/robocasa`) extends the upstream
[`robocasa/robocasa`](https://github.com/robocasa/robocasa) with a uv-managed
install layout so it can sit alongside the
[`policy_abstraction`](https://github.com/arthurfenderbucker/policy_abstraction)
project without polluting the root venv. One feature commit ahead of
`upstream/main`.

## Changes vs upstream

| File | Change | Purpose |
|---|---|---|
| `pyproject.toml` (new) | Defines a Python 3.11 project with all runtime deps pinned, plus the websocket transport deps (`websockets`, `msgpack-numpy`, `tyro`, `loguru`) needed by `policy_abstraction/scripts/servers/serve_robocasa_env.py` | Replaces the upstream's `setup.py`-only install. Lets `uv sync` create a self-contained `.venv` under `third_party/robocasa/`. |
| `[tool.uv.sources]` | Pulls `robosuite` from `git+https://github.com/ARISE-Initiative/robosuite@master` | Avoids vendoring robosuite as a submodule; uv handles the editable install |
| `uv.lock` (new) | uv lockfile | Reproducible installs across machines |

The upstream `setup.py` is left untouched; `pyproject.toml` takes precedence
under uv.

## Why a standalone venv

The root `policy_abstraction` venv pins `numpy==1.23.3`, while robocasa
requires `numpy==2.2.5`. Installing robocasa into the root venv would replace
~30 packages and break the perception/policy stack. The fork ships its own
`.venv` (created with `uv sync` from inside this directory) and the robocasa
env server is launched from this venv via the `start_robocasa_env_server`
helper in `policy_abstraction/envs/robocasa/utils.py`.

## Usage

```bash
cd third_party/robocasa
uv sync                       # creates .venv with all deps
```

Then launch the env server from the parent repo:

```bash
# from the policy_abstraction repo root
uv run python scripts/servers/serve_robocasa_env.py --task PnPCounterToCab
```

See [`policy_abstraction/docs/install_robocasa.md`](https://github.com/arthurfenderbucker/policy_abstraction/blob/main/docs/install_robocasa.md)
for the full setup, including env server launch and dataset placement.
