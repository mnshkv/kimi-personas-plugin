---
name: personas
description: >
  Navigator for the two bundled personas: ponytail (lazy senior dev — the
  smallest complete change, no over-engineering) and caveman (maximum-terse
  answers that keep every technical fact). Use when the user asks "which
  persona should I use", says "persona", "personalities", compares the two
  modes, or is unsure how to turn a mode on or off.
---

# Personas

Two answer-style personas ship with this plugin. Both are prompt-only: no
hooks, no background processes, nothing activates on session start. The user
turns one on by asking, and it stays on until they turn it off.

## Which one

| User wants | Persona | Ask for it with |
|---|---|---|
| Less code, less bloat, the smallest change that fully solves the task | **ponytail** | "ponytail mode", "be lazy", "simplest solution", "yagni" |
| Shorter answers, fewer output tokens, same technical content | **caveman** | "talk like caveman", "caveman mode", "be brief", "less tokens" |
| Review or audit work with the same minimalism applied | **ponytail-review** / **ponytail-audit** | "review this", "audit this codebase" |
| Compress a memory file (CLAUDE.md, todo list) into caveman format | **caveman-compress** | "compress this file" |
| Even terser caveman, or caveman in Classical Chinese | **ultracave** / **megacave** | "ultracave", "caveman ultra", "megacave", "caveman wenyan" |
| A ledger of deferred shortcuts | **ponytail-debt** | "ponytail debt", "list the shortcuts" |
| Ponytail's measured benchmark impact | **ponytail-gain** | "what does ponytail save" |
| The ponytail quick-reference card | **ponytail-help** | "ponytail help" |

The two personas compose fine: "ponytail caveman" means write the least code
*and* reply tersely. Follow both skills when asked.

## Turning off

- ponytail: "stop ponytail" or "normal mode"
- caveman (any level): "stop caveman" or "normal mode"

Levels (ponytail: lite/full/ultra; caveman: caveman/ultracave/megacave) stick
for the rest of the session once chosen.

## Details

Each skill is self-contained — read the one the user picked instead of
summarizing it here. Upstream docs: ponytail at
https://github.com/DietrichGebert/ponytail (MIT), caveman at
https://github.com/JuliusBrussee/caveman (Apache-2.0).
