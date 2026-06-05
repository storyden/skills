---
name: storyden-cli
description: Storyden CLI skill for agents using the `sd` command to inspect, search, create, update, move, publish, triage, and manage Storyden content, nodes, threads, assets, metadata, properties, authentication contexts, and local Storyden development data. Use when the user asks to operate on a Storyden instance through the CLI, automate content workflows, survey site structure, mutate nodes or threads, run Storyden admin tasks, or dogfood the `sd` command. Prefer the installed `sd` CLI and its built-in help over hardcoded command knowledge.
---

# storyden-cli

Minimal operating guide for the Storyden `sd` CLI. The CLI is the source of
truth for command docs, examples, flags, and output formats.

## Start here

Before using an unfamiliar command, ask the installed CLI:

```bash
sd --help
sd node --help
sd thread --help
sd search --help
sd <command> <subcommand> --help
```

Use `--format json` or `--format jsonl` whenever the result will be parsed,
piped, compared, or fed into another command. Prefer plain output only for
human-facing summaries.

## Agent workflow

1. Confirm the target instance before making assumptions. Use `sd auth --help`
   to inspect auth commands; `sd auth switch` is interactive and useful when a
   human can choose the context.

```bash
sd auth --help
sd config path
```

2. Survey before mutating:

```bash
sd search "query" --format json
sd node list --format json
sd node tree
sd thread list --format json
```

3. Inspect exact objects before edits:

```bash
sd node get <slug-or-id> --format json
sd thread get <slug-or-id> --format json
```

4. For bulk or scripted work, keep data flowing through JSON/JSONL and let
   `sd --help` tell you which commands accept stdin, multiple arguments, batch
   files, or flags.

## Safety

- `sd` can mutate real Storyden data. Confirm the auth context and target
  instance before create/update/delete/move/publish actions.
- For local development, the user may allow broad mutation against
  `localhost:8000`; for production, treat writes as high impact.
- If an auth command needs browser/OAuth interaction, ask the user to complete
  it.
- Do not rely on this skill for exhaustive command syntax. Run `--help`; the
  installed CLI docs are intentionally richer and version-matched.

## Development Note

When modifying the Storyden CLI source in the Storyden repository, reinstall it
after changes so agents and humans test the same binary:

```bash
go install ./cmd/sd
```
