# ck New-User Guide (Ubuntu): Code Search + LLM Conversation Search

This guide is for new users who want to use `ck` for two practical workflows:

1. **Code search** in Python and Go repositories.
2. **LLM conversation search** across prompt/chat transcript files.

It is based on the local official documentation in `./docs-site`.

---

## 1) Setup and installation on Ubuntu

## Recommended install path (NPM)

```bash
npm install -g @beaconbay/ck-search
ck --version
```

`ck` can also be installed via Cargo:

```bash
cargo install ck-search
ck --version
```

Or from source:

```bash
git clone https://github.com/BeaconBay/ck
cd ck
cargo install --path ck-cli
ck --version
```

## Ubuntu checklist before first semantic search

- Have working internet for first model download.
- Ensure free disk space (up to ~500MB for larger models).
- Ensure cache path is writable (`~/.cache/ck/models/` on Linux).

First semantic search auto-triggers model download + indexing:

```bash
ck --sem "error handling" src/
```

---

## 2) Quick-start workflows for your two use cases

## A. Python and Go code search

From repository root:

```bash
# One-time index build for the repo
ck --index .

# Python concepts
ck --sem "retry logic" .
ck --sem "async error handling" .

# Go concepts
ck --sem "context cancellation" .
ck --sem "goroutine leak prevention" .

# Exact names/symbols
ck --lex "NewClient" .
ck --lex "TODO" .

# Hybrid when you want semantic + keyword balance
ck --hybrid "http timeout middleware" .
```

## B. LLM conversation search

If your conversations are in text/markdown/jsonl files in your repo (for example `chats/`, `prompts/`, `notes/`), index and search them semantically:

```bash
ck --index .
ck --sem "user asked for rollback strategy" chats/
ck --sem "tool call failed due to permissions" chats/
ck --lex "function_call" chats/
```

For LLM tooling pipelines, prefer stream-friendly output:

```bash
ck --jsonl --sem "incident timeline" chats/
ck --jsonl --no-snippet --sem "latency regression" chats/
```

---

## 3) How to choose models, and what makes a model compatible with `--model`

## Available model names in docs

Commonly documented names:

- `bge-small` (default)
- `nomic-v1.5`
- `jina-code`
- `mxbai-xsmall`

Use them with:

```bash
ck --index --model <MODEL_NAME> .
```

## Compatibility checklist for `--model`

A model is compatible for practical use in `ck` when:

1. The model name is supported by the installed `ck` build/provider.
2. You can index successfully with it (`ck --index --model ...`).
3. The index metadata reports matching model/dimensions (`ck --status .`).
4. Search runs without embedding dimension mismatch errors.

If you switch models, `ck` rebuilds the index because each repo index is tied to one embedding model at a time.

## Model selection guidance for your use cases

- Start with **`bge-small`** for fastest setup and iteration.
- Prefer **`jina-code`** for code-structure-heavy Python/Go repositories.
- Prefer **`nomic-v1.5`** when long context/doc-heavy files matter.
- Try **`mxbai-xsmall`** when you want a local, balanced semantic model.

---

## 4) How to test model selection

Use this repeatable A/B workflow:

```bash
# Baseline model
ck --switch-model bge-small .
ck --status .
ck --sem --scores "request validation and auth middleware" .

# Candidate model
ck --switch-model jina-code .
ck --status .
ck --sem --scores "request validation and auth middleware" .
```

What to compare:

- Top results relevance for your real queries.
- Whether key files/functions appear in top results.
- Indexing time and local resource usage.

For chunking sanity checks before committing to a model:

```bash
ck --inspect --model bge-small src/main.py
ck --inspect --model nomic-v1.5 src/main.py
```

For LLM conversation corpora, run the same query set against your chat directory:

```bash
ck --switch-model bge-small .
ck --sem --scores "postmortem summary with action items" chats/

ck --switch-model nomic-v1.5 .
ck --sem --scores "postmortem summary with action items" chats/
```

---

## 5) Reindexing / refreshing `ck` (full and partial)

## Default behavior

- `ck` does incremental updates automatically using file hashing.
- In normal usage, you usually do **not** need full rebuilds.

## Full refresh options

```bash
# Explicit full rebuild
ck --clean .
ck --index .

# Equivalent model-driven rebuild
ck --switch-model nomic-v1.5 .
```

Use full refresh after:

- Model changes.
- Major ignore-rule changes (`.ckignore`, `.gitignore`) that should affect indexed scope.
- Corruption/synchronization issues.

## Partial refresh options

```bash
# Add/update a specific file in existing index
ck --add src/new_module.py

# Check whether index is stale and which model is active
ck --status .
```

## Ignore changes and refresh

After editing `.ckignore`, do a full rebuild so exclusions are applied cleanly:

```bash
ck --clean .
ck --index .
```

---

## 6) Project dependencies (what new users should plan for)

## Runtime / usage dependencies

- `ck` semantic search uses local embedding models (downloaded once, cached locally).
- No API keys required.
- CPU-only operation is supported (no GPU required).
- Internet is only needed for initial model download (then can be used offline).

## Install-time paths

- **NPM package**: `@beaconbay/ck-search`
- **Cargo package**: `ck-search`

## If building from source

- Rust toolchain (MSRV documented as Rust 1.89+ in contributing docs).
- Cargo and Git.

## Optional tooling dependencies (docs development only)

If you are modifying the docs site itself (`./docs-site`), use:

- Node.js 18+ or 20+
- pnpm 10+

---

## 7) Minimal operational playbook (recommended)

```bash
# 1) Install and verify
npm install -g @beaconbay/ck-search
ck --version

# 2) Index once
ck --index .

# 3) Run your two query styles
ck --sem "python retry with exponential backoff" .
ck --sem "conversation where assistant suggested rollback" chats/

# 4) If results need improvement, test a model switch
ck --switch-model jina-code .
ck --sem --scores "python retry with exponential backoff" .

# 5) If index drift is suspected
ck --status .
ck --clean . && ck --index .
```

This gives a safe default path for new users and a clear upgrade path for model tuning.
