---
name: caveman-compress
description: >
  Compress a memory file such as CLAUDE.md or a todo list into caveman format
  to save input tokens, keeping a readable backup. In Kimi Code the agent
  compresses the file itself; an optional Python CLI can do it via the
  Anthropic API, the claude CLI, or an OpenAI-compatible endpoint. Use for
  /caveman-compress, "compress this memory file", "caveman compress <filepath>".
---

# Caveman Compress

## Purpose

Compress natural language files (CLAUDE.md, todos, preferences) into
caveman-speak to reduce input tokens. Compressed version overwrites original.
Human-readable backup saved as `<filename>.original.md`, but NOT beside the
source file — it lives in an out-of-tree data dir
(`$XDG_DATA_HOME/caveman-compress/backups/<parent-dir-name>/`, or
`%LOCALAPPDATA%\caveman-compress\backups\<parent-dir-name>\` on Windows) so
skill auto-loaders don't re-ingest it as a live file.

## Trigger

`/caveman-compress <filepath>` or when user asks to compress a memory file.

**Requires an explicit filepath.** If the user pastes raw text instead of
naming a file, do NOT improvise an inline compression — ask them for the
filepath to compress, or offer to save the pasted text to a file first.
Writing your own reasoning/commentary about the request into the compressed
output (e.g. "User passed text directly, not a filepath...") is the exact
failure this guards against — never let that leak into a saved file.

## Process

Two ways to compress. Prefer the first in Kimi Code.

### Option A — the agent compresses (default, no data leaves the machine)

1. Check the file is safe and compressible:
   `python3 scripts/detect.py <absolute_filepath>` (token-free; refuses
   secret-looking filenames). Skip if the file is code/config.
2. Read the file and compress it yourself, following the Compression Rules
   below. Keep YAML frontmatter and everything in code spans/blocks verbatim.
3. Back up the original to the out-of-tree backup dir (see Purpose), then
   overwrite the file with the compressed version.
4. Validate the result:
   `python3 scripts/validate.py <backup> <compressed>` (token-free). If
   validation reports structure or payload drift, fix the compressed file by
   hand — targeted fixes only, never recompress from scratch.

### Option B — the bundled Python CLI (sends the file to a model provider)

> **Data leaves the machine.** The CLI sends the file's full text to a
> configured model provider. Files whose names look like secrets (`.env`,
> `credentials`, `secrets`, `.netrc`, …) are refused before they are read, and
> a file whose text holds a credential (a private key block, or a cloud, Git
> host or model API token) is refused before anything is sent. Don't compress
> a file the provider shouldn't see.

1. The compression scripts live in `scripts/` (adjacent to this SKILL.md).
2. From the directory containing this SKILL.md, run:
   `python3 -m scripts <absolute_filepath>`
3. The CLI detects the file type (no tokens), calls the provider to compress,
   validates the output (no tokens), cherry-picks targeted fixes on errors,
   and retries up to 2 times. If it still fails, the original file is left
   untouched.
4. Provider, in priority order: `CAVEMAN_COMPRESS_PROVIDER=anthropic` with
   `ANTHROPIC_API_KEY` (needs the `anthropic` Python package),
   `CAVEMAN_COMPRESS_PROVIDER=openai-compat` with `CAVEMAN_COMPRESS_ENDPOINT`
   (defaults to local Ollama) and `CAVEMAN_COMPRESS_API_KEY`,
   `CAVEMAN_COMPRESS_PROVIDER=claude` for the `claude` CLI, or
   `CAVEMAN_COMPRESS_PROVIDER=opencode` for the opencode CLI. Model via
   `CAVEMAN_COMPRESS_MODEL` (or `CAVEMAN_MODEL`).
5. Return result to user.

## Compression Rules

### Remove
- Articles: a, an, the
- Filler: just, really, basically, actually, simply, essentially, generally
- Pleasantries: "sure", "certainly", "of course", "happy to", "I'd recommend"
- Hedging: "it might be worth", "you could consider", "it would be good to"
- Redundant phrasing: "in order to" → "to", "make sure to" → "ensure", "the reason is because" → "because"
- Connective fluff: "however", "furthermore", "additionally", "in addition"

### Preserve EXACTLY (never modify)
- Code blocks (fenced ``` and indented)
- Inline code (`backtick content`)
- URLs and links (full URLs, markdown links)
- File paths (`/src/components/...`, `./config.yaml`)
- Commands (`npm install`, `git commit`, `docker build`)
- Technical terms (library names, API names, protocols, algorithms)
- Proper nouns (project names, people, companies)
- Dates, version numbers, numeric values
- Environment variables (`$HOME`, `NODE_ENV`)

### Preserve Structure
- All markdown headings (keep exact heading text, compress body below)
- Bullet point hierarchy (keep nesting level)
- Numbered lists (keep numbering)
- Tables (compress cell text, keep structure)
- Frontmatter/YAML headers in markdown files

### Compress
- Use short synonyms: "big" not "extensive", "fix" not "implement a solution for", "use" not "utilize"
- Fragments OK: "Run tests before commit" not "You should always run tests before committing"
- Drop "you should", "make sure to", "remember to" — just state the action
- Merge redundant bullets that say the same thing differently
- Keep one example where multiple examples show the same pattern

CRITICAL RULE:
Anything inside ``` ... ``` must be copied EXACTLY.
Do not:
- remove comments
- remove spacing
- reorder lines
- shorten commands
- simplify anything

Inline code (`...`) must be preserved EXACTLY.
Do not modify anything inside backticks.

If file contains code blocks:
- Treat code blocks as read-only regions
- Only compress text outside them
- Do not merge sections around code

## Pattern

Original:
> You should always make sure to run the test suite before pushing any changes to the main branch. This is important because it helps catch bugs early and prevents broken builds from being deployed to production.

Compressed:
> Run tests before push to main. Catch bugs early, prevent broken prod deploys.

Original:
> The application uses a microservices architecture with the following components. The API gateway handles all incoming requests and routes them to the appropriate service. The authentication service is responsible for managing user sessions and JWT tokens.

Compressed:
> Microservices architecture. API gateway route all requests to services. Auth service manage user sessions + JWT tokens.

## Boundaries

- ONLY compress natural language files (.md, .txt, .typ, .typst, .tex, extensionless)
- NEVER modify: .py, .js, .ts, .json, .yaml, .yml, .toml, .env, .lock, .css, .html, .xml, .sql, .sh
- If file has mixed content (prose + code), compress ONLY the prose sections
- If unsure whether something is code or prose, leave it unchanged
- Original file is backed up as FILE.original.md before overwriting — in the out-of-tree backup data dir (see Purpose), not beside the source file
- Never compress FILE.original.md (skip it)
