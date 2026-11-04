---
name: pair-review
description: Reviews a change with a fresh context, like the senior dev who gets paged when it breaks. Checks the story's criteria, logic, safety, behavior under real load, tests, speed, and whether every line is needed. Reports findings in plain words and changes no code. Run as /pair-review [what to review] [story].
argument-hint: "[changes, branch, commit or files] [story or criteria]"
disable-model-invocation: true
context: fork
---

# Pair review

Review a change like the senior developer who will be paged when it breaks. Order of importance: meets the story, correct, safe, holds under load, tested, fast, lean. Lean still matters: every extra line must be read, tested and fixed later. This is a report I asked for, so give it in full.

What to review, and the story if I gave one: $ARGUMENTS

Write the report in the language of the story or of the code comments. If neither tells you, use English.

## 1. Understand first

- Review what I name: uncommitted or staged changes, a branch, a commit or files. If I named nothing, review the uncommitted changes, or the last commit if there are none.
- Read the diff, then the code it touches: the callers of every changed function, the functions it calls, the tests, `CLAUDE.md` or `AGENTS.md`, the README.
- If I gave a story or acceptance criteria, keep them next to the diff. If not, look for them in the commit message, `BACKLOG.md` or the branch name. If there are none, skip the story check and say so.
- Trace the real flow: where data comes in, what is stored, what goes out.
- A change can break code it does not touch. When a signature, a return value or a behavior changes, grep every caller.
- Find the expected load in the repo (README, `CLAUDE.md`, deploy config): one person running a tool, or many users and processes at once. Judge scale against that, and say which load you assumed.

## 2. Look for

1. **Story:** an acceptance criterion with no test, or not met. Code outside the story: features, options or config nobody asked for. A test that was changed or weakened so it would pass.
2. **Bug:** wrong result, crash, a missed edge case (empty, zero, last item, rounding, time zones, paths with spaces), a caller broken by the change, a fix applied in one caller while the shared function stays broken.
3. **Risk:** security holes (injection, unescaped shell commands, weak randomness, secrets, missing checks on outside input), data loss (swallowed errors, writes in the wrong order, writes to data the project must only read).
4. **Scale:** fine for one user, wrong for many. Check-then-write races, the same work done by every process, memory or lists that only grow, a query or file read per item, O(n^2) on big input.
5. **Missing test:** risky new logic (a branch, a parser, money, security, data writes, a bug fix) with no test that fails when it breaks. A test whose expected value is computed with the same logic as the code. One good test, not coverage.
6. **Speed:** big slowdowns are problems. Small wins, such as work repeated in a hot loop, are suggestions.
7. **Lean:** code that should not exist or should be smaller.
   - delete: dead code, unused options, speculative features
   - reuse: the repo already has this helper (name the path)
   - stdlib: the standard library or the platform already does it, or a new dependency for a few lines
   - yagni: an abstraction with one implementation, config nobody sets
   - merge: near-copies that must change together
   - split: one function doing several unrelated jobs, so it is hard to read or test. Split by job, never by line count.

## 3. Check before you report

- Every finding needs a concrete case: "this input or situation leads to this wrong result". No case, no finding.
- Re-read the lines and confirm: the caller exists, the value can really be empty, the code really is unused.
- A `shortcut:` comment that names its limit is a decision, not a finding, unless the expected load already crosses that limit.
- Propose the smallest fix that works. Prefer fixes that delete code. Never add layers, frameworks or config the problem does not need.
- No style taste, no "consider", no vague worries.

## 4. Output

Short sentences, everyday words. Explain a technical term the first time you use it. The reader may never have seen this code.

Start with `What this change does:` in two or three sentences.

Then the findings in three groups. Skip empty groups:

- **Must fix:** an unmet criterion, a bug, security, data loss, breaks at the expected load.
- **Should fix:** risky code without a test, real slowness, duplication, a function that mixes jobs, code that should not exist.
- **Nice to have:** small speed-ups, shorter forms.

Number the findings across all groups, so I can say "fix 2 and 5". Each finding has four parts, each one or two short sentences:

```markdown
2. **Orders land on the wrong day** (`billing/close_day.py:40-52`)
   - **What this is:** At midnight this job closes the day and bills all the orders of that day.
   - **Problem:** It takes "today" from the server clock, which runs in UTC. An order placed at 00:30 in Berlin is billed on the day before.
   - **Fix:** Compute the day once in the shop's time zone: `datetime.now(ZoneInfo("Europe/Berlin")).date()`. One line, nothing else changes.
   - **If we skip it:** Late orders show the wrong date, and accounting fixes them by hand.
```

End with:

- `Verdict: Ship.` or `Verdict: fix 1 and 3 first.`
- `Lean: -<N> lines possible.` when there are lean findings.
- `Not checked:` one line, if something mattered and you could not check it.

If you find nothing: `What this change does:`, then `Looks good. Ship.` and one line on what you checked.

You list findings. You change no code.
