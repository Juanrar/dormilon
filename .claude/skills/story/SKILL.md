---
name: story
description: Refines ONE user story right before building it, with 2 or 3 Given/When/Then acceptance criteria, the smallest plan that meets them, and what stays out. Writes no code; that is /pair's job. Run as /story [lite|full|ultra] <story or #number>.
argument-hint: "[lite|full|ultra] <story or #number>"
disable-model-invocation: true
---

# Story

We refine a single story, right before building it. You propose and I decide. You plan like a lazy senior developer: the smallest complete change that meets the criteria, and nothing "for later".

Write in the language I write in, including the template labels.

## Level

Read the level from the argument. Default: `full`.

| Level | Behavior |
|-------|----------|
| `lite` | Plan what the story asks for. If a smaller plan would do, name it in one line and let me pick. |
| `full` | The steps and rules below. |
| `ultra` | Also question the story. Before the criteria, push back on any part the need does not justify and propose the smaller story. |

## Steps

1. **Read the story.** It can be pasted text, a number from the backlog in this conversation or in `BACKLOG.md`, or an issue. Read `CLAUDE.md` (or `AGENTS.md`) for the stack and the constraints. Read the code the story touches.
2. **Find the scope.** List every place the change must reach: screens, commands, data, callers, tests, config. Check what it could break for me or my users: data it could destroy or expose, behavior that stops working. That is scope. Extra features are not.
3. **Acceptance criteria.** Propose 2 or 3 in Given/When/Then format. They describe what I see or get, never how it is built. Use concrete example values ("3 sessions", "the path `C:\My Projects\api`"), so `/pair` can turn each one into a test without computing the expected result. If a project constraint first matters in this story, make it one of the criteria.
4. **More than 3 criteria means the story is too big.** Don't bloat it: propose a split. Give the new stories in backlog format (`As a..., I want..., so that...`) for me to add to the tracker, and continue with the first one.
5. **Plan.** 5 bullets at most, each one line of about 25 words or fewer, no code. Say what each part does and what it reuses, not how the functions look inside: that detail belongs to `/pair`. A technical fact that changes the plan (a library limit, a quirk of the data format) goes in the `Risk:` line. For each part, take the first option that works:
   1. Does it need to exist for these criteria? If not, it goes to "Out of scope".
   2. Is it already in the codebase? Reuse it, and name what you reuse.
   3. Does the standard library or the platform do it? Use that.
   4. Does an installed dependency do it? Use that. A new dependency is a decision for me: name it and say why the standard library is not enough.
   5. Otherwise, the minimum new code, cutting through the layers the criteria need.

   Infrastructure the story needs (project setup, a test runner, a file reader) is built here, only as much as this story uses. A deliberate shortcut is fine if the plan names its limit and when to upgrade, for example "reads every file on each refresh; upgrade when it gets slow with 500+ sessions".
6. **Wait for my approval.** We adjust until I say yes.

## Rules

- Don't invent requirements. If information is missing, ask me.
- The simplest thing that meets the criteria. No abstractions, layers, options, config or patterns "for the future".
- Never cut: validation where outside input comes in, error handling that prevents data loss, security, accessibility, the project constraints, anything the story asks for.
- Suggest, don't decide. If a pattern could help (extracting a component, a middleware, a shared helper), put it under "Suggestions" with its reason. Only suggest it when the code shows a concrete signal, such as something that already repeats. Not in advance.
- Don't write code.

## Output format

A block ready to paste into the issue. `/pair` takes it as input.

```markdown
## Story
As a <actor>, I want <action>, so that <benefit>.

## Acceptance criteria
- [ ] Given <context>, when <action>, then <result>.
- [ ] ...

## Plan
- ...

## Out of scope
- <what stays out of this story, so it doesn't sneak in>

## Suggestions (your call)
- <optional; leave the section out if empty>
```

After the block, end with these lines. Keep them short: they are a summary I read in one pass, not a report.

- `Skipped:` one line on what you did not check or left out of the plan on purpose.
- `Risk:` up to 3 short bullets: what this story could break or expose, and technical facts that change the plan. Leave it out if there is none.
- A question that asks me to approve the story or change it. Don't start building until I say yes.
