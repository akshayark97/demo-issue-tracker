---
name: review-baseline
description: Known accepted gaps and recurring weakness class from the 2026-06-12 full-codebase review
metadata:
  type: project
---

Recurring weakness class in this codebase: API routes pass `req.json()` bodies to the typed store without runtime validation. The `Status` union is only enforced at compile time, so an invalid `status` string can enter the store (via POST body, PATCH body, or the unvalidated `status as Status` cast in the reorder route) and crashes `Board.tsx`'s `byStatus` lookup (`map[i.status].push` on undefined). PATCH can also overwrite `id` via `Object.assign`, desyncing the Map key.

**Why:** First full review (2026-06-12) found no validation layer; flagged as the main must-fix theme. Intentionally out of scope: DB, auth, middleware, test suite — do not re-flag those.

**How to apply:** In future reviews, check whether new/changed API code validates `status` against `STATUSES` and whitelists patch fields. If a validation helper has since been added to `lib/`, this memory is resolved — update it.
