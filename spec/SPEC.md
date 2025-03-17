Below is a sample specification document for a command-line tool named **`ntnts`**. This spec outlines high-level goals, architecture, commands, and extensibility points for building an “execution-free, repository-grounded” code-intent analysis tool. The approach is MutaGReP-style in that it gathers symbolic information from source files, generates purpose-driven “intents” (in natural language) for each symbol, and embeds them for later searching or referencing.

---

## 1. Overview

**Tool Name:** `ntnts`
**Primary Goal:**
- Provide a CLI for extracting code artifacts (files, classes, functions) from a GitHub repository or a local repository path.
- Generate “intents” (high-level natural-language descriptions of the purpose of each symbol) using LLMs.
- Create vector embeddings of those intents (using `polars-fastembed`).
- Output or store these [natural language description, vector embedding] pairs for further analysis and search.

**Key Technical Points:**
1. **Input Sources:** Either a public/private GitHub repository, or a local directory structure.
2. **Discovery:** Use utilities such as `polars-ls` and `octopolars` to collect file and symbol information.
3. **LLM-based Summaries (Intents):** Use a large language model (LLM) to parse and describe the purpose of each symbol.
4. **Embedding:** Use `polars-fastembed` to embed the generated text.
5. **Storage / Indexing:** Initially store embeddings locally (e.g., in CSV/Parquet). Optional future integrations with Qdrant or LanceDB for advanced indexing.

**Rationale:**
- By generating discrete “intents” (natural language clarifications of each symbol’s role), developers and tools can more easily browse or search a codebase based on semantics, not just textual matching.
- The approach is “execution-free” in the sense that we are analyzing static code rather than running it.
- The end output is a structured representation of each codebase artifact (symbol name, location, descriptive intent, vector embedding).

---

## 2. Command-Line Interface Specification

### 2.1. Base Command

```bash
ntnts
```

When run with no arguments or subcommand, it should display a help message summarizing the usage and the available subcommands.

### 2.2. Subcommands

1. **`ntnts scan [OPTIONS] <REPO_OR_PATH>`**
   - **Description:** Collects code artifacts from the specified repository (local or remote).
   - **Key Steps:**
     1. If `<REPO_OR_PATH>` is a GitHub URL, clone or retrieve metadata (using `octopolars` or a similar approach).
     2. If `<REPO_OR_PATH>` is a file system path, parse the directory structure (potentially using `polars-ls`) to list files and subdirectories.
     3. Identify code files by configurable file extensions (e.g., `.py`, `.js`, `.rs`, etc.).
     4. Output a local index of discovered files (e.g., `ntnts_scan_output.json` or `.parquet`).
   - **Options/Flags:**
     - `--extensions <ext1,ext2,...>`: Comma-separated list of file extensions to include (default: standard language defaults).
     - `--output <path>`: Path to store the discovered index file (default: in the working directory).
     - `--include-dotfiles`: Include hidden or dotfiles in the scan (optional).

2. **`ntnts analyze [OPTIONS] <INDEX_FILE>`**
   - **Description:** For each file in the index (produced by `ntnts scan`), parse out major symbols (classes, functions, etc.).
   - **Key Steps:**
     1. Traverse each file in the index.
     2. Parse the file (ideally hooking into a language-specific parser or a universal multi-language approach).
     3. Extract symbol definitions (classes, enums, data structures, methods, functions, etc.).
     4. Generate short textual “intents” using an LLM (e.g., “This function handles user authentication by verifying JWT tokens,” etc.).
     5. Store the result in an intermediate file (e.g., `ntnts_analyze_output.parquet`).
   - **Options/Flags:**
     - `--lang <language>`: Force a language mode if autodetection fails.
     - `--model <modelNameOrPath>`: Specify the LLM to use for generating text.
     - `--output <path>`: Path to store the result of analysis.
     - `--batch-size <N>`: Process files in batches to manage memory or API usage.

3. **`ntnts embed [OPTIONS] <ANALYSIS_FILE>`**
   - **Description:** Uses `polars-fastembed` to transform each “intent” text into a vector embedding.
   - **Key Steps:**
     1. Reads the `ANALYSIS_FILE` (the output of `ntnts analyze`).
     2. For each symbol’s “intent” text, calls the embedding method.
     3. Merges the vector embeddings back into the data structure.
     4. Outputs a combined artifact (e.g., `ntnts_vectors.parquet`).
   - **Options/Flags:**
     - `--embedding-model <modelNameOrPath>`: Choose the model used in `polars-fastembed`.
     - `--dims <integer>`: Dimension size for the embeddings (if configurable).
     - `--output <path>`: Where to write the final embedded data.
     - `--batch-size <N>`: Batch the embedding to handle large codebases.

4. **`ntnts search [OPTIONS] <EMBEDDED_DATA> <QUERY>`** *(Planned or future extension)*
   - **Description (Hypothetical Future Use):** Takes a user query in natural language, converts it to an embedding, then searches the embedded code-intent data for nearest matches. This is where Qdrant or LanceDB might eventually come in.
   - **Key Steps:**
     1. Convert `<QUERY>` to an embedding.
     2. Search among the `<EMBEDDED_DATA>` for nearest neighbors.
     3. Return top matching symbols with their source file references and original intent text.
   - **Options/Flags:**
     - `--top-k <K>`: Number of matches to return (default: 5).
     - `--engine <engineName>`: (Future) Possibly `qdrant`, `lancedb`, or an in-memory search.

---

## 3. Configuration and Environment

- **Configuration File (Optional)**: `~/.ntnts/config.toml` or a `.ntnts` file at the project root. This might store defaults like:
  - Preferred LLM model names
  - Preferred embedding model
  - File extension filter
  - Qdrant or LanceDB connection details for advanced usage

- **Environment Variables**:
  - `NTNTS_TOKEN` or `OPENAI_API_KEY` (if the LLM requires an API token).
  - `NTNTS_EMBED_MODEL_PATH` (optional override for local embedding model).

---

## 4. Architecture

### 4.1. Process Flow

1. **Discovery**:
   - Execute `ntnts scan` to gather all relevant files.
   - Output is a structured list of files (and possibly basic metadata).

2. **Symbol Analysis & Intent Generation**:
   - Execute `ntnts analyze` on the discovery output.
   - A static parser or heuristics identify major code symbols.
   - An LLM is used to create a short textual “intent” describing each symbol’s function/purpose.
   - Output is a table or dataset of `[file, symbol, intent]`.

3. **Embedding**:
   - Execute `ntnts embed` to convert each “intent” into a vector using `polars-fastembed`.
   - Output is a combined dataset of `[file, symbol, intent, vector_embedding]`.

4. **(Optional Future) Searching**:
   - `ntnts search` to find symbols by semantic similarity to a user query.

### 4.2. Data Structures

- **File Index Schema** (post-scan)
  ```text
  {
    "file_path": string,
    "relative_path": string,
    "language": string,
    ... // additional file-level metadata
  }
  ```

- **Symbol Analysis Schema** (post-analyze)
  ```text
  {
    "file_path": string,
    "symbol_name": string,
    "symbol_type": string,     // e.g. "class", "function", "enum"
    "intent_text": string,     // LLM-generated high-level description
    ... // additional symbol metadata
  }
  ```

- **Embedding Schema** (post-embed)
  ```text
  {
    "file_path": string,
    "symbol_name": string,
    "intent_text": string,
    "vector": list<float>, // The embedded representation
    ... // additional indexing or DB linking
  }
  ```

---

## 5. Dependencies and Extensions

1. **Core Dependencies (MVP):**
   - `polars` (for data handling)
   - `polars-ls` (directory listing / scanning)
   - `octopolars` (GitHub integration, if pulling from remote repos)
   - `polars-fastembed` (embedding generation)
   - LLM or wrapper library for generating text (e.g., OpenAI, local GPT, etc.)

2. **Optional / Future Dependencies:**
   - **Qdrant** or **LanceDB** for vector storage and nearest-neighbor searches. (Not strictly required in the MVP, but planned for advanced usage.)
   - **Other language-specific parsing libraries** (for robust symbol extraction beyond heuristics).

---

## 6. Example Workflow

1. **Scan a local repo**
   ```bash
   ntnts scan --extensions py,rs ~/projects/my_repo
   # => Produces ntnts_scan_output.json
   ```

2. **Analyze symbols in discovered files**
   ```bash
   ntnts analyze ntnts_scan_output.json --lang python --model "openai/gpt-4"
   # => Produces ntnts_analyze_output.parquet
   ```

3. **Embed the generated intents**
   ```bash
   ntnts embed ntnts_analyze_output.parquet --embedding-model "sentence-transformers/all-MiniLM-L6-v2"
   # => Produces ntnts_vectors.parquet
   ```

4. **(Future) Search**
   ```bash
   ntnts search ntnts_vectors.parquet "How does the code handle user authentication?"
   # => Returns top matched symbols, with file references and intent_text
   ```

---

## 7. Future Roadmap

- **Automatic Language Detection**: Enhanced scanning with multi-language detection.
- **Refined Symbol Parsing**: Integration with AST-based parsing for Python, Java, C++, etc.
- **Interactive AI-Assistant**: Chat-based summarization or Q&A on the codebase.
- **Embeddings Database**: Qdrant or LanceDB integration for large-scale indexing.
- **Multi-Repo Aggregation**: Merging embeddings from multiple projects into a single knowledge base.

---

## 8. Security and Privacy Considerations

- LLM calls may involve sending code snippets or summaries to a 3rd-party API, so caution is needed with proprietary code.
- Local embeddings should be handled in a secure manner; embeddings can contain sensitive information if the text is sensitive.
- For private GitHub repos, ensure proper authentication handling (SSH keys, tokens, etc.).

---

## 9. Conclusion

The **`ntnts`** tool aims to provide a straightforward CLI workflow to **discover**, **analyze**, and **embed** code artifacts from either GitHub or local repositories. By generating purpose-oriented “intents” for each symbol, developers can build a semantic understanding of a codebase, enabling more powerful search and navigation in the future. The MVP architecture focuses on local scanning and embedding, with an eye toward later integration with vector databases such as Qdrant or LanceDB.
