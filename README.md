# kimi-personas-plugin

**Two answer-style personas for [Kimi Code](https://www.kimi.com/code): [ponytail](https://github.com/DietrichGebert/ponytail) — the lazy senior developer who writes the least code that fully solves the task — and [caveman](https://github.com/JuliusBrussee/caveman) — terse, fluff-free answers that keep every technical fact. Prompt-only skills: nothing runs in the background, nothing activates on session start.**

[![License: MIT / Apache-2.0](https://img.shields.io/badge/License-MIT%20%2F%20Apache--2.0-blue.svg)](LICENSE.ponytail-MIT)
[![Platform: macOS / Windows / Linux](https://img.shields.io/badge/Platform-macOS%20%2F%20Windows%20%2F%20Linux-black.svg)]()
[![Kimi Code](https://img.shields.io/badge/Kimi%20Code-plugin-orange.svg)](https://www.kimi.com/code/docs/en/kimi-code-cli/customization/plugins.html)

## Why

Coding agents drift toward over-engineering: extra abstractions, unrequested options, dependencies for five lines, polite filler in every reply. Both upstream projects attack this with a persona the model adopts for the session:

- **Ponytail** constrains the *code*: YAGNI ladder (does it need to exist → stdlib → platform feature → existing dependency → one line → minimum), plus senior-dev review and audit skills that rank what to delete.
- **Caveman** constrains the *prose*: answer first, kill ceremony, never touch a technical fact — output tokens are billed and read.

Neither needs a daemon, a proxy, or an API key to do its main job. This port keeps them that way.

## Features

| Skill | What it does |
|---|---|
| `personas` | Navigator: which persona fits the request, how to turn each on and off. |
| `ponytail` | Lazy senior dev mode for any coding task. Levels: lite / full (default) / ultra. Stays on until "stop ponytail". |
| `ponytail-review` | Senior-dev review of a change (uncommitted, branch, or PR): bugs, security, load, missing tests, speed, bloat — each finding with a concrete case and the smallest fix. |
| `ponytail-audit` | The same review for the whole repo, ranked, max 20 findings. One-shot report, changes nothing. |
| `ponytail-debt` | Grep `shortcut:` (legacy `ponytail:`) comments into a debt ledger with ceiling, upgrade trigger, and rot-risk tags. |
| `ponytail-gain` | Ponytail's published benchmark scoreboard: −53% lines of code, −45% output tokens, −26% cost, −41% time, hidden-check pass rate held. |
| `ponytail-help` | Quick-reference card for levels and skills. |
| `caveman` | Terse-answer persona: fragments allowed, negations and payload never cut, ceremony gone. Stays on until "stop caveman". |
| `ultracave` | Caveman at maximum compression (`/caveman ultra`). |
| `megacave` | Caveman in Classical Chinese, 文言文 (`/caveman wenyan`). |
| `caveman-compress` | Compress a memory file (CLAUDE.md, todos) into caveman format to save input tokens, with a human-readable backup in an out-of-tree data dir. |

## Install

In any Kimi Code session:

```
/plugins install https://github.com/mnshkv/kimi-personas-plugin
```

Then `/plugins reload` (or start a new session).

## Usage

Just ask — the personas turn on by request and stay on for the session:

- "ponytail mode" — smallest complete change from here on; "ponytail ultra" to also get pushback on the request itself
- "review this" / "audit this codebase" — one-shot ponytail review reports
- "ponytail debt" — ledger of deferred shortcuts
- "talk like caveman" — terse answers; "caveman ultra" / "caveman wenyan" for the heavier dialects
- "compress CLAUDE.md" — caveman-compress the memory file
- "stop ponytail" / "stop caveman" — back to normal

Not sure which fits? Ask "which persona should I use" — the `personas` skill answers.

## How it works

Every skill is a `SKILL.md` prompt loaded on demand by Kimi Code; there is no runtime, hook, or MCP server. The only executable code is the optional `caveman-compress` CLI (`skills/caveman-compress/scripts/`, pure Python 3, stdlib only): `detect.py` sniffs file types and refuses secret-looking paths, `validate.py` checks a compressed file against the original for structure and payload drift — both token-free. The compressor's model call can run through the Anthropic API, the `claude` CLI, opencode, or any OpenAI-compatible endpoint (e.g. local Ollama), selected via `CAVEMAN_COMPRESS_PROVIDER`. In Kimi Code the agent can also compress the file itself, in which case nothing leaves the machine.

## Differences from the originals

Ported from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) (MIT) and [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) (Apache-2.0). Honest list of what did not make it:

- **No hooks.** Upstream ponytail ships JS hooks (auto-activation on session start, mode tracking, a statusline "band", session import) and caveman ships a hook that tracks the active mode. Kimi Code has no hook API, so: nothing auto-activates (the default-mode env var / config file from upstream has no effect here), and `caveman status` reports the mode the skill followed plus "(not tracked by this host)", exactly as the upstream skill prescribes for hookless hosts.
- **No `claude` CLI dependency for the core personas.** Both personas are prompt-only upstream too; the caveman Go engine, LLM proxy, browser extension, MCP server, `shrink`, evals, and all other SaaS/infrastructure code are intentionally not ported. Caveman skills that depend on that machinery (`caveman-learn`, `caveman-stats`, `caveman-setup`, `cavecrew`, `caveman-commit`, etc.) are not ported.
- **`caveman-compress` provider path.** The bundled CLI keeps its original providers (Anthropic SDK, `claude` CLI, opencode, OpenAI-compatible). Under Kimi Code the agent compresses the file itself by default, so the external provider is optional. `scripts/benchmark.py` (upstream measurement tooling) was dropped.
- **Ponytail TOML commands not ported.** `commands/*.toml` upstream duplicates the six ponytail skills one-to-one (same names, same content), so porting them would create exact duplicates.
- **Ponytail help card trimmed.** The Claude-Code-specific sections (statusline band, `/plugin` marketplace auto-update, Codex invocation syntax) were removed; the portable parts (levels, skills, deactivation) remain.
- **No `sessionStart`.** Deliberately: personas should be opt-in per session, not forced.

## License & attribution

- Ponytail content (`skills/ponytail*`): MIT, © 2026 DietrichGebert — see [LICENSE.ponytail-MIT](LICENSE.ponytail-MIT). Upstream: https://github.com/DietrichGebert/ponytail
- Caveman content (`skills/caveman`, `skills/caveman-compress`, `skills/megacave`, `skills/ultracave`): Apache-2.0, © 2026 Julius Brussee — see [LICENSE.caveman-Apache-2.0](LICENSE.caveman-Apache-2.0), [NOTICE.caveman](NOTICE.caveman), and [LICENSING.caveman.md](LICENSING.caveman.md). Pre-3.0 MIT parts keep their text in [LICENSE.caveman-MIT](LICENSE.caveman-MIT). "Caveman" and its logo are trademarks of Julius Brussee — see [TRADEMARKS.caveman.md](TRADEMARKS.caveman.md).
- Port to Kimi Code format: © mnshkv.
