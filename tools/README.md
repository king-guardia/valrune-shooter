# tools/

| Path | Purpose | Phase |
|---|---|---|
| `validate/` | Schema validation for everything in `data/`. Runs in CI. | 2 |
| `coverage/` | Gap report — slot coverage vs targets, per-element triple participation, orphan and neglect detection. | 2 |
| `balance/` | `engine.js`, the single shared math implementation. Imported by both the Balance Lab and the agents so they can never disagree about a number. | 4 |
| `sheets-sync/` | Pull bulk edits down from Google Sheets into `data/`. | 2 |
