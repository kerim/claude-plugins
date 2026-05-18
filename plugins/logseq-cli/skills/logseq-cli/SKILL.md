---
name: logseq-cli
description: Interface with Logseq DB graphs using the `logseq` command-line tool bundled with Logseq.app. Use this skill whenever a request involves running `logseq` commands, inspecting or editing graphs/pages/blocks/tasks/tags/properties, running Datascript queries, managing graphs and db-worker-node servers, or interpreting CLI output.
---

# Logseq CLI

Operate the `logseq` CLI to inspect and modify graphs, pages, blocks, tasks, tags, and properties, run Datascript queries, view page/block trees, manage graphs and backups, and control db-worker-node servers.

## What this CLI is

The current `logseq` binary is bundled with the **Logseq desktop app** (Electron). It is not the old `@logseq/cli` npm package — that package is obsolete and its flags (especially `-p` / `--properties-readable` and positional query arguments) no longer apply.

On macOS, the launcher is typically a tiny shim at `~/.local/bin/logseq` that execs the Electron binary with `ELECTRON_RUN_AS_NODE=1`:

```sh
ELECTRON_RUN_AS_NODE=1 exec "/Applications/Logseq.app/Contents/MacOS/Logseq" \
  "/Applications/Logseq.app/Contents/Resources/app.asar/js/logseq-cli.js" "$@"
```

A short Electron code-signing line may appear on stderr when running — it is harmless.

## Canonical skill

The CLI ships its own authoritative skill. Whenever you suspect this file is stale, re-pull it:

```bash
logseq skill show         # print built-in skill markdown
logseq skill install      # write it to disk
```

The built-in skill's policy — followed here too — is to **not memorize options or examples**, because they change. Always check live:

```bash
logseq --help                            # top-level commands
logseq <command> --help                  # command-specific options
logseq <command> <subcommand> --help     # for grouped commands
logseq example                           # discover runnable examples
logseq example <command-or-prefix...>    # scoped examples (e.g. `logseq example upsert page`)
```

## Command groups (high-level map)

Use `--help` to get the live, exact list. As of this writing, the groups are:

- **Graph inspect & edit**
  - `list page|tag|property|task|node|asset` — typed listing
  - `upsert block|page|task|asset|tag|property` — create or update
  - `remove block|page|tag|property`
  - `query`, `query list` — Datascript queries (see below)
  - `qsearch` — search graph Markdown Mirror with QMD
  - `search block|page|property|tag` — typed search by title
  - `show` — show page/block tree
- **Graph management**
  - `graph list|create|switch|remove|validate|info|export|import`
  - `graph backup list|create|restore|remove`
  - `server list|cleanup|start|stop|restart` — db-worker-node servers
  - `doctor` — runtime diagnostics
  - `sync status|start|stop|upload|download|asset download|remote-graphs|ensure-keys|grant-access|config set|get|unset`
- **Authentication**: `login`, `logout`
- **Utilities**: `completion`, `debug`, `example`, `qmd`, `skill`

## Global options (apply to most commands)

- `-g, --graph <name>` — target graph (most commands need this)
- `-o, --output <human|json|edn>` — output format. Default `human`. Use `json` or `edn` only when machine-readable output is required.
- `--config <path>` — path to `cli.edn` (default `<root-dir>/cli.edn`)
- `--root-dir <path>` — CLI root dir (default `~/logseq`)
- `--timeout-ms <ms>` — request timeout (default `10000`)
- `--profile` — stage timing to stderr
- `-v, --verbose` — debug logging to stderr
- `--version`, `-h, --help`

## Datascript queries

The `query` command no longer takes a positional query string or the old `-p` flag. It uses named flags:

```bash
logseq query --graph "<name>" --query '<EDN query>'
logseq query --graph "<name>" --name <named-query> --inputs '<EDN vector>'
logseq query list --graph "<name>" [--output edn]
```

- `--query` accepts an EDN Datascript query.
- `--name` runs a built-in query or a `custom-queries` entry from `cli.edn`. `query list` enumerates both.
- `--inputs` is an EDN vector of arguments passed to the query.
- Use `--output json` or `--output edn` for parseable output; the default `human` form is for reading.

Discover working examples with `logseq example query`.

## Tasks: prefer task-scoped commands

If a request is about tasks, prefer the task-typed commands before falling back to block/page workflows:

- `list task` to read; `upsert task` to create or update.
- Task state belongs in `--status` (e.g. `--status todo|doing|done`), **not** in `--content`. Do not write `TODO`/`DOING`/`DONE` into the content string.
- If a task block also needs tags, do the tag association on the **same block** by id afterwards, e.g.:
  ```bash
  logseq upsert task --graph "<g>" --target-page "<page>" --content "Some content" --status done --output json
  # then use the returned block id:
  logseq upsert block --graph "<g>" --id <id> --update-tags '["AI-GENERATED" "CLI"]'
  ```

## Structured writes

When writing multi-item or hierarchical content, build a **block tree** rather than packing everything into one `--content` string. Each logical bullet, row, or subsection should usually become its own block (siblings/children). Reserve `--content` for true single-block writes or targeted single-block updates.

## Tag association

Use explicit tag options, not hashtags-in-content or comma-separated strings:

- Prefer `--update-tags` and `--remove-tags` on `upsert block` / `upsert page`.
- `--update-tags` expects an **EDN vector**, e.g. `'["AI-GENERATED" "CLI"]'`. A comma-separated string is not parsed as tags.
- Tag values may be tag title/name strings, db/id, UUID, or `:db/ident`. String values may include a leading `#`, but they still go inside the vector.
- Tags must already exist and be public. Create first when needed: `logseq upsert tag --name "<TagName>"`. After a tag-association failure, verify the tag exists/is public before retrying.

## Anti-patterns

- ❌ `upsert block --content "DONE Implemented and verified ..."` — task state in content.
  ✅ `upsert task --status done --content "Implemented and verified ..."`
- ❌ `--content "Summary #AI-GENERATED"` — treating hashtags as tag association.
  ✅ `--update-tags '["AI-GENERATED"]'`
- ❌ `--update-tags "AI-GENERATED,CLI"` — comma-separated string.
  ✅ `--update-tags '["AI-GENERATED" "CLI"]'`
- ❌ Hardcoding old `@logseq/cli` flags (`-p`, `--properties-readable`, positional query string, `export -f`, `mcp-server`). They no longer exist.

## Tips

- `query list` returns both built-in queries and `custom-queries` from `cli.edn`.
- `show --id` accepts either one db/id or an EDN vector of ids.
- `remove block --id` also accepts one db/id or an EDN vector.
- `upsert block` enters **update mode** when `--id` or `--uuid` is provided; otherwise it creates.
- The CLI works whether or not the Logseq desktop app is running; some commands (server, sync) interact with a long-running db-worker-node process.
- If `logseq` reports it doesn't have read/write permission for `root-dir`, check filesystem permissions or set `LOGSEQ_CLI_ROOT_DIR`.
- In sandboxed environments, `graph create` may print a process-scan warning to stderr; if command status is `ok`, the graph is still created.

## Claude Code integration

### Sandbox (read this first)

The CLI spawns a `db-worker-node` daemon that binds an HTTP server on 127.0.0.1 for IPC. Claude Code's macOS sandbox blocks loopback `bind()` by default, so the daemon fails before the CLI returns. Required settings in `~/.claude/settings.json`:

```json
{
  "sandbox": {
    "network": {
      "allowLocalBinding": true
    },
    "filesystem": {
      "allowWrite": ["/Users/YOU/logseq"]
    }
  }
}
```

**If `logseq` commands fail with `Error (server-start-failed): db-worker-node failed to publish health`, connection-refused on 127.0.0.1, or `EPERM`/`EACCES` on `bind()`, the user is missing `allowLocalBinding: true`.** Stop and surface this to the user. Do NOT work around it with `dangerouslyDisableSandbox` (brittle, prompts every call), `excludedCommands` (doesn't propagate to the daemon child), or `allowAllUnixSockets` (no-op on macOS — wrong knob). See the marketplace README's "Sandbox configuration" section for the full explanation.

### Bash tool pattern

```typescript
Bash({
  command: 'logseq query --graph "<graph-name>" --query \'[:find (pull ?b [*]) :where [?b :block/title]]\' --output json',
  description: "Run Datascript query, JSON output"
})
```

Tips:

- Use **single quotes** around the EDN query and double quotes for string literals inside it.
- For programmatic parsing, always pass `--output json` (or `edn`) — the default `human` format is not stable.
- The user's default shell is fish; quoting works the same as bash/zsh for these commands.

### Discovery workflow

1. `logseq graph list` — find graph names (case-sensitive; use the exact string with `--graph`).
2. `logseq example <command-prefix>` — get a runnable template.
3. `logseq <command> --help` — confirm flags.
4. Replace placeholder ids/uuids with real entities from `logseq list ...`, `logseq show ...`, or `logseq query ...`.
5. For graph transfer, keep `graph export --file` and `graph import --input` paths consistent.

## Related skills

- `logseq-db-knowledge` — Logseq DB graph data model (nodes, properties, tags, tasks).
- `logseq-db-plugin-api-skill` — Building Logseq plugins; useful when prototyping queries before embedding them in plugin code.
- `logseq-schema` — Authoritative Datascript schema and `:db/ident` reference for writing Datalog queries.
- `qmd` — Local markdown search index; complements `logseq qsearch`.
