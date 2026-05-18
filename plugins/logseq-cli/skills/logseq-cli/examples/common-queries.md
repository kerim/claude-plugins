# Common Logseq CLI Query Patterns

> **Deprecated.** The previous contents of this file documented the old
> `@logseq/cli` npm package (v0.4.2) and used flags that no longer exist in the
> current Logseq.app-bundled `logseq` CLI — notably `-p` /
> `--properties-readable` and the positional query string syntax. Following
> those examples will produce wrong results or errors.
>
> The current CLI is the source of truth for examples. Use:
>
> ```bash
> logseq example                            # all examples
> logseq example query                      # query examples
> logseq example upsert                     # all upsert examples
> logseq example upsert task                # one specific command
> logseq <command> --help                   # exact flags for any command
> ```
>
> See `../SKILL.md` for the current command map and Datascript query syntax
> (`logseq query --graph "<g>" --query '<EDN>' --output json|edn`).

## Examples discovery cheatsheet

```bash
# What can I list?
logseq example list

# What can I create/modify?
logseq example upsert
logseq example remove

# Graph management
logseq example graph
logseq example graph backup

# Sync
logseq example sync

# Search / inspect
logseq example search
logseq example qsearch
logseq example show
```

Pick an example, replace placeholder ids/uuids with real entities discovered
via `logseq list ...`, `logseq show ...`, or a Datascript `logseq query ...`,
and run it.
