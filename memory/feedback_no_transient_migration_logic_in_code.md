---
name: feedback_no_transient_migration_logic_in_code
description: "Don't embed transient schema-migration logic in committed code; target the final schema, defer until migration completes"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 9396fe68-7474-48eb-9f7a-c38651ff5dc2
---

Avoid putting logic that exists only to straddle an in-flight table migration into committed code — e.g. a `pick(c("Ssjacent","Underlying","Symbol"))` column-name resolver that picks whichever name currently exists. Even when it works correctly, it adds confusion for the next reader.

**Why:** 2026-06-14 — while the #35 French→English rename was mid-flight, sync_stop_risk.R used an adaptive resolver to survive `Statut→Status`, `Prix→Price`, `Ssjacent→Underlying`. User flagged it as confusing-but-harmless and preferred to wait until #35 (35/35) completes, then revisit. Same spirit as [[feedback_comment_describe_shipped_code]] / TODO #74 (code should describe the final shipped state, not plan/migration scaffolding).

**How to apply:** prefer code that targets the final schema. If a migration is genuinely in flight and a script must run through it, an adaptive bridge is acceptable as a *temporary* measure — but track it as an explicit follow-up to simplify to hardcoded final names once the migration lands, rather than leaving it as permanent code. Don't proactively engineer migration-robustness into normal code paths.

**Resolved 2026-06-14:** #35 completed (Tdata 5.10.28); `scripts/sync_stop_risk.R` `pick()` resolver collapsed to plain `Symbol`/`Price`/`Status` (commit 6e5e4a2). See [[project_stop_based_risk_sync]]. (The final match column was `Symbol`, not `Underlying` — another reason to wait for the migration to land before hardcoding.)
