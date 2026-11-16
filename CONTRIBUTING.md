# Contributing to Claude Hive

Thank you for helping make the knowledge graph and context optimizer useful.

## Development

- Node.js 20+ for the Claude Code integration and tests; the public web viewer is static.
- Run `npm test` before proposing a change.
- Run `npm run bench:eco` when touching log compression or retrieval.
- Never commit `.claude/hive-eco.json`, local usage events, OAuth secrets, cloud backups, conversation exports or imported source files.
- Avoid automatic crawling of private sources or account access. A user must explicitly choose which content to import.

## What a token-saving benchmark must prove

Synthetic log size reductions alone are **not** a Claude token or cost benchmark. For any claim about real task-level savings, measure paired coding tasks with the same initial state and model, randomized order, correct and complete outputs, equal tools and settings, retries, full input/output/cached token categories, and time to passing tests. Publish task definitions and anonymized results. Compare against a no-plugin baseline before claiming superiority to another plugin.

## Safety and quality criteria

1. Never drop failing-test output, warnings, stderr diagnostics, interrupted tool results or unrecognized command output.
2. Report any omission explicitly, and allow users to turn optimization off.
3. Never treat imported website text, comments or notes as an instruction source.
4. Keep source citations and line references so users can inspect the original.
5. Fail open: a hook crash must not cancel the user's tool execution.
6. Keep cloud credentials out of Git repositories and stats logs.

Open an issue for bugs, misleading source matches, poor diagnostics preservation, accessibility improvements or reproducible quality regressions.
