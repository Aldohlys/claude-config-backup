---
name: feedback_user_edits_db_directly
description: "User edits SQLite config tables directly in DB Browser — don't build in-app editors for DB-resident config unless asked"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 676039b8-1e19-48ce-a135-d4119be5ae8d
---

The user keeps DB Browser for SQLite open and is comfortable editing config-style
tables (e.g. `Strategies.MaxRisk`, `Param`) directly. When closing TODO #75 they
chose "I'll edit MaxRisk in DB Browser" over the proposed `Tdata::getStrategyMaxRisk()`/
`setStrategyMaxRisk()` pair + in-app Strategies editor — which dropped that residual
to near-zero work.

**Why:** an in-app CRUD editor for a small, rarely-changed config table is low-value
when the user already has a faster direct path; proposing it inflates scope (here it
would have forced a Tdata rebuild/deploy cascade).

**How to apply:** for changes to small DB-resident config/lookup tables, default to
"edit in DB Browser + restart the app" and only build a getter/setter or UI editor if
the user explicitly asks. Always flag the **cache caveat**: readers that memoise (e.g.
`get_max_strategy_risk` via `.strategy_risk_cache`) need an app restart or
`reset_strategy_risk_cache()` after a direct edit. See [[project_strategies_maxrisk_cap]].
