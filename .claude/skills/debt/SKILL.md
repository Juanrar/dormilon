---
name: debt
description: Lists every `shortcut:` comment in the repo as a debt ledger, so a deliberate shortcut can't quietly become permanent. One-shot report that changes nothing. Run as /debt [marker word].
argument-hint: "[marker word, default shortcut]"
disable-model-invocation: true
---

# Debt

`/pair` marks every deliberate shortcut with a comment that names its limit and when to upgrade: `shortcut: <the limit>, <when to upgrade>`. This skill collects them into one ledger.

Talk to me in the language I write in.

## Scan

Grep the repo for the comment marker. Skip `.git` and everything `.gitignore` ignores (dependencies, build output, caches):

```bash
git grep --no-index --exclude-standard -nE '(#|//|/[*]|--|;) ?shortcut:'
```

Add other comment prefixes if the stack uses them. If I name my own marker word (`/debt TODO`), grep for that word instead.

Each hit is one ledger row. The comment prefix keeps prose that only mentions the convention, like this file, out of the ledger. Skip hits that are not a deferral, such as a note about a keyboard shortcut.

## Output

One row per marker, grouped by file:

`<file>:<line>, <what was simplified>. limit: <the limit named>. upgrade: <when to revisit>.`

Take the limit and the trigger straight from the comment. If I want an owner per row, add `git blame -L<line>,<line> <file>`.

Tag every marker that names no upgrade trigger with `no-trigger`. Those are the ones that rot.

End with `<N> markers, <M> with no trigger.` If there are none: `No shortcut debt. Clean ledger.`

## Boundaries

Read and report only. Change nothing. If I ask to keep the ledger, write it to `DEBT.md`.
