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

## Install

```sh
npx -y @50heads/cli@2026.1009.1 --help
npm install -g @50heads/cli@2026.1009.1
```
