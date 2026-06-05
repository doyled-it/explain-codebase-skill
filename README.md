# explain-codebase-skill

A Claude Code skill that explains how a codebase works through **ASCII diagrams and clear prose** — and uses [codegraph](https://github.com/colbymchenry/codegraph) for fast structural exploration when it's available.

Ask "how does authentication work?", "walk me through the request flow", or "give me an overview", and Claude answers with a monospace diagram (flow, tree, sequence, layers, state machine) plus cited `file:line` references and concrete follow-up questions.

## What it does

- **Diagram-first explanations.** Anything with structure, flow, or state gets an ASCII diagram, not just paragraphs.
- **codegraph-accelerated exploration (optional).** When `codegraph` is installed and the project is indexed, the skill queries the pre-built code graph (`query`, `callers`, `callees`, `impact`) instead of repeatedly grepping and reading — typically cutting exploration tokens and tool calls by roughly half. If codegraph isn't present, it falls back to the usual Glob/Grep/Read.
- **Honest about staleness.** Graph output and doc claims are treated as leads to verify against the current source, never as ground truth.

## Install

### As a plugin (recommended)

```
/plugin marketplace add doyled-it/explain-codebase-skill
/plugin install explain-codebase@doyled-it-skills
```

### As a bare skill

Clone and symlink the skill directory into your skills folder:

```bash
git clone https://github.com/doyled-it/explain-codebase-skill.git
ln -s "$PWD/explain-codebase-skill/skills/explain-codebase" ~/.claude/skills/explain-codebase
```

## Optional: codegraph

The skill works without it, but installs cheaply and pays for itself on any repo you explore more than once:

```bash
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
cd your-project && codegraph init && codegraph index
```

See [`skills/explain-codebase/references/codegraph.md`](skills/explain-codebase/references/codegraph.md) for the full command reference and MCP-server setup.

## Layout

```
.claude-plugin/
  plugin.json          # plugin manifest
  marketplace.json     # single-plugin marketplace, source "./"
skills/explain-codebase/
  SKILL.md             # behavior, exploration strategy, gotchas
  references/
    diagram-toolkit.md # ASCII pattern gallery + worked examples
    codegraph.md       # install, commands, MCP setup
```

## License

MIT — see [LICENSE](LICENSE).
