# Changelog

## 0.4.0 — Claude Hive ECO

- Real opt-in Claude Code `PostToolUse` hook for reducing verbose successful Bash test/build results, preserving important diagnostics and skipping unfamiliar commands.
- Local, query-ranked, deduplicated source retrieval with a strict approximate context budget, source attribution and line ranges.
- `on`, `off`, `status`, `stats`, and `pack` commands; no model API or third-party token optimizer dependency.
- Usage statistics record numeric-only, approximate result-body size reductions (never commands or raw output).
- Deterministic offline benchmarks, hook integration tests and contributor guidelines.

## 0.3.0 — Knowledge sources

- Explicitly imported public wiki/encyclopedia pages, local text and image metadata.
- Browser graph viewer and user-managed cloud adapters retained from earlier versions.
