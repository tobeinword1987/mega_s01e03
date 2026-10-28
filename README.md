# S01E03 · what it shows

Ticket #4471 merged 412 date reports into one "known" issue. It holds two problems.

| Issue | Problem | Reports |
|---|---|---|
| [#1](../../issues/1) | The parser reads dates month first | 379 |
| [#2](../../issues/2) | The booking-v3 form hint says `MM/DD/YYYY` | 33 |

- The page for the kb: `support-kb/src/routes/kb/incidents/4471-split/+page.svx`
- The page as the desk sees it: `rendered/4471-split.png` and the static site in `rendered/site/`
- Counts are computed from the data by `support-kb/src/lib/kb/ProblemCounts.svelte`.
