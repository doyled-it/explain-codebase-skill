---
name: explain-codebase
description: Explain how this codebase works using ASCII diagrams plus prose. Use when the user asks to understand or be onboarded to the code — "how does X work," "walk me through," "explain the architecture/flow," "where does X happen," "trace this request," "give me an overview," or any "why is it built this way" question. Distinct feature: answers are illustrated with monospace ASCII diagrams (flows, trees, sequences, layers, state machines) and cite specific files and line numbers.
allowed-tools: Read, Glob, Grep, Bash
---

# Codebase Teacher

Answer questions about the codebase with clear explanations and ASCII visualizations.

## Behavior

Defaults like searching and reading code are assumed. What this skill adds:

- **Lead with a one-sentence answer**, then expand.
- **Always include an ASCII diagram** for anything with structure, flow, or state (see `references/diagram-toolkit.md`) — this is the point of the skill, not an optional extra.
- **Cite specific files and line numbers** (`path:line`), verified against the current file.
- **End with 2-4 concrete follow-up questions** the user is likely to want next.
- **Be candid about bad design.** If something is confusing, inconsistent, or a likely footgun, say so plainly rather than narrating it as intentional.

## Exploring the codebase

Before falling back to repeated grep/read passes, check for **codegraph** — a local, pre-indexed code graph that answers structural questions (symbols, callers, callees, impact) in one query. It typically cuts exploration tokens and tool calls roughly in half on a well-indexed repo. Use it as the fast path; grep/read remain the always-available fallback.

1. **Detect:** `command -v codegraph`. If it isn't installed, explore with Glob/Grep/Read as usual and skip the rest of this section.
2. **Ensure an index exists:** if a query reports the project isn't indexed, run `codegraph init` then `codegraph index` (the first index can take a while on large repos; it auto-syncs afterward).
3. **Prefer graph queries for structure:**
   - `codegraph query <symbol|text>` — locate definitions and symbols fast (full-text).
   - `codegraph callers <symbol>` / `codegraph callees <symbol>` — trace call flow; this is your raw material for sequence and data-flow diagrams.
   - `codegraph impact <symbol>` — blast radius, for "what breaks if I change X" questions.
   Build the ASCII diagram from this output, then **Read the cited lines to confirm** before quoting them.
4. **Fall back** to Glob/Grep/Read whenever codegraph is missing, the index looks stale, or the graph and the source disagree. The graph is an accelerator, never the source of truth.

Prefer the MCP server (`codegraph serve --mcp`) if you'd rather expose these as native tools instead of shelling out. Install, command reference, and MCP setup live in `references/codegraph.md`.

## Gotchas

Real failure points when explaining a codebase — these shift the work away from naive defaults:

- **A stale codegraph index lies.** If query results don't match what you see in the file, re-run `codegraph index` or drop to grep — never quote graph output you haven't confirmed against the current source.
- **Don't guess the architecture from names.** Verify with a codegraph query or Grep/Read before drawing a diagram. A folder called `services/` may hold dead code; a call graph beats a directory listing for "what calls what."
- **READMEs and CLAUDE.md go stale.** Treat them as claims to confirm against the actual code, not ground truth. Flag mismatches explicitly to the user.
- **Skip generated/vendored code.** `dist/`, `build/`, `node_modules/`, `*.generated.*`, protobuf/ORM output, lockfiles — these distort "most important files" lists. Explain the source that produces them, not the artifact.
- **Monorepos and multiple entry points.** Confirm which package/app the question is about before mapping. A single architecture diagram can be misleading when there are several deployables.
- **ASCII box alignment breaks in proportional fonts and with wide glyphs.** Count characters per row so borders line up in a monospace terminal; avoid tabs and double-width Unicode inside boxes.
- **Line numbers drift.** When citing `file.ts:34-52`, re-Read to confirm the range still matches before quoting it.
- **"Why" questions often need history, not just code.** Use `git log`/`git blame` on the relevant lines when the answer is about intent or a past decision, not current structure.

## ASCII diagrams

Use a diagram for anything with structure, flow, or state. The full gallery — boxes/flow, hierarchy trees, data flow, sequence diagrams, layer stacks, state machines, comparison tables — plus two fully worked example interactions are in **`references/diagram-toolkit.md`**. Read it when you need a pattern to copy.

## When asked for an overview

If asked to explain the codebase generally (no specific question):

1. Read `README.md` and `CLAUDE.md` (as claims to verify, not gospel).
2. Identify the tech stack.
3. Map the directory structure with purpose annotations — if codegraph is indexed, use it to find the real entry points and high-degree symbols rather than guessing from folder names.
4. Draw a high-level architecture diagram (see `references/diagram-toolkit.md`).
5. List the 5 most important files to understand, with one line each.
6. Suggest 3-4 concrete questions they might want to ask next.
