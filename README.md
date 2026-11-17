# candy-skills

A Claude Code plugin marketplace with four plugins. Each one was tested on real work before it was added here.

```
/plugin marketplace add CandyButcher27/candy-skills
/plugin install vera@candy-skills
```

Install any of `mimi`, `vera`, `scrappy` and `pixi` the same way.

## mimi: project memory

Keeps a `mimi/` folder in each project for what the code cannot tell an agent: current state, fixed issues, decisions with the reasons behind them, and traps. The project's `CLAUDE.md` imports it, so it loads every session with no hook.

- `/mimi-start`: set a project up, or get a brief of where the last session stopped.
- `/mimi-close`: write what the session learned, then check the memory for rot.
- `/mimi`: audit the memory. `/mimi-logging`: show what it costs and saves.

Needs Python 3.11 or newer. Source and evaluation: https://github.com/CandyButcher27/mimi

## vera: pre-push verification

Run `/vera` when a large change is finished and before you push. It:

1. runs your test suite;
2. writes a scope of every place untrusted input enters the change;
3. sends three adversaries on three models (haiku, sonnet, opus) to break it, each in its own git worktree, each finding backed by a repro that ran;
4. re-runs every repro itself and tests any scope line the adversaries skipped;
5. traces unclear causes (`debugger` agent), predicts what a fix could break (`blast-radius` skill), and fixes each finding with the `patcher` agent, using the repro as the test;
6. attacks the fixes again, for at most two rounds, and reports each finding as Complaint, Cause, Fix and Live Check.

It commits each fix on a `fix/vera-<date>` branch and never pushes.

Record so far: on real open-source repositories (GitPython, nltk, pyload, unstructured) the adversaries found new, reproducible bugs, one of which the maintainers later fixed upstream. On a generated Flask shop, vera found and fixed a session forgery that exposed other shoppers' orders, and a regular-expression stall that held one checkout for 9 seconds. All three adversaries missed the stall; vera's own check of the skipped scope lines caught it.

## scrappy: lookups without page dumps

`/scrappy`, or any request to look something up. Every search goes to the `scout` agent, which reads the raw results and returns at most 10 bullets, each with its source URL. Raw mode, through the `scrappy` agent, writes full results to a file when you need exact URLs, image links or page text.

The plugin starts the scrappy MCP server from https://github.com/CandyButcher27/scrappy with `uvx`, so it needs [uv](https://docs.astral.sh/uv/). Search sources need API keys. Run this once to add them:

```
uvx --from git+https://github.com/CandyButcher27/scrappy scrappy setup
```

## pixi: distinctive frontends

`/pixi` builds a landing page or marketing site from a real site screenshot reference, instead of the look a model produces by default. The `pixi-builder` agent runs the same method unattended, for example to build two designs in parallel.

pixi's scripts live in https://github.com/CandyButcher27/pixi. Set `PIXI_HOME` to a clone of that repository, or let the skill clone it to `~/.pixi` on first use. The repository does not include the reference gallery, so on a fresh install references come from scrappy (see above).

## License

MIT
