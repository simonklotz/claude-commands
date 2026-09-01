---
description: Set up minimal Claude Code memory + guardrails in a fresh Angular repo (instead of /init)
---

# Task: set up minimal Claude Code memory and guardrails for this repo

This is a fresh Angular workspace. `.claude/CLAUDE.md` contains the scaffolded Angular
best-practices block; ESLint (flat config, angular-eslint) and Prettier are configured. If the
repo deviates from that state (no `.claude/CLAUDE.md`, an already-customized `CLAUDE.md`, no
angular-eslint), adapt: classify whatever memory files exist, and skip lint steps that have no
tooling to land in — noting every skip in the final report. Execute end-to-end **without asking
questions** — every decision is pre-made below.

**Hard constraint: no git mutations of any kind** (no commit, push, merge, branch, add, stash — at
any point, for any reason). File edits only. Suggest a branch name and commit message at the end;
the user commits.

## Principles — pre-decided, do not relitigate

- `CLAUDE.md` is injected into every session; every line spends instruction budget whether
  relevant or not. Repository overviews do not help coding agents; practice specifications may
  (arxiv.org/abs/2602.11988). Never run `/init`; never paste command listings, file inventories,
  architecture narration, or history into `CLAUDE.md`.
- Every instruction goes to the **strongest layer that can hold it**:
  1. mechanically enforceable → ESLint / tsconfig / CI,
  2. harness-enforceable → `.claude/settings.json` permission rules,
  3. only what is neither derivable from the repo nor enforceable → prose in the root
     `CLAUDE.md`. Budget ≤60 lines; a fresh repo should land well under 30 and grow only with
     evidence (a regression traced to a missing line).
- Prose guardrails are probabilistic — they degrade exactly in long, busy, compacted sessions.
  Permission rules are deterministic and bind in **every** permission mode: evaluation order is
  deny → ask → allow → mode default, so an `ask` rule prompts even in auto mode and beats any
  `allow` (code.claude.com/docs/en/permissions.md).

## Step 1 — classify every instruction in `.claude/CLAUDE.md`

Account for every line; nothing is silently dropped. Buckets: **A** derivable from the repo
(drop), **B** mechanically enforceable (move to lint/tsconfig), **C** not derivable (root
`CLAUDE.md` prose), **drop** (generic craft advice or obsolete).

Standard verdicts for the scaffolded Angular block — but **verify each rule name against the
installed angular-eslint first** (`ls node_modules/@angular-eslint/eslint-plugin/dist/rules/` and
the template plugin's rule keys); this table was checked against v21.1.0 and rule sets drift:

| Instruction | Disposition |
| --- | --- |
| strict typing, no `any` | tsconfig `strict` + `no-explicit-any` (usually already active) — verify, drop prose |
| standalone components | `@angular-eslint/prefer-standalone` |
| `inject()` over constructor DI | `@angular-eslint/prefer-inject` |
| signals / `readonly` signal props | `@angular-eslint/prefer-signals` (idioms only — "signals over Subjects/RxJS state" stays prose) |
| OnPush change detection | `@angular-eslint/prefer-on-push-component-change-detection` |
| `host` object over `@HostBinding`/`@HostListener` | `@angular-eslint/prefer-host-metadata-property` |
| `input()`/`output()` over decorators | `prefer-signals` + `@angular-eslint/prefer-output-emitter-ref` |
| `providedIn: 'root'` singletons | `@angular-eslint/use-injectable-provided-in` |
| relative template/style URLs | `@angular-eslint/relative-url-prefix` |
| native control flow `@if`/`@for` | `@angular-eslint/template/prefer-control-flow` |
| `NgOptimizedImage` for static images | `@angular-eslint/template/prefer-ngsrc` (template `<img>` only) |
| accessibility (focusable + key events on interactive elements) | `angular.configs.templateAccessibility` preset (static checks only) |
| no `ngClass`/`ngStyle`; reactive forms; WCAG AA runtime half; signals-first state | **no rule exists** (as of 21.x) → the prose Conventions block |
| small/focused components, inline templates for small ones, async pipe, "don't assume globals", lazy-load feature routes | drop — generic craft or judgment calls |
| no `mutate` on signals | drop — the API was removed in Angular 17; the rule is obsolete |

## Step 2 — audit read-only, then enable

Before touching `eslint.config.js`, write a temporary flat config **outside the repo** (e.g. the
scratchpad) that spreads the repo's config and adds all Step-1 rules as `error`; run
`npx eslint src --config <tmp> --format json` and count violations per rule. Then fix **all**
violations (a fresh scaffold typically has only a handful, e.g. the generated root component
lacking OnPush) and add the rules to the repo's `eslint.config.js` as `error`. Field notes:

- `prefer-signals` findings are usually just missing `readonly` — mechanical.
- Spec files are linted too, including inline test `@Component`s.
- `ngSrc` conversions need `NgOptimizedImage` in the component's `imports`, numeric `width`/
  `height` (no units) or `fill`, and a check of the CSS so the new attributes don't change layout;
  mark the LCP image `priority`.
- Run Prettier over every file you touch.

## Step 3 — CI gate

If a workflow already runs tests/builds, add `npm run lint` (per npm project) so a violation
blocks the pipeline. If the repo has no CI yet, record in the policy doc (Step 6) that the lint
rules are unenforced until a CI lint step exists — without CI, Step 2 is theater.

## Step 4 — write `.claude/settings.json` (checked in, verbatim)

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

## Step 5 — replace the memory files

Delete `.claude/CLAUDE.md`. Write the root `CLAUDE.md` from this skeleton (target: under 30 lines
for a fresh repo; hard cap 60):

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

## Conventions not enforceable by lint (the enforced ones live in `eslint.config.js`)

- Signals-first state: services hold private writable signals, expose `.asReadonly()` + `computed()`.
- Reactive forms over template-driven; `class`/`style` bindings, never `ngClass`/`ngStyle`.
- WCAG AA is a hard requirement (focus, contrast, ARIA); template lint covers only the static part.
```

Sections for **Invariants** and **Traps** are added later, only when the project has earned them:
an invariant is something whose violation compiles and passes tests; a trap is an intentional
absence or a prod-only failure mode. A fresh repo has neither — do not pad. Likewise, when the
project later grows a version-pinned external API surface (Shopify, Stripe, Firebase, …), add one
Process bullet mandating research of the current official docs before writing code against that
surface, naming the single file that records the pinned version.

## Step 6 — write `docs/CLAUDE-CODE-POLICY.md`

The decision record future sessions must read before touching `CLAUDE.md`. Contents: the size
budget and its raise-triggers (evidence only); a what-lives-where table (prose / lint+CI /
settings.json / docs / auto-memory); the prose→lint map **with the audited per-rule violation
counts from Step 2**; the guardrail design plus its residual gaps (interpreter indirection via
allowed `node`/`npm`, leading env assignments like `GIT_DIR=x git push`, un-enumerated plumbing,
the human "don't ask again" click); and the traps: removing the CI lint step silently reverts
conventions to prose, `Write(path)` rules are invalid, and any session predating
`.claude/settings.json` is guarded only by prose.

## Step 7 — verify and report

`npm run lint`, `npm test`, and the production build must all pass. Then report: per-rule
violation counts found and fixed, the final `CLAUDE.md` line count, anything this repo lacked
(no CI, missing rules in the installed angular-eslint version), and a suggested branch name +
one-line conventional commit message. Do not commit.
