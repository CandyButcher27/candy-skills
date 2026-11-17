---
name: vera
description: Pre-push verification for a finished piece of coding work - runs the project's tests, sends 3 independent adversaries to break the change with runnable repros, confirms each finding, traces unclear causes, patches each finding with its repro as the test, re-attacks the patch, and reports. Use when the user says "/vera", "run vera", "pre-push check", "verify before I push", or finishes a large coding change and wants it checked before pushing. Not for design or visual review, and not for a single small edit.
---
# Vera

Pre-push bug finding and fixing. You run in the main thread, because the steps dispatch subagents. You never push; you ask.

## 0. Set up
- Find the base: `git merge-base HEAD origin/main` (or the branch the user names). The change under review is `git diff --stat <base>`. With no git repo or no base, ask the user what to review.
- If the current branch is `main` or `master`, create `fix/vera-<yyyymmdd>` before any fix. Never commit to main.
- Install the dependencies (a venv via `uv sync`, or `npm install`), so adversaries run the real code instead of stubbing imports.
- A static site with no server code, no form handling and no stored data skips steps 3 to 7. Say so in the report.

## 1. Tests
Run the project's full test suite. A failing test stops vera: report it, with the `debugger` agent (`vera:debugger` when installed as a plugin) first if the cause is unclear. Do not attack code that already fails its own tests.

## 2. Scope
Write `interrogate-scope.md` at the repo root (do not commit it): the files the change touches, and in them every place untrusted input enters or reaches a sink, one line each with `file:line`: routes and handlers, form and query fields, file paths and joins, `subprocess` or shell, SQL strings, regexes run on user input, outbound URL fetches, deserialisation, sessions, secrets and their defaults, auth and ownership checks. Then add an **Intent** list: what the product must never allow (code execution, data loss, one user reading or changing another's data, file access outside its folders, a single request stalling or crashing the server), from the spec or the goal. Adversaries review only what they read, so this list is their scope.

## 3. Interrogate
Run 3 `general-purpose` agents, all in one message, one per model: `model: "haiku"`, `model: "sonnet"`, `model: "opus"`. Three models miss different things; three copies of one model tend to converge on the same bug. Give each its own copy: `isolation: "worktree"` when the session's working folder is the repo; otherwise run `git worktree add ../wt-adv<n> HEAD` yourself, copy `interrogate-scope.md` into each, and name the path in the brief. Each gets this brief with the blanks filled:

> Working folder (a git worktree; work only here): <path>. Scope: the places listed in `interrogate-scope.md`. Intent: <the intent list>. Break this change. Prove each finding with a run against the real code, such as a failing test, a repro script, or command output that shows the wrong behaviour. Use this runtime, which has the dependencies installed: <venv python, or node with node_modules>. Keep repros in your working directory. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run, including style, naming and risks that need a future change. No findings is a valid result. Do not run git commands. Stop every server you start. At the end, list every scope line you exercised.

Three adversaries, not one: in the trials each real bug was often found by only 1 of 3. The brief is the tested form (tested on 3 haiku); the haiku, sonnet and opus mix was set by the owner on 2026-11-17. In its first run (a Flask shop) all three found the session-forgery bug, only opus found the fractional-cart effect, only sonnet the oversized-id 500, and none tried the ReDoS-prone email regex. Do not invoke `pstack:interrogate`.

**Confirm.** A finding is real only after you run its repro yourself. Group findings that share one root cause into one finding. Copy each confirmed repro to `../vera-repros/`, outside the repo, and point its imports at the real tree, so the patcher and the re-attack run it against the code being fixed, not a worktree copy.

**Coverage gap.** On 2026-11-17 the adversaries missed a ReDoS-prone email regex that was in scope, twice: 3 haiku in one run, then haiku, sonnet and opus in the next. Both times this step found it. After the reports, check the scope list against what the adversaries say they ran. For each scope line nobody exercised, try it yourself with one quick run, and treat what you find like any other finding.

## 4. Debugger (only when needed)
Per finding, only when the cause is not obvious from the repro: the `debugger` agent (`vera:debugger` as a plugin; sonnet) with the finding, the repro and its output. It returns the root cause and where the fix belongs. It fixes nothing.

## 5. Blast radius (only when step 4 ran)
A `sonnet` `general-purpose` agent told to read the blast-radius skill first: `../blast-radius/SKILL.md` relative to this skill's base directory (give the agent the absolute path), given the finding, the root cause and the proposed fix location. It reports what could break if the fix lands there and proves one safety fact by a run. Tell it to read history with plain `git log` and to skip `arena`, `why` and `unslop`. Its output goes to the patcher as constraints.

## 6. Patch
One `patcher` agent (`vera:patcher` as a plugin) per finding (haiku; `model: "sonnet"` after a failed check), given the finding, the repro command, its output, and any step 5 constraints. Findings that touch the same file run one after another. Add no fix skill to the brief. After each patch, run the repro yourself, then the full test suite. Commit each fix on its own: `fix: <what>`, only the files the patcher changed, by path.

## 7. Interrogate again
Same 3-adversary brief in fresh worktrees from the fixed HEAD, with a new `interrogate-scope.md`: the fix diff (`git diff <commit before the fixes>`), one line per changed place, and each finding's repro path in `../vera-repros/`; each repro must now fail to exploit. A new confirmed finding goes back to step 4. Stop after 2 patch rounds; list what is still open and let the user decide.

## 8. Final run
The full test suite on the final tree, and every repro in `../vera-repros/` once more. Remove the worktrees you created (`git worktree remove`), and delete `interrogate-scope.md`. Keep `../vera-repros/` and name it in the report. Then ask the user about the push.

## 9. Report
One short summary: the base and diff reviewed, the test results, the number of findings, rounds used, and what is still open. Then one block per fixed finding, in this order:
- **Complaint:** what was wrong, and the repro with its output before the fix.
- **Cause:** the root cause and where it lives (the debugger's finding, or "obvious from the repro").
- **Fix:** the files changed, one line each. Not the diff.
- **Live Check:** evidence from runs that happened: the repro after the fix, the re-run on the diff, the final suite. Say what was not run.

End with **Not covered**: files outside the scope list, and scope lines nobody exercised. A clean result means "nothing found in what was read and run", not "no bugs".

Write the report in short, plain technical English yourself, with no writing skill.
