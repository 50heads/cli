# 50heads CLI

Ask verified people from a script, a Makefile or CI, wait for the answers and see what they think. 20 credits an answer at Tier 1, priced in your account's currency.

```sh
npx -y @50heads/cli --help
# or
npm install -g @50heads/cli
50heads --help
```

Needs Node 20 or later.

## Sign in

For CI and scripts, make an API key in the portal (Developers, API keys) with a daily spend cap, and set it:

```sh
export FIFTYHEADS_API_KEY=fh_live_…
```

On your own machine you can sign in once in the browser instead:

```sh
50heads login     # kept in the macOS Keychain, the Secret Service, or ~/.config/50heads (mode 600)
50heads status    # how this machine is signed in, and who that acts as
50heads limits    # the rate limits on this key, and how much of each is used
50heads logout
```

The same sign-in works for `npx @50heads/mcp`. The API key wins when both are there.

## Ask

```sh
# Price and time first. Never spends.
50heads estimate --text "Which name sounds friendlier?" --option Pip --option Moss
# 50 answers, $12.70. 20 credits an answer.

# Ask and wait for the answers.
50heads ask --text "Which name sounds friendlier?" --option Pip --option Moss --wait

# Two images: an A or B question.
50heads ask --text "Which cover would you pick up?" \
  --image https://example.com/a.png --image https://example.com/b.png --n 100

# From a file: the question draft that POST /v2/questions takes (50heads schema prints its JSON Schema).
50heads ask --file question.json --json
```

`question.json`:

```json
{
  "type": "single_choice",
  "text": "Which name sounds friendlier?",
  "language": "en",
  "options": [{ "label": "Pip" }, { "label": "Moss" }],
  "n": 50,
  "tier": 1,
  "targeting": { "countries": ["GB", "IE"], "tags": [] }
}
```

Wrap it as `{"draft": {…}, "then": [{"text": "Why did you pick {winner}?"}]}` to add follow-ups.

### Go or no go

On a yes/no question, `--go-above 60` asks for a verdict: a go when Yes is above 60% with enough answers, a no go when it is not, and unclear when there are too few answers or the range crosses the line. `--go-min-answers` sets the answers needed (default 80% of `--n`). The line is fixed once the question is live and costs nothing extra. The result prints the verdict first, as how these people answered against your line, not a fact.

```sh
50heads ask --text "Is this button label clear enough to ship?" --go-above 60 --wait
```

Ranking and pairwise results also print who beats whom, and pairwise rows a 95% range on each win share.

### Many at once

```sh
50heads ask --csv asks.csv --json > asked.jsonl
cat asks.jsonl | 50heads ask - --json
```

CSV columns (a header row, any order): `text` (required), `type`, `options` (`A|B|C`) or `option1` to `option8`, `images` (`|`, in option order), `context`, `language`, `n`, `tier`, `rush`, `neither`, `mostly` (on a yes/no row, adds Mostly; otherwise the row is Yes or No), `go_above` and `min_answers` (on a yes/no row, the go line as a whole percentage and the answers needed), `countries` (`|`), `tags` (`|`), `stimulus_image`, `stimulus_text`. Every row is checked before anything is asked; a row that fails is reported and the rest go ahead (exit 8).

### Never twice by accident

Every ask carries an idempotency key made from a hash of the question. Running the same command or file again, or retrying after a dropped connection, returns the same question within 24 hours and never charges twice. To ask the same question again on purpose, pass `--new` (or `--salt something` for a whole file). `--key` sets the key yourself.

## Read

```sh
50heads wait q_123 --timeout 900          # waits on the server, prints the result
50heads results q_123                     # the result so far
50heads answers q_123 --option 0 --limit 100
50heads answers q_123 --q price --all --json    # one answer per line
50heads answers q_123 --format jsonl --out q_123.jsonl   # JSON lines, for scripts and model training
50heads list --status live
50heads cancel q_123                      # stops it; unanswered heads are refunded
50heads balance
50heads templates
```

`answers` filters by `--option` (0 is the first), `--tier`, `--country`, `--age-band` and `--q` (a word in written answers). Country and age filters show nothing when fewer than 5 answers match. Answers never identify a head.

`answers --format jsonl` prints every matching answer as JSON lines (one answer per line), from the API's JSONL export: the question, its options, the answer, `choice` (the answer as option indexes: `index`, `indexes`, `order` or `pair`) and `cohortHash`. `--out FILE` writes it to a file instead of stdout.

`results` ends with the cohort hash: the same value means the same kind of audience was asked (who, how many, which tier and targeting). It never identifies anyone.

## File

```sh
50heads projects create "Spring menu" --description "Names and menus for the spring launch"
50heads projects list                     # with question counts; --archived for archived ones
50heads file q_123 --project prj_1 --label naming --label launch --bookmark
50heads file q_123 --ref JIRA-42          # your own id, set once
50heads labels                            # labels in use, with counts
50heads list --project prj_1 --label naming
50heads projects update prj_1 --archive
50heads projects delete prj_1 --yes       # the questions are kept, outside any project
```

Filing costs nothing. A live question must be canceled before it is archived. A different `--ref` on a question that already has one is refused (exit 5).

## Webhooks

Instead of polling, have 50heads send events to your own https address as they happen. The key or sign-in needs the `webhooks:read` and `webhooks:write` scopes; in a team, changes need the owner or an admin.

```sh
50heads webhooks create --url https://example.com/hooks/50heads \
  --event question.completed --event question.underfilled --label launch
# whe_123  https://example.com/hooks/50heads  on: question.completed, question.underfilled; only labels launch
# Signing secret (shown only now): whsec_…
50heads webhooks test whe_123             # a signed webhook.test event now; exit 1 if it did not get a 2xx
50heads webhooks list
50heads webhooks update whe_123 --pause   # --resume, --event, --url, --project, --label, --clear-filter
50heads webhooks rotate whe_123 --overlap-hours 48
50heads webhooks deliveries --failed
50heads webhooks replay dlv_456
50heads webhooks delete whe_123 --yes
50heads webhooks events --type question.completed --limit 5   # the latest events, as deliveries carry them
```

Events: `question.reserved`, `question.completed`, `question.underfilled`, `question.canceled`, `question.refused`, `credits.purchased`, `credits.low`, `credits.expiring`, `dispute.resolved`, `connection.cap_warning`, `approval.requested`, `approval.decided`, `approval.expired`, `approval.refused`, and for an organization's main team `member.provisioned`, `member.deprovisioned` and `policy.changed`. `--project` and `--label` limit question events to those projects or labels; other events always pass. The secret is printed once, on `create` and on `rotate`; after a rotation the old secret keeps signing for 24 hours unless you say otherwise (0 to 168), so the receiver can switch without missing a delivery.

Each delivery is signed twice: the Standard Webhooks headers (`webhook-id`, `webhook-timestamp`, `webhook-signature`), which the Standard Webhooks libraries verify with the secret as given, and `X-50heads-Signature`. To check a delivery you captured, with no network:

```sh
FIFTYHEADS_WEBHOOK_SECRET=whsec_… 50heads webhooks verify --body-file body.json --headers-file headers.txt
```

`--headers-file` takes `Name: value` lines (as `curl -D` writes them) or a JSON object. The body must be byte for byte as received. `--tolerance 300` also refuses a signature more than five minutes old, as a live receiver should. Exit 5 when it does not verify.

## Slack and Zapier

The Slack workspaces 50heads is added to, and the Zapier accounts connected to it. Asks from either spend your account's credits, up to each one's daily cap. Add a workspace and change its cap in the portal, under Settings, then Integrations; Zapier connects from Zapier.

```sh
50heads apps list
# app_123  Slack · Acme  400 of 2,000 credits spent today  added 2026-10-10 by Ana Price
# conn_456  Zapier  2 hooks  1,200 of 5,000 credits spent today  connected 2026-10-09 by Ana Price
50heads apps remove app_123 --yes   # takes 50heads out of the workspace and revokes its Slack access
```

`list` needs the `apps:read` scope; `remove` needs `apps:write` and, in a team, the owner or an admin. Questions already asked from Slack still finish in 50heads after a removal, but their results are not posted to Slack. A Zapier connection's hooks are its Zaps' triggers; remove the connection in the portal, under Settings, then Integrations, and its hooks go with it.

## Organization (admins)

Read the organization of the team a key acts in. These commands never change anything; domains, sign-in rules, single sign-on, SCIM and service accounts are changed by a person in the portal (Settings, Organization). They need a key (or a sign-in) made for one of the organization's teams by an organization admin, or one of the organization's service accounts, with the scope shown.

```sh
50heads org show                 # organization:read: workspaces, domains, single sign-on, SCIM, sign-in rules
50heads org domains              # organization:read: each domain and the TXT record that proves it
50heads org scim                 # organization:read: people and groups sent, tokens by label and prefix
50heads org sessions usr_123     # members:read: where one person is signed in
50heads org service-accounts     # organization:read: scopes, caps, today's spend, last token
50heads audit list --action domain --limit 50          # audit:read, newest first
50heads audit export --from 2026-10-01 --out audit.jsonl   # audit:read, oldest first, JSON lines (or --format csv)
```

Each export is itself recorded in the audit log. Events never hold secrets, codes or anything that identifies a head.

## Spend approvals

An organization can hold an agent's bigger asks until a person approves them. A held ask is not an error: the credits are held, nothing goes to heads, and `50heads ask` prints the approval and where a person decides, then exits 9.

```sh
50heads ask --text "Replace the paper menu with a QR menu?" --type yes_no --n 50 --wait-approval
# Waiting for approval apr_7QK2M9 … (exit 0 once approved and live; exit 1 if denied or expired)
50heads approvals wait apr_7QK2M9      # wait for a decision on one you already asked
50heads approvals status apr_7QK2M9    # read it once (exit 9 while pending)
```

A denial prints `refused_by_policy` with the approver's role and their reason, never their name. Nobody deciding in time prints `approval_expired`; the credits go back either way. See [Spend approvals](https://50heads.com/help/teams/spend-approvals).

## Output and exit codes

Human-readable by default. With `--json`, each result is one JSON object on stdout (JSON Lines for many), progress stays on stderr (`--quiet` turns it off), and errors are JSON on stderr: `{"error": {"code", "message", "exit"}}`.

| Exit | Meaning |
| --- | --- |
| 0 | Done |
| 1 | Something went wrong |
| 2 | The command line is wrong |
| 3 | Not signed in, or the key is not allowed to do that |
| 4 | Not enough credits, or over the key's daily cap |
| 5 | The question needs fixing (estimate prints the fixes), or a webhook delivery does not verify |
| 6 | No question, webhook endpoint or delivery with that id |
| 7 | 50heads is unavailable or rate limited; try again |
| 8 | Some asks in a bulk run failed; the rest were asked |
| 9 | The ask is held for a person to approve (spend approvals); `ask --wait-approval` waits for the decision instead, and a denial or expiry is exit 1 |
| 124 | `wait` ran out of time and the question is still live |

## In CI

```yaml
- run: npx -y @50heads/cli ask --file .50heads/headline.json --wait --timeout 1200 --json > result.json
  env:
    FIFTYHEADS_API_KEY: ${{ secrets.FIFTYHEADS_API_KEY }}
```

The key's daily cap is the limit on what a runaway job can spend.

## Environment

| Variable | Meaning |
| --- | --- |
| `FIFTYHEADS_API_KEY` | API key for CI and scripts |
| `FIFTYHEADS_API_URL` | API base, default `https://api.50heads.com` |
| `FIFTYHEADS_AUTH_URL` | Sign-in server for `login` |
| `FIFTYHEADS_CREDENTIALS_STORE=file` | Keep the sign-in in a file, not the keychain |
| `FIFTYHEADS_NO_BROWSER=1` | `login` prints the link without opening a browser |
| `FIFTYHEADS_WEBHOOK_SECRET` | The signing secret for `webhooks verify` |

## Versions

Calendar versions, `YYYY.MDD.N`, released from the monorepo by `.github/workflows/release-cli.yml`. See `CHANGELOG.md`.

## Development

```sh
pnpm --filter @50heads/cli test
pnpm --filter @50heads/cli build && node packages/cli/dist/cli.js --help
```

The sign-in and credential store are in `packages/login`, shared with `@50heads/mcp`. Commands are in `src/run.ts`; `run()` takes its I/O as an argument, so the tests drive it in-process against the fake API from `@50heads/mcp/testing`.
