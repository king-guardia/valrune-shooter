# data/

Every number that affects balance. Code contains none.

| Path | What |
|---|---|
| `source/` | The original spreadsheet brain dump. **Input, not schema.** Never edit it to fix a schema problem — fix the schema. |
| `schema/` | JSON schemas, enforced in CI. |
| `*.json` | Generated and hand-maintained data. Must validate. |

**Numeric ability parameters may be `null`.** Structure is authored first; numbers come
from the Balance Lab and the calibrator. A `null` is a to-do, not a defect.

Geometry references band ids (`r1`, `dis2`, `w1`), never raw numbers.
