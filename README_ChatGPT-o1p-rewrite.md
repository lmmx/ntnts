# ntnts (pronounced "intents")

<!-- Badges (placeholders shown below) -->
[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)
[![pdm-managed](https://img.shields.io/badge/pdm-managed-blueviolet)](https://pdm.fming.dev)
[![PyPI](https://img.shields.io/pypi/v/ntnts.svg)](https://pypi.org/project/ntnts)
[![Supported Python versions](https://img.shields.io/pypi/pyversions/ntnts.svg)](https://pypi.org/project/ntnts)
[![License](https://img.shields.io/pypi/l/ntnts.svg)](https://pypi.org/project/ntnts)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/lmmx/ntnts/master.svg)](https://results.pre-commit.ci/latest/github/lmmx/ntnts/master)

**ntnts** is a command-line tool and Python library for *execution-free, repository-grounded code-intent analysis*. Inspired by the MutaGReP approach, it:
- Reads files from either a **local directory** or a **GitHub repository** (using [`polars-ls`](https://github.com/lmmx/polars-ls) or optionally [`octopolars`](https://github.com/lmmx/octopolars)).
- **Generates “intents”** (high-level natural-language descriptions) for each *symbol* (class, function, etc.) using an LLM.
- **Embeds** these textual “intents” into **vector representations** with [`polars-fastembed`](https://github.com/lmmx/polars-fastembed).
- Outputs a Polars dataframe (or other format) for further exploration, filtering, or search.

## Installation

```bash
pip install ntnts
