---
description: Set up minimal Claude Code memory + guardrails in a fresh repo (instead of /init)
---

# Task: set up minimal Claude Code memory and guardrails for this repo

Execute end-to-end **without asking questions beyond Step 0** — every other decision is pre-made
below. If the repo deviates from the state a step assumes, adapt and name every skip in the final
report rather than aborting or asking.

**Hard constraint: no git mutations of any kind** (no commit, push, merge, branch, add, stash — at
any point, for any reason). File edits only. Suggest a branch name and commit message at the end;
the user commits.

## Step 0 — project type (mandatory first action)

Detect the stack cheaply first: `angular.json` present → Angular; else a `tsconfig.json` or an
ESLint config → TypeScript/JavaScript; else Other. Then call the **AskUserQuestion** tool (this
renders the arrow-key picker) with exactly these options, putting the detected type **first** with
"(Recommended)" appended to its label. Wait for the answer; it selects the pipeline.

> **Question:** "What kind of project should I set up Claude Code guardrails for?"
> (header: "Project type")
>
> 1. **Angular workspace** — Scaffolded `.claude/CLAUDE.md` and angular-eslint expected. Full
     >    pipeline: classify the Angular conventions block, map to lint rules, audit & fix violations,
     >    guardrails, policy doc.
> 2. **TypeScript / JavaScript** — No Angular. Classify whatever memory files exist, work with the
     >    ESLint/Prettier setup that's already installed (no new tooling), guardrails, policy doc.
> 3. **Other stack** — Any other language/toolchain. Skip lint mapping unless the repo already has
     >    a linter; guardrails, CI gate if CI exists, policy doc.

Pipeline per answer:

- **Angular workspace** → all steps below, including the Angular mapping table in Step 1.
- **TypeScript / JavaScript** → Steps 1–3 run against whatever memory files and lint tooling
  already exist (the Angular table does not apply; classify with the same buckets). Install no
  new tooling.
- **Other stack** → same as TypeScript/JavaScript, and skip Steps 2–3 entirely if the repo has no
  linter; note that in the policy doc and the report.

## Principles — pre-decided, do not relitigate

- `CLAUDE.md` is injected into every session; every line spends instruction budget whether
  relevant or not. Repository overviews do not help coding agents; practice specifications may
  (arxiv.org/abs/2602.11988). Never run `/init`; never paste command listings, file inventories,
  architecture narration, or history into `CLAUDE.md`.
- Every instruction goes to the **strongest layer that can hold it**:
  1. mechanically enforceable → linter / compiler config / CI,
  2. harness-enforceable → `.claude/settings.json` permission rules,
  3. only what is neither derivable from the repo nor enforceable → prose in the root
     `CLAUDE.md`.
- **A root `CLAUDE.md` is optional, not an artifact this command must produce.** It is created
  only if surviving prose passes the bar in Step 5. A fresh repo usually needs none. Hard cap if
  one is created: 60 lines; a fresh repo should land well under 30, growing only with evidence
  (a regression traced to a missing line).
- Prose guardrails are probabilistic — they degrade exactly in long, busy, compacted sessions.
  Permission rules are deterministic and bind in **every** permission mode: evaluation order is
  deny → ask → allow → mode default, so an `ask` rule prompts even in auto mode and beats any
  `allow` (code.claude.com/docs/en/permissions.md).

## Step 1 — classify every instruction in the existing memory files

Inventory `.claude/CLAUDE.md`, a root `CLAUDE.md`, and any other agent-memory files present.
Account for every line; nothing is silently dropped — dropped items are recorded in the policy
doc (Step 6). Buckets: **A** derivable from the repo (drop), **B** mechanically enforceable (move
to lint/compiler config), **C** not derivable (candidate prose — Step 5 decides), **drop**
(generic craft advice or obsolete).

**Angular only** — standard verdicts for the scaffolded block, but **verify each rule name against
the installed angular-eslint first** (`ls node_modules/@angular-eslint/eslint-plugin/dist/rules/`
and the template plugin's rule keys); this table was checked against v21.1.0 and rule sets drift:

| Instruction | Disposition |
| --- | --- |
| strict typing, no `any` | tsconfig `strict` + `no-explicit-any` (usually already active) — verify, drop prose |
| standalone components | `@angular-eslint/prefer-standalone` |
| `inject()` over constructor DI | `@angular-eslint/prefer-inject` |
| signals / `readonly` signal props | `@angular-eslint/prefer-signals` (idioms only — "signals over Subjects/RxJS state" is candidate prose) |
| OnPush change detection | `@angular-eslint/prefer-on-push-component-change-detection` |
| `host` object over `@HostBinding`/`@HostListener` | `@angular-eslint/prefer-host-metadata-property` |
| `input()`/`output()` over decorators | `prefer-signals` + `@angular-eslint/prefer-output-emitter-ref` |
| `providedIn: 'root'` singletons | `@angular-eslint/use-injectable-provided-in` |
| relative template/style URLs | `@angular-eslint/relative-url-prefix` |
| native control flow `@if`/`@for` | `@angular-eslint/template/prefer-control-flow` |
| `NgOptimizedImage` for static images | `@angular-eslint/template/prefer-ngsrc` (template `<img>` only) |
| accessibility (focusable + key events on interactive elements) | `angular.configs.templateAccessibility` preset (static checks only) |
| no `ngClass`/`ngStyle`; reactive forms; WCAG AA runtime half; signals-first state | **no rule exists** (as of 21.x) → candidate prose |
| small/focused components, inline templates for small ones, async pipe, "don't assume globals", lazy-load feature routes | drop — generic craft or judgment calls |
| no `mutate` on signals | drop — the API was removed in Angular 17; the rule is obsolete |

## Step 2 — audit read-only, then enable

Before touching the lint config, write a temporary config **outside the repo** (e.g. the
scratchpad) that spreads the repo's config and adds all Step-1 bucket-B rules as `error`; run the
linter with `--format json` against it and count violations per rule. Then fix **all** violations
(a fresh scaffold typically has only a handful, e.g. the generated root component lacking OnPush)
and add the rules to the repo's lint config as `error`. Angular field notes:

- `prefer-signals` findings are usually just missing `readonly` — mechanical.
- Spec files are linted too, including inline test `@Component`s.
- `ngSrc` conversions need `NgOptimizedImage` in the component's `imports`, numeric `width`/
  `height` (no units) or `fill`, and a check of the CSS so the new attributes don't change layout;
  mark the LCP image `priority`.
- Run Prettier (or the repo's formatter) over every file you touch.

## Step 3 — CI gate

If a workflow already runs tests/builds, add the lint command (per package/project) so a violation
blocks the pipeline. If the repo has no CI yet, record in the policy doc that the lint rules are
unenforced until a CI lint step exists — without CI, Step 2 is theater.

## Step 4 — write `.claude/settings.json` (checked in, verbatim, all stacks)

```json
{
  "permissions": {
    "ask": [
      "Bash(git commit:*)", "Bash(git push:*)", "Bash(git merge:*)", "Bash(git rebase:*)",
      "Bash(git reset:*)", "Bash(git checkout:*)", "Bash(git switch:*)", "Bash(git restore:*)",
      "Bash(git add:*)", "Bash(git rm:*)", "Bash(git mv:*)", "Bash(git clean:*)", "Bash(git stash:*)",
      "Bash(git cherry-pick:*)", "Bash(git revert:*)", "Bash(git am:*)", "Bash(git apply:*)",
      "Bash(git tag:*)", "Bash(git branch:*)", "Bash(git notes:*)", "Bash(git replace:*)",
      "Bash(git filter-branch:*)", "Bash(git filter-repo:*)", "Bash(git bisect:*)",
      "Bash(git gc:*)", "Bash(git prune:*)", "Bash(git repack:*)", "Bash(git maintenance:*)",
      "Bash(git reflog expire:*)", "Bash(git reflog delete:*)", "Bash(git update-ref:*)",
      "Bash(git symbolic-ref:*)", "Bash(git pack-refs:*)", "Bash(git commit-tree:*)",
      "Bash(git worktree:*)", "Bash(git submodule:*)", "Bash(git remote:*)", "Bash(git config:*)",
      "Bash(git pull:*)",
      "Bash(git -C:*)", "Bash(git -c:*)", "Bash(git --git-dir:*)", "Bash(git --work-tree:*)",
      "Bash(gh pr create:*)", "Bash(gh pr merge:*)", "Bash(gh pr close:*)", "Bash(gh pr reopen:*)",
      "Bash(gh pr edit:*)", "Bash(gh pr ready:*)", "Bash(gh pr review:*)", "Bash(gh pr comment:*)",
      "Bash(gh pr lock:*)", "Bash(gh pr unlock:*)", "Bash(gh pr update-branch:*)",
      "Bash(gh issue create:*)", "Bash(gh issue edit:*)", "Bash(gh issue close:*)",
      "Bash(gh issue reopen:*)", "Bash(gh issue delete:*)", "Bash(gh issue comment:*)",
      "Bash(gh issue transfer:*)", "Bash(gh issue pin:*)", "Bash(gh issue unpin:*)",
      "Bash(gh issue lock:*)", "Bash(gh issue unlock:*)", "Bash(gh issue develop:*)",
      "Bash(gh label create:*)", "Bash(gh label edit:*)", "Bash(gh label delete:*)", "Bash(gh label clone:*)",
      "Bash(gh release create:*)", "Bash(gh release edit:*)", "Bash(gh release delete:*)", "Bash(gh release upload:*)",
      "Bash(gh repo create:*)", "Bash(gh repo edit:*)", "Bash(gh repo delete:*)", "Bash(gh repo rename:*)",
      "Bash(gh repo archive:*)", "Bash(gh repo unarchive:*)", "Bash(gh repo sync:*)", "Bash(gh repo fork:*)",
      "Bash(gh workflow run:*)", "Bash(gh workflow enable:*)", "Bash(gh workflow disable:*)",
      "Bash(gh run cancel:*)", "Bash(gh run rerun:*)", "Bash(gh run delete:*)",
      "Bash(gh secret:*)", "Bash(gh variable:*)", "Bash(gh gist:*)", "Bash(gh api:*)"
    ],
    "deny": [
      "Edit(.git/**)"
    ]
  }
}
```

Hard-won details: never write `Write(path)` rules — path permission checks only evaluate
`Edit(path)`, which covers **all** file-editing tools; a `Write(...)` entry is dead and triggers a
startup warning. Compound commands are split on `&&`/`;`/`|`/newlines and each subcommand must
match independently. The `git -C/-c/--git-dir/--work-tree` entries close the
flags-before-subcommand hole. Expect the auto-mode classifier to block *you* from editing this
file once it exists — that is by design; the user maintains it. The ask rules bind from the next
session onward. If `.claude/settings.local.json` gains allows later, keep `gh` entries read-only
(`view`/`list`/`diff`/`status`).

## Step 5 — decide whether a root `CLAUDE.md` is warranted; replace the memory files

Delete `.claude/CLAUDE.md` if present (its content is now in lint rules or the candidate-prose
list). Then apply this bar to **every** candidate-prose line from Step 1:

> Would its absence plausibly cause a wrong implementation that lint, types, and tests would not
> catch?

- **No line passes → create no `CLAUDE.md` at all.** This is the expected outcome for a fresh
  repo. The generic conventions (e.g. Angular's signals-first / reactive-forms / `ngClass` ban /
  WCAG-AA lines) usually fail the bar in a fresh repo. The git-mutation prose rule **alone does
  not justify creating the file** — `.claude/settings.json` is the enforcement; record the
  residual prose gap (indirect git via scripts) as an accepted gap in the policy doc instead.
- **Lines pass → create the file with only those lines**, in this skeleton (target under 30
  lines; hard cap 60):

```markdown
# CLAUDE.md

<2–4 lines: only what a reader cannot get from ls + package.json — project intent, unusual
structure. No command listings, no framework narration.>

## Process

- Git/GitHub mutations (commit, push, merge — anything that writes to the repo, its history, or a
  remote) only on explicit instruction in the current turn; the user stating an intention ("I'd
  like to…") is not an instruction. `.claude/settings.json` enforces the direct paths; this rule
  also covers indirect ones (scripts, `sh -c`, npm scripts that shell out to git).
- Before editing this file or creating any new agent-memory file, read
  `docs/CLAUDE-CODE-POLICY.md`.

## Conventions not enforceable by lint

<only the surviving lines>
```

Sections for **Invariants** and **Traps** are added later, only when the project has earned them:
an invariant is something whose violation compiles and passes tests; a trap is an intentional
absence or a prod-only failure mode. A fresh repo has neither — do not pad. Likewise, when the
project later grows a version-pinned external API surface (Shopify, Stripe, Firebase, …), add one
Process bullet mandating research of the current official docs before writing code against that
surface, naming the single file that records the pinned version. Earning an Invariants/Traps/
Process line later is also what justifies creating the file later if none exists yet.

## Step 6 — write `docs/CLAUDE-CODE-POLICY.md`

The decision record future sessions must read **before creating or editing any `CLAUDE.md`** —
this matters doubly when Step 5 created none: the policy doc is then the only thing standing
between the next session and a reflexive `/init`. Contents: the CLAUDE.md-is-optional rule with
the Step-5 bar and the size budget + raise-triggers (evidence only); a what-lives-where table
(prose / lint+CI / settings.json / docs / auto-memory); the prose→lint map **with the audited
per-rule violation counts from Step 2** (if lint ran); **every considered-and-dropped prose line**
so nothing vanished silently; the guardrail design plus its residual gaps (interpreter indirection
via allowed `node`/`npm`, leading env assignments like `GIT_DIR=x git push`, un-enumerated
plumbing, the human "don't ask again" click — plus, when no `CLAUDE.md` exists, the accepted
absence of the prose backstop for indirect git); and the traps: removing the CI lint step silently
reverts conventions to prose, `Write(path)` rules are invalid, and any session predating
`.claude/settings.json` is guarded only by prose.

## Step 7 — verify and report

Run the repo's lint, tests, and build — whichever of these exist — and all must pass. Then
report: the chosen project type; per-rule violation counts found and fixed (if lint ran); whether
a root `CLAUDE.md` was created (and its line count) or why none was needed; anything this repo
lacked (no CI, no linter, missing rules in the installed lint plugin); and a suggested branch
name + one-line conventional commit message. Do not commit.
