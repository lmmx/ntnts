# ntnts (pronounced "intents")

[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)
[![pdm-managed](https://img.shields.io/badge/pdm-managed-blueviolet)](https://pdm.fming.dev)
[![PyPI](https://img.shields.io/pypi/v/ntnts.svg)](https://pypi.org/project/ntnts)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ntnts.svg)](https://pypi.org/project/ntnts)
[![License](https://img.shields.io/pypi/l/ntnts.svg)](https://pypi.python.org/pypi/ntnts)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/lmmx/ntnts/master.svg)](https://results.pre-commit.ci/latest/github/lmmx/ntnts/master)

An execution-free, repository-grounded CLI for extracting meaningful "intents" from codebases, embedding them for semantic exploration.

Inspired by [MutaGReP](https://arxiv.org/abs/2311.15653), `ntnts` leverages [octopolars](https://pypi.org/project/octopolars), [polars-ls](https://pypi.org/project/polars-ls), and [polars-fastembed](https://pypi.org/project/polars-fastembed) to simplify semantic codebase understanding.

## Installation

```bash
pip install ntnts
```

**Polars dependency**: Required but provided as extras:

```bash
pip install ntnts[polars]          # standard Polars
pip install ntnts[polars-lts-cpu]  # for compatibility with older CPUs
```

### Requirements

- Python 3.9+
- Optional: [octopolars](https://github.com/lmmx/octopolars) for GitHub integration

`ntnts` optionally uses `octopolars` for pulling GitHub repositories. For authenticated GitHub access, configure via the `GH_TOKEN` environment variable or GitHub CLI.

## Features

- **Repository analysis**: Extract code symbols (functions, classes, enums) from local or GitHub repositories.
- **Purpose-oriented intents**: Generate action-oriented natural language descriptions (intents) for symbols using LLMs (default: 5 intents per symbol).
- **Embedding generation**: Create efficient vector embeddings of intents using `polars-fastembed`.
- **Dataframe output**: Structured Polars dataframe outputs for further analysis, retrieval, or ranking.

## Usage

### Basic CLI Usage

```bash
ntnts path/to/local/repo
```

- Extracts symbols, generates intents, and produces embeddings in a dataframe.

### Example: Retrieve semantic matches

```bash
ntnts path/to/repo --query "database modification" --top-k 5
```

- Retrieves top 5 files most semantically relevant to modifying databases.

### Output formats

Default outputs directly to terminal. Other formats (JSON, Parquet) available via flags:

```bash
ntnts path/to/repo --output json
ntnts path/to/repo --output parquet
```

## Architecture Overview

- **Discovery**: Uses `polars-ls` or `octopolars` to parse repositories.
- **Symbol Intent Generation**: Uses LLM APIs (user-provided API keys, local or remote) for generating textual intents.
- **Embedding**: Employs `polars-fastembed` for vector embeddings.

## Future ideas

- Optional Qdrant and LanceDB integrations for large-scale semantic indexing.
- Multi-repo embedding aggregation to enhance semantic exploration across multiple codebases.

## Library usage

You can use the `ntnts` library programmatically:

```python
from ntnts import embed_repo

# Generate embeddings for a local repository
df = embed_repo("path/to/repo")
```

## Dependencies and optional integrations

- Core: `polars`, `polars-ls`, `polars-fastembed`
- Optional: `octopolars` for GitHub repositories.

## Contributing

Maintained by [lmmx](https://github.com/lmmx). Contributions are welcome!

1. Open issues or discussions for bugs, features, or questions.
2. Submit pull requests:
   - Install dev extras (e.g., via [uv](https://docs.astral.sh/uv/)): `uv pip install -e .[dev]`
   - Ensure test suite passes and documentation is updated as necessary.

## License

Licensed under the [MIT License](https://opensource.org/licenses/MIT).
