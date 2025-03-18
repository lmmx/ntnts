Below is an **adapted README** for **`ntnts`**, following a style similar to the old `octopolars` README while reflecting the new tool’s goals, features, and usage.

---

# ntnts (pronounced "intents")

[![pdm-managed](https://img.shields.io/badge/pdm-managed-blueviolet)](https://pdm.fming.dev)
[![PyPI](https://img.shields.io/pypi/v/ntnts.svg)](https://pypi.org/project/ntnts)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ntnts.svg)](https://pypi.org/project/ntnts)
[![License](https://img.shields.io/pypi/l/ntnts.svg)](https://pypi.org/pypi/ntnts)

> **Execution-free, repository-grounded code intent analysis.**
> Summarize code symbols with LLM-generated “intents” and embed them using Polars dataframes—leveraging optional local or GitHub-based file discovery.

---

## Overview

**`ntnts`** is a CLI tool (and importable Python library) for:
- **Collecting files** from a local repository (using [polars-ls](https://github.com/lmmx/polars-ls)) or from GitHub (using [octopolars](https://github.com/lmmx/octopolars))
- **Analyzing code** to discover symbols (classes, functions, etc.)
- **Generating “intents”** (short natural-language descriptions of each symbol’s purpose, typically 5 descriptions per symbol) using an LLM
- **Embedding** those textual intents into vectors (using [polars-fastembed](https://github.com/lmmx/polars-fastembed))

The end result is a Polars-based inventory of your codebase—each row representing a file or symbol, with columns for:
- **Symbol name and type**
- **Generated “intent” text** (or multiple variants)
- **Vector embeddings** for semantic search or clustering

This approach draws inspiration from [MutaGReP](https://arxiv.org/abs/2302.07822) to facilitate “repository-grounded” plan search for code usage. Instead of dumping entire codebases into an LLM, **`ntnts`** extracts code artifacts in a structured way and “grounds” them with a semantic representation.

---

## Installation

```bash
pip install ntnts
```

Like the original `octopolars`, **`polars`** itself is *optional* but strongly recommended. You can activate it via extras:

```bash
pip install ntnts[polars]          # for standard Polars builds
pip install ntnts[polars-lts-cpu]  # for older CPUs without modern instruction sets
```

### Requirements

- **Python 3.9+**
- **Local or remote code source**:
  - Local scanning is handled by [`polars-ls`](https://github.com/lmmx/polars-ls) (included by default).
  - GitHub scanning is possible if [`octopolars`](https://github.com/lmmx/octopolars) is also installed (an optional extra) and you have a GitHub token or the `gh` CLI for private/authenticated repositories.
- **LLM** (optional for certain features):
  - By default, you can use a local embedding method (via `polars-fastembed`).
  - Generating textual “intents” requires an LLM; you can integrate your own API key or local model (not strictly enforced by `ntnts`).

---

## Features

- **Execution-free code analysis**: No need to run the code or resolve imports. `ntnts` uses static methods (AST-like parsing or heuristics) to gather symbols and is designed to avoid heavy dependencies.
- **LLM-based symbol descriptions**: Create short “intent” statements describing each symbol’s purpose. (Default is 5 variants per symbol, which can be condensed into a single embedding.)
- **Vector embeddings**: Easily embed those “intents” with [polars-fastembed](https://github.com/lmmx/polars-fastembed). Store or export the resulting vectors for semantic search, clustering, or re-ranking.
- **Local or GitHub**: Point `ntnts` at a local directory or use [octopolars](https://github.com/lmmx/octopolars) to read from GitHub repos.
- **Lightweight polars-based**: Everything is loaded into Polars DataFrames for efficient filtering, sorting, output to CSV/Parquet/JSON, etc.
- **Pluggable**: Future integration with vector databases like Qdrant or LanceDB is planned but not required in the MVP.

---

## Usage

In the spirit of minimal CLI design, **`ntnts`** can be invoked for a variety of tasks with a single command. The underlying steps (scan, analyze, embed, retrieve) can be controlled by flags or arguments. Below is a high-level example.

### 1. Basic Command

```bash
ntnts path/to/local/repo
```

- **Scans** the given directory for code files (via `polars-ls`).
- **Analyzes** each code file for symbols (classes, functions, etc.).
- **Generates** textual intents for each symbol (if configured with an LLM).
- **Embeds** those intents (by default, with `polars-fastembed`).

This outputs a **Polars DataFrame** to your terminal by default, showing a summary of discovered symbols and partial textual “intents.” (You can override the output format with `--output csv|json|parquet|...`.)

### 2. Filtering or Retrieving by Query (Optional)

If you provide a `--query "..."` argument, **`ntnts`** will:
- Convert your query into a vector using the same embedding method.
- Perform a nearest-neighbor match among your code symbols’ embeddings.
- Return the top matches.

Example:

```bash
ntnts path/to/local/repo --query "How does the code handle user authentication?"
```

### 3. Setting Custom LLM or Embedding Parameters

```bash
ntnts . --model "openai/gpt-4" --embedding-model "sentence-transformers/all-MiniLM-L6-v2"
```

### 4. Controlling the Number of “Intents” per Symbol

```bash
ntnts . --intents 5
```

Generates 5 separate textual descriptions, then merges them into a single embedding row per symbol.

### 5. Output Formats

```bash
ntnts . --output json
```

Writes the final DataFrame as JSON. Similarly, `--output parquet` or `--output csv` is supported. You can also limit columns/rows, or use `--quiet` for an abbreviated preview, similar to the usage in `octopolars`.

---

## Example Workflow

1. **Analyze a Local Repository**

   ```bash
   ntnts ~/projects/my_repo --output parquet
   # 1) Collect files with polars-ls
   # 2) Identify code symbols
   # 3) Generate LLM-based 'intents'
   # 4) Embed them
   # => Writes a .parquet file containing symbol data + embeddings
   ```

2. **(Optional) GitHub Repository**

   If you install `ntnts` with the GitHub extra (which pulls in `octopolars`), you can do:

   ```bash
   ntnts https://github.com/myuser/myrepo
   # If a token or 'gh' CLI is configured, it can parse the remote repo’s files
   # and do the same analysis -> output a DataFrame
   ```

3. **Querying After Embedding**

   ```bash
   ntnts https://github.com/myuser/myrepo --query "JWT token verification" --top-k 3
   # Surfaces the top 3 symbols that match the concept of "JWT token verification"
   ```

---

## Library Usage

You can import parts of **`ntnts`** as a Python library if you prefer a programmatic approach:

```python
from ntnts import analyze_repo

# Analyze a local folder
df = analyze_repo("/path/to/my_repo", num_intents=5, model="openai/gpt-4")

# 'df' is a Polars DataFrame with columns like:
# file_path, symbol_name, symbol_type, intent_text, vector, ...
print(df)
```

You could then incorporate your own filtering, sorting, or advanced nearest-neighbor search (e.g., hooking up Qdrant or LanceDB if desired).

---

## Project Structure

- `cli.py` (or `__main__.py`): Defines the main **`ntnts`** CLI.
- `analyze.py`: Core logic to parse code symbols, generate or load LLM-based “intents,” return Polars DataFrame.
- `embed.py`: Utility to apply `polars-fastembed` for embedding textual data.
- `retrieve.py`: Optional similarity search routines.

Where possible, **caching**, **GitHub access**, and **local file listing** are offloaded to:
- [polars-ls](https://github.com/lmmx/polars-ls) for local scanning.
- [octopolars](https://github.com/lmmx/octopolars) for remote GitHub scanning (installed via extras).

---

## Contributing

Maintained by the same author as [octopolars](https://github.com/lmmx/octopolars). Contributions are welcome!

1. **Issues & Discussions**: Please open a GitHub issue or discussion for bugs, feature requests, or questions.
2. **Pull Requests**: PRs are welcome!
   - Install the dev extras: `pip install -e .[dev]`
   - Run tests (when available) and update docs or examples if relevant.
   - For any bug report, please include the package version and, if possible, a traceback or error message.

---

## License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).

---

### Why “ntnts”?

*(Q&A style note)*
**`ntnts`** is pronounced *“intents”*, highlighting that the tool’s main focus is extracting **intent** descriptions from code: short, high-level natural-language statements about each symbol’s purpose.

### Future Plans

- Deeper **AST-based** symbol extraction for more languages.
- **Multi-repo** aggregation for large-scale code analytics.
- Integrations with **Qdrant** or **LanceDB** for advanced vector search.
- Additional **LLM** options (local or remote) with easy plugin support.

These are under active development—there’s no fixed timeline, and **`ntnts`** remains **lightweight** in its MVP form, focusing on simple, execution-free code analysis.
