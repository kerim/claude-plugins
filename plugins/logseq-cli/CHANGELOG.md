# Changelog

All notable changes to the `logseq-cli` skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-05-18

### Changed

- **Skill rewritten for the Logseq.app-bundled `logseq` CLI.** The previous
  skill documented the obsolete `@logseq/cli` npm package (v0.4.2). The
  current `logseq` binary is shipped with the Logseq desktop app (Electron)
  and exposes a substantially larger command surface — `upsert`, `remove`,
  typed `list`/`search`, `graph` management, `server`, `sync`, `qsearch`/`qmd`,
  task-scoped commands, an `example` discovery command, and a built-in
  `skill show` that emits the canonical skill.
- **`query` syntax**: documented the new flag-based form. The command no
  longer accepts a positional query string and no longer supports `-p` /
  `--properties-readable`. Use `--query '<EDN>'`, optionally `--name` and
  `--inputs '<EDN-vector>'`, and `--output json|edn` for parseable output.
- Skill now follows the upstream CLI's policy of **not memorizing options
  or examples** — it points to `logseq <command> --help` and `logseq example`
  as live sources of truth.
- Added Claude Code integration guidance: Bash tool pattern with quoting
  tips, and a discovery workflow. Preserved 1.0.1's critical sandbox
  guidance (db-worker-node requires `allowLocalBinding: true`; do not
  work around it with `dangerouslyDisableSandbox`, `excludedCommands`,
  or `allowAllUnixSockets`) and elevated it under a "read this first"
  heading.
- Added task-scoped command guidance: prefer `upsert task --status ...` over
  putting `TODO`/`DOING`/`DONE` markers in `--content`.
- Added tag-association guidance: prefer `--update-tags '["a" "b"]'` (EDN
  vector) over hashtags-in-content or comma-separated strings.
- `examples/common-queries.md` rewritten as a deprecation note plus an
  `logseq example` discovery cheatsheet. The previous contents used removed
  flags and would have produced wrong results.
- `plugin.json` description updated to reflect the new CLI.

### Removed

- References to obsolete `@logseq/cli` features that no longer exist:
  `-p` / `--properties-readable`, positional query argument, `export -f`,
  `export-edn`/`import-edn`, `mcp-server`, `append`, and `validate` (the
  last is now `graph validate`).

## [1.0.1] - 2026-05-16

### Added

- Critical sandbox guidance: the CLI's `db-worker-node` daemon binds
  127.0.0.1 and requires `allowLocalBinding: true` in
  `~/.claude/settings.json`. Documents the failure signature and
  rejects three common (wrong) workarounds.

### Changed

- Marketplace homepage updated to `github.com/kerim/spellbook`.

## [1.0.0] - 2025-05-01

### Added

- Initial release. Documented `@logseq/cli` v0.4.2 commands (`list`, `query`,
  `search`, `show`, `export`), datalog query patterns, and Claude Code
  integration notes.
