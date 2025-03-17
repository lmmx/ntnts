# ntnts (pronounced "intents")

[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)
[![pdm-managed](https://img.shields.io/badge/pdm-managed-blueviolet)](https://pdm.fming.dev)
[![PyPI](https://img.shields.io/pypi/v/ntnts.svg)](https://pypi.org/project/ntnts)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ntnts.svg)](https://pypi.org/project/ntnts)
[![License](https://img.shields.io/pypi/l/ntnts.svg)](https://pypi.python.org/pypi/ntnts)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/lmmx/ntnts/master.svg)](https://results.pre-commit.ci/latest/github/lmmx/ntnts/master)

Extract code symbols, generate purpose-driven intents, and create vector embeddings for repository exploration with Polars.

## Installation

```bash
pip install ntnts
```

> The `polars` dependency is required but not included in the package by default.
> It is shipped as an optional extra which can be activated by passing it in square brackets:
> ```bash
> pip install ntnts[polars]          # most users can install regular Polars
> pip install ntnts[polars-lts-cpu]  # for backcompatibility with older CPUs
> ```

### Optional GitHub Support

For GitHub repository analysis, you can install with the GitHub extra:

```bash
pip install ntnts[github]  # Includes octopolars for GitHub integration
```

### Requirements

- Python 3.9+
- For GitHub integration: [gh](https://cli.github.com/) GitHub CLI tool, for a [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) to avoid rate limits and enable file listings.
    - **alternatively** set the `GH_TOKEN` environment variable as your GitHub token.

ntnts is supported by:

- [Polars](https://www.pola.rs/) for efficient data filtering and output formatting.
- [polars-ls](https://github.com/lmmx/polars-ls) for directory listing and local file enumeration.
- [polars-fastembed](https://github.com/pola-rs/polars-fastembed) for generating vector embeddings.
- [octopolars](https://github.com/lmmx/octopolars) (optional) for GitHub repository access.

## Features

- **Execution-free, repository-grounded analysis**: Analyze source code without executing it, grounding in the repository's symbols.
- **Symbol mining**: Extract code symbols (classes, functions, methods) via AST parsing.
- **Intent generation**: Create purpose-driven natural language descriptions (typically 5 variants) for each symbol.
- **Vector embeddings**: Condense intents into vector embeddings for semantic searching and exploration.
- **Output formats**: Display data in a Polars repr table (which can be [read back in](https://docs.pola.rs/api/python/stable/reference/api/polars.from_repr.html)) or export to JSON.
- **Integration**: Builds on established tools like polars-ls, polars-fastembed, and optionally octopolars.

## Motivation

`ntnts` is inspired by the [MutaGReP paper](https://arxiv.org/abs/2402.18071), which proposes a novel approach to code understanding by mining symbols, generating purpose-driven intents, and using these for downstream tasks. Traditional approaches to code analysis often struggle with large codebases, either failing to maintain context or requiring execution environments.

This tool offers an "execution-free" approach to understanding codebases by:
1. Extracting symbols without running the code
2. Generating human-readable "intents" describing each symbol's purpose
3. Creating vector embeddings to enable semantic search and exploration

The result is a structured dataframe where rows correspond to files in the repository, with arrays of intents and their vector embeddings, enabling powerful exploration and retrieval.

## Usage

### Command-Line Interface

```sh
$ ntnts --help
Usage: ntnts [OPTIONS] [PATH]

  Extract code symbols, generate intents, and create embeddings from repositories.

  By default, this processes a local directory path.

    The --github/-g flag enables GitHub repository analysis (requires octopolars).

    The --query/-q flag performs retrieval with the given query.

    The --k flag controls how many results to return in retrieval (default: 5).

    The --output/-o flag controls the output format (default: terminal display).

  Examples
  --------

  - Process a local repository

      ntnts .

  - Process and query a local repository

      ntnts . -q "database connection handling"

  - Process a GitHub repository (requires octopolars)

      ntnts -g username/repo

Options:
  -g, --github                Use GitHub repository instead of local path
  -q, --query TEXT           Query for retrieval
  -k INTEGER                 Number of results to return (default: 5)
  -o, --output-format TEXT   Output format: table, json (default: table)
  --help                     Show this message and exit.
```

## Project Structure

- `cli.py`: Defines the CLI (`ntnts`) with all available options and flags.
- `symbols.py`: Logic for extracting symbols from source code.
- `intents.py`: Generation of purpose-driven descriptions for symbols.
- `embeddings.py`: Integration with polars-fastembed for vector creation.

## Contributing

Maintained by [lmmx](https://github.com/lmmx). Contributions welcome!

1. **Issues & Discussions**: Please open a GitHub issue or discussion for bugs, feature requests, or questions.
2. **Pull Requests**: PRs are welcome!
   - Install the dev extra (e.g. with [uv](https://docs.astral.sh/uv/): `uv pip install -e .[dev]`)
   - Run tests (when available) and include updates to docs or examples if relevant.
   - If reporting a bug, please include the version and the error message/traceback if available.

## License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).

## Future Developments

- **Advanced retrieval mechanisms**: Integration with vector databases like Qdrant or LanceDB
- **Multi-repo aggregation**: Ability to merge embeddings from multiple repositories for comprehensive analysis
- **Enhanced language support**: Expand symbol extraction to additional programming languages
- **Interactive exploration**: Tools for visualizing and navigating the embedded code space
