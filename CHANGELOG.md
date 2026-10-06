# Changelog

Changes to `@50heads/cli`, the `50heads` command. It has its own calendar versions, separate from `@50heads/mcp`.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions are calendar versions, `YYYY.MDD.N`: the year, the month and two-digit day, then the release number for that day (`2026.924.1` is the first release on 24 September 2026). Breaking changes are called out in the notes.

Add what changed under Unreleased as you go, written for the people using it. The release workflow moves it under the new version and uses it for the GitHub Release.

## [Unreleased]

## [2026.1006.1] - 2026-10-06

Maintenance release. No change to tools, resources or prompts.

## [2026.925.1] - 2026-09-25

### Added

- First public release: `ask` (from flags, a JSON file, JSON Lines or CSV on stdin, with `--wait`), `estimate`, `wait`, `results`, `answers`, `list`, `cancel`, `balance`, `templates`, `schema`, `login`, `logout` and `status`.
- `--json` on every command, and exit codes scripts can branch on.
- Asks are keyed by a hash of the question, so re-running a script never asks or charges twice. `--new` asks again on purpose.
- One sign-in for the CLI and `npx @50heads/mcp`.

