# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

x-hots is an OpenClaw skill that fetches real-time trending topics from X (Twitter) and supports interest-domain-based analysis. It is packaged as a skill (`SKILL.md` at repo root defines the skill contract) backed by a single Python script (`scripts/x_hots.py`).

The skill is designed to be driven by an OpenClaw agent: the agent analyzes the user's natural-language input to pick 2–3 search keywords per interest domain, invokes the script with those keywords as a JSON array, then synthesizes background/opinions/controversies from the returned tweets. `SKILL.md` is the authoritative workflow spec — the Python script is just the data-fetching step (Step 2) within it.

## Running the script

The script takes a single JSON-array argument of search keywords and prints JSON to stdout:

```bash
python3 scripts/x_hots.py '["AI", "LLM", "stock market"]'
```

Prerequisites: Python 3.7+ and `tweepy` (`pip3 install tweepy`). The only required credential is `X_BEARER_TOKEN` (app-only OAuth 2.0), exported in the shell environment. There is no build step, no test suite, and no lint configuration in this repo.

Each keyword query returns up to 10 tweets (default `max_results_per_query=10`), filtered to original non-retweet English tweets (`-is:retweet lang:en`), sorted by relevancy.

## Architecture

Two tightly-coupled artifacts; changes to one usually require changes to the other:

- **`SKILL.md`** — the skill definition (frontmatter `name: x-hots`) AND the human/agent-facing workflow, parameter contract, output schema, example invocations, and error-handling reference. The JSON output format documented here must match what `scripts/x_hots.py` actually emits.
- **`scripts/x_hots.py`** — the implementation. Three functions: `create_client()` (reads `X_BEARER_TOKEN`, builds a `tweepy.Client` with `wait_on_rate_limit=True`), `search_hot_topics(client, queries, max_results_per_query=10)` (loops queries, catches `tweepy.TweepyException` per-query so one failing query doesn't abort the rest), and `main()` (parses argv[1] as JSON, assembles the top-level `{success, timestamp, queries, results}` envelope, prints with `ensure_ascii=False`).

Error reporting is JSON-on-stdout + `sys.exit(1)` for fatal errors (missing token, bad JSON argv); per-query failures are kept inside `results` with `success: false` and the process still exits 0.

## Conventions

- The skill/CLI contract is bilingual-aware: English is the default tweet language filter in the script. If you change the language filter or query syntax in `x_hots.py`, update the "Notes" section of `SKILL.md` to match.
- Output is UTF-8 JSON (`ensure_ascii=False`) so tweet text with non-ASCII characters renders correctly — keep this when modifying `main()`.
- This is a skill repo, not an application — keep the surface minimal. New functionality should extend `x_hots.py` and be reflected in `SKILL.md`'s workflow/output sections rather than introducing new entry points.

## Generated files

`scripts/__pycache__/` is committed by accident (a `.pyc` is staged). Prefer not to add Python bytecode; consider a `.gitignore` entry.
