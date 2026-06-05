# codegraph reference

[codegraph](https://github.com/colbymchenry/codegraph) is a local, pre-indexed code-graph tool that gives coding agents semantic understanding of a codebase — symbols, call relationships, and impact radius — without external services or API keys. It supports 20+ languages (TypeScript, Python, Go, Rust, Java, C#, Swift, Kotlin, …) and is framework-aware (Django, Flask, FastAPI, Express, NestJS, Rails, Spring).

This skill uses it as an **optional accelerator**: when present, querying the graph replaces repeated grep/read passes; when absent, the skill falls back to Glob/Grep/Read.

## Install

```bash
# macOS / Linux
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh

# Windows (PowerShell)
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 | iex

# Or via npm
npm i -g @colbymchenry/codegraph
```

Verify with `command -v codegraph`.

## Index a project

Run once per repo; the native file watcher (FSEvents/inotify) keeps it current afterward with debounced updates.

```bash
codegraph init      # initialize the project index
codegraph index     # build / rebuild the knowledge graph (first run can be slow on large repos)
```

## Query commands (the fast path for this skill)

| Command | Use it for |
| --- | --- |
| `codegraph query <symbol\|text>` | Locate definitions/symbols fast (SQLite FTS5 full-text). Start here. |
| `codegraph callers <symbol>` | Who calls this — raw material for sequence/data-flow diagrams. |
| `codegraph callees <symbol>` | What this calls — trace outward flow. |
| `codegraph impact <symbol>` | Blast radius before a change ("what breaks if I touch X"). |

Output includes verbatim source grouped by file, relationship maps, and blast-radius analysis, in both human-readable and JSON form. Always Read the cited lines to confirm before quoting them in an explanation.

## MCP server (alternative to shelling out)

Expose the same capabilities as native MCP tools instead of CLI calls:

```bash
codegraph serve --mcp
```

To register it with Claude Code automatically:

```bash
codegraph install   # configures supported agents (Claude Code, Cursor, Codex, …)
```

After that, the codegraph MCP tools are available directly and you don't need Bash to call the CLI.

## Caveats

- **Stale index.** If results disagree with the file on disk, re-run `codegraph index` or fall back to grep. Never quote graph output you haven't confirmed against current source.
- **Unindexed project.** Queries error until `codegraph init` + `codegraph index` have run.
- **Generated/vendored code** is indexed like anything else; still skip it when picking "most important files."
