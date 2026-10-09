# 50heads CLI

Ask verified people from a script, a Makefile or CI, wait for the answers and read the split. 20 credits an answer at Tier 1, priced in your account's currency; fifty answers take about ten minutes.

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
50heads status
50heads logout
```

The same sign-in works for `npx @50heads/mcp`. The API key wins when both are there.

## Ask

```sh
# Price and time first. Never spends.
50heads estimate --text "Which name sounds friendlier?" --option Pip --option Moss
# 50 answers, about 10 minutes, $12.70. 20 credits an answer.

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

### Many at once

```sh
50heads ask --csv asks.csv --json > asked.jsonl
cat asks.jsonl | 50heads ask - --json
```

CSV columns (a header row, any order): `text` (required), `type`, `options` (`A|B|C`) or `option1` to `option8`, `images` (`|`, in option order), `context`, `language`, `n`, `tier`, `rush`, `neither`, `mostly` (on a yes/no row, adds Mostly; otherwise the row is Yes or No), `countries` (`|`), `tags` (`|`), `stimulus_image`, `stimulus_text`. Every row is checked before anything is asked; a row that fails is reported and the rest go ahead (exit 8).

### Never twice by accident

Every ask carries an idempotency key made from a hash of the question. Running the same command or file again, or retrying after a dropped connection, returns the same question within 24 hours and never charges twice. To ask the same question again on purpose, pass `--new` (or `--salt something` for a whole file). `--key` sets the key yourself.

## Read

```sh
50heads wait q_123 --timeout 900          # waits on the server, prints the result
50heads results q_123                     # the result so far
50heads answers q_123 --option 0 --limit 100
50heads answers q_123 --q price --all --json    # one answer per line
50heads list --status live
50heads cancel q_123                      # stops it; unanswered heads are refunded
50heads balance
50heads templates
```

`answers` filters by `--option` (0 is the first), `--tier`, `--country`, `--age-band` and `--q` (a word in written answers). Country and age filters show nothing when fewer than 5 answers match. Answers never identify a head.

## Output and exit codes

Human-readable by default. With `--json`, each result is one JSON object on stdout (JSON Lines for many), progress stays on stderr (`--quiet` turns it off), and errors are JSON on stderr: `{"error": {"code", "message", "exit"}}`.

| Exit | Meaning |
| --- | --- |
| 0 | Done |
| 1 | Something went wrong |
| 2 | The command line is wrong |
| 3 | Not signed in, or the key is not allowed to do that |
| 4 | Not enough credits, or over the key's daily cap |
| 5 | The question needs fixing (estimate prints the fixes) |
| 6 | No question with that id |
| 7 | 50heads is unavailable or rate limited; try again |
| 8 | Some asks in a bulk run failed; the rest were asked |
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

## Versions

Calendar versions, `YYYY.MDD.N`, released from the monorepo by `.github/workflows/release-cli.yml`. See `CHANGELOG.md`.

## Development

```sh
pnpm --filter @50heads/cli test
pnpm --filter @50heads/cli build && node packages/cli/dist/cli.js --help
```

The sign-in and credential store are in `packages/login`, shared with `@50heads/mcp`. Commands are in `src/run.ts`; `run()` takes its I/O as an argument, so the tests drive it in-process against the fake API from `@50heads/mcp/testing`.
