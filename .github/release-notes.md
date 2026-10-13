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

## Install

```sh
npx -y @50heads/cli@2026.1013.1 --help
npm install -g @50heads/cli@2026.1013.1
```
