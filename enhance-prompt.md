---
description: Sharpen a raw prompt into a precise work order for this session
disable-model-invocation: true
---

Sharpen the raw prompt below into a precise work order for this Claude Code session
in this repository. Sharpening is the entire task for this turn — you do not carry
out the prompt and you do not start any implementation.

Use whatever repo context you have or can obtain for this: project instructions,
available skills and commands, conventions in the affected code. Resolve
abbreviations, ticket keys, and file, component, and symbol names from the raw prompt
against what actually exists in the repo, and name them concretely in the result
instead of paraphrasing them.

Do not ask follow-up questions. Where the raw prompt is open, pick the most obvious
reading and write it into the prompt as an explicit assumption ("Assumption: …"), so
that I can spot and correct it while reading.

The sharpened prompt

- is written in the language of the raw prompt and gives the task to you instead of addressing me,
- states the goal and the desired end state in its first sentence,
- names scope, affected places, and constraints as concretely as the repo allows,
- says what is explicitly out of scope,
- says how the result can be verified,
- stays as short as it needs to be for that: no role assignment, no filler phrases, no justification of why the task matters, no restating in other words,
- will afterwards be entered by me, unchanged, in a new session. Where it refers to me as the requester (e.g. a later approval/decision), it stays with "I"/"me" — never my name or a third person. Only the instructions themselves are addressed to you as a direct order.

Output: only the sharpened prompt, in exactly one code block, without a single word
before or after it — no heading, no introduction, no summary, no list of your changes.
No further code fences (three backticks) inside the block; if you need examples,
indent them instead.

Raw prompt:

$ARGUMENTS
