# UV Setup and Guidelines (Ubuntu)

In Python projects, it is recommended to use [`uv`](https://docs.astral.sh/uv/) for dependency management.

## Setting up `uv`

Installing `uv`:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
On Ubuntu, `uv` is typically installed to `~/.local/bin`. If `uv` isn't found after installation, add this to your shell startup (usually `~/.bashrc` for terminal sessions, or `~/.profile` for login shells):
```bash
export PATH="$PATH:$HOME/.local/bin"
```
To append it automatically (pick one):
```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.bashrc
# or: echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.profile
```
Then reload your config:
```bash
source ~/.bashrc
```

## Running a `uv` Project

Syncing Python version, .venv, dependencies (optional command):
```bash
uv sync
```
Using `.venv` (optional command, as it should be automatic):
```bash
source ./.venv/bin/activate
```
Syncing and running project:
```bash
uv run main.py
```
Linting with ruff:
```bash
uvx ruff check .
```
```bash
uvx ruff format
```

## Setting up a `uv` Project

Creating `pyproject.toml` and `.python-version`:
```bash
uv init --bare
```
Adding package dependencies to `pyproject.toml`:
```bash
uv add [package-name]
```
Adding package dependencies from `requirements.txt` to `pyproject.toml`:
```bash
uv add -r requirements.txt
```
Adding ruff and pytest as development dependencies:
```bash
uv add ruff --dev
```
```bash
uv add pytest --dev
```
Generating `requirements.txt` from a UV lock file:
```bash
uv export -o requirements.txt
```
