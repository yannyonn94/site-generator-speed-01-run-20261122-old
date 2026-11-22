# Faster makefun-site — a 3.7× faster drop-in for makefun-site 1.0.0

A fused single-pass reimplementation of makefun-site 1.0.0: one pass over each Markdown body (no token objects), chunk-scanning inline rendering (plain text copied in slices, never char by char), a hybrid HTML escaper, a memoised slugifier, templates compiled once per case to plain JavaScript functions, and reading minutes counted once per source. On the practice set it returns the reference's answers byte for byte on all 3 cases (1,070 sources, 2.7 MB of Markdown, 0 mismatches) at ~3.7× the speed of the house baseline in-process — 177 ms vs ~650 ms per 3-case run — and ~1.75× end to end in a fresh process.

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/yannyonn94/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/yannyonn94/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

## What it does

`run(input)` returns exactly what makefun-site 1.0.0's `build` returns for the same input: the output tree of every page's `index.html`, the index and tag pages, `build-report.txt` and each static file, byte for byte. Same front matter, template dialect, Markdown subset, slug rule and report — the reference code is the spec wherever the README and the code differ.

## How it is faster

- **Fused Markdown pass.** Blocks render straight to the output array. The reference lexes every body into token objects and then renders them; this build never creates a token.
- **Chunk-scanning inline renderer.** A single regex jumps to the next syntax mark or escapable character; the plain text between marks is copied whole. The reference escapes character by character (one regex replace per character).
- **Hybrid escaper.** Plain text takes one native regex test and returns untouched; text with specials takes a char-code loop.
- **Memoised slugs, per case.** Titles and tag names repeat across posts and pages; each distinct string is slugified once. Nothing is cached from one case to the next — every case is answered alone.
- **Compiled templates.** Each template is parsed once per case and compiled to a plain JavaScript function (with a tree-walking interpreter as fallback if code generation is unavailable), so rendering is direct property access and native loops instead of a node-tree walk per page.
- **No recounting.** Reading minutes are counted once per source in the Markdown phase; a post's card is built once and reused by its page.

## Files

- `main.ts` — the kit's driver, kept as shipped: reads `/input/cases.json`, answers each case with `run`, writes `/output/answers.json`. Turns V8 incremental marking off before importing `solve.js`, as the house baseline does.
- `solve.js` — the faster drop-in. Dependency-free ES module, no install, no network, no clock, no randomness: its output is a function of its input alone.
- `package.json` — `{"type": "module"}` so the `.js` files load as ES modules.
- `LICENSE` — MIT.

## Verify locally

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot . --driver process
```

On the practice set this scores `passed: 1`, `cases_passed: 3/3`, with the same `result_sha256` inputs as the reference's answers (byte-identical `answers.json`, 7,421,485 bytes).
