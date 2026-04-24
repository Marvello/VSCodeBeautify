# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

VS Code extension wrapping [js-beautify](https://github.com/beautify-web/js-beautify) to format JS, CSS, and HTML files in-editor. Published as "Beautify" (`HookyQR.beautify`) on the VS Code marketplace.

## Commands

```bash
npm install            # Install deps (runs postinstall to set up vscode test infra)
npm test               # Run tests (launches VS Code extension host via vscode/bin/test)
```

Tests require VS Code and run inside an extension host — they cannot run headlessly via plain Node. Use the "Tests" launch config in `.vscode/launch.json` to debug tests (F5). The test workspace is `test/data/`.

## Architecture

Two files carry all logic:

- **`extension.js`** — Extension entry point. Registers two commands (`HookyQR.beautify` for selection, `HookyQR.beautifyFile` for full file). The `Formatters` class builds VS Code document formatting providers from `beautify.language` config, mapping language types/extensions/filenames to js-beautify's `js`, `css`, or `html` formatters. Re-registers providers on config change and file open.

- **`options.js`** — Config resolution with a specific priority chain: VS Code editor settings → EditorConfig (`.editorconfig`) → `.jsbeautifyrc` (searched recursively from file dir to workspace root, then parent dirs, then `beautify.config` setting, then `~/.jsbeautifyrc`). Exports a single function `(doc, type, formattingOptions) → Promise<config>`. Comments in `.jsbeautifyrc` are stripped before JSON parse.

## Testing

- `test/extension.test.js` — Main test suite. Data-driven: defines input/expected-output pairs per language and `.jsbeautifyrc` setting. Tests both full-file beautify and single-line selection beautify.
- `test/issues.test.js` — Regression tests for specific GitHub issues.
- `test/issues/` — Per-issue test modules (e.g., `265.js`, `273.js`).
- Test `.jsbeautifyrc` is written dynamically per test context into `test/data/`.

## Key Details

- Config supports both array shorthand (`"css": ["css", "less"]`) and object form (`"css": {"type": [...], "ext": [...], "filename": [...]}`) in `beautify.language`.
- Partial/selection formatting sets `end_with_newline: false` to avoid trailing newlines on ranges.
- Lint with ESLint (`.eslintrc.json` at root, separate config in `test/`). JSHint config also present (`.jshintrc`).
- `schema/beautifyrc.json` provides JSON validation for `.jsbeautifyrc` files in VS Code.
