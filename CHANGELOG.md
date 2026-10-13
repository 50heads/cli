# Changelog

Changes to `@50heads/cli`, the `50heads` command. It has its own calendar versions, separate from `@50heads/mcp`.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions are calendar versions, `YYYY.MDD.N`: the year, the month and two-digit day, then the release number for that UTC day (`2026.924.1` is the first release on 24 September 2026).

Pending changes are captured as changesets and written here when the package is released. The GitHub Release uses the same notes.

## [Unreleased]

## [2026.1013.1] - 2026-10-13

**@50heads/cli 2026.1013.1 adds filing, organisation, webhook, answers export and approval flows, plus new status and limits commands.** It also changes login scopes for organisation admin access and updates wording and output to US English, with no change to commands or tools beyond the MCP server tool count.

### Added

- **New filing commands: 50heads projects lists, creates, updates and deletes projects; 50heads labels lists labels in use; and 50heads file moves a question into a project, labels, archives or bookmarks it, or sets a reference once.**
- **50heads webhooks now lists, creates, updates, rotates, tests and deletes endpoints, lists deliveries and replays them, and can verify a captured delivery offline against Standard Webhooks and X-50heads-Signature.**
- **50heads answers --format jsonl now prints each matching answer as JSON lines, with choice and cohortHash, and --out can write it to a file.**
- **New organisation commands read domains, DNS records, SCIM provisioning, sessions, service accounts and audit events, with audit export to JSON lines or CSV.**
- **New status and limits commands show who the sign-in or key acts as, and the rate limits on the key and current usage in the window.**
- **Spend approvals are now supported in ask, including --wait-approval and 50heads approvals wait or status for an approval already asked.**

### Changed

- **login now asks for organization:read, members:read and audit:read so an organization admin can use org and audit, and existing sign-ins and API keys keep their current scopes.**
- **Spelling is now US English throughout, including canceled, unauthorized, organization, authorized and enroll, and ask and estimate now print the answer count and price with no time estimate.**

## [2026.1009.1] - 2026-10-09

**@50heads/cli 2026.1009.1 adds private asks for your own audience, including shared links, answer limits and set publishing from CSV.** It also updates wording and result output to show verified people, tier details and closed links, and adds more language options for ask.

### Added

- **--answered-by private asks your own audience, prints a link to share, lets people answer in the browser, and charges 5 credits for each accepted answer.**
- **--as-set with a CSV and --answered-by private publishes 2 to 10 questions as one set behind one link, answered in order.**
- **ask now accepts ja, ko, sv, da, nb, cs, ro, fi and tr, in addition to the eight languages it already took.**

### Changed

- **A yes/no question with no options is now Yes or No, and --mostly, the CSV mostly column, or type yes_mostly_no adds Mostly.**
- **Descriptions now say verified people, and results and wait lead with who answered; for heads, that line names the tier and how they were verified.**
- **When a shared link closes, results show the status closed.**

## [2026.1006.2] - 2026-10-06

**No user-facing change was recorded for this release.**

### Changed

- **Only the version number moved on.** Commands, flags and output match 2026.1006.1.

## [2026.1006.1] - 2026-10-06

**The help wording for tier 3 changed.** Nothing else in the command output changed.

### Changed

- **The `--tier` help calls tier 3 established heads.** It no longer says trusted panel.

## [2026.925.1] - 2026-09-25

**First public release of the 50heads command.**

### Added

- **`ask` reads flags, a JSON file, JSON Lines or CSV on stdin, and can wait for the answers.** `estimate`, `wait`, `results`, `answers`, `list`, `cancel`, `balance`, `templates`, `schema`, `login`, `logout` and `status` ship with it.
- **`--json` is on every command, and the exit codes are ones a script can branch on.**
- **An ask is keyed by a hash of the question, so running a script again does not ask or charge twice.** `--new` asks again on purpose.
- **One sign-in covers this command and `npx @50heads/mcp`.**

[2026.1006.2]: https://github.com/50heads/cli/releases/tag/v2026.1006.2
[2026.1006.1]: https://github.com/50heads/cli/releases/tag/v2026.1006.1
[2026.925.1]: https://github.com/50heads/cli/releases/tag/v2026.925.1

[2026.1009.1]: https://github.com/50heads/cli/releases/tag/v2026.1009.1

[2026.1013.1]: https://github.com/50heads/cli/releases/tag/v2026.1013.1
