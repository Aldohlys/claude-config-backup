---
name: reference_china_index_onshore_offshore
description: China is tracked in macro_context as ^HSCE (offshore H-shares) + 000001.SS (onshore A-shares); Yahoo has no CSI 300 history; Hang Seng is not pure China
metadata:
  node_type: memory
  type: reference
  originSessionId: 2ffd0f85-5bcb-48bf-8859-c7a664ed8e6b
  modified: 2026-10-05T04:17:37.164Z
---

The macro_context World markets section tracks China with two indices (RStudies 189c8bf, 2026-10-05):
- Offshore: `^HSCE` (Hang Seng China Enterprises Index, mainland companies listed in HK, HKD). It replaced `^HSI`, because the Hang Seng also holds HK domestic names (HSBC, AIA, HK property), so it is not a pure China read.
- Onshore: `000001.SS` (Shanghai Composite, A-shares, CNY). It is weighted toward state-owned banks and energy and leaves out Shenzhen growth names.

The sector-map China driver for KWEB, FXI and XME is `^HSCE`. Steel, LIT and REMX use `000001.SS`.

Yahoo returns only one row, with no history, for CSI 300 (`000300.SS` and `399300.SZ`), so CSI 300 can't be used through Tdata/quantmod. Re-test it before proposing it again.

Reading: in risk-off moves, offshore usually falls more, because foreign money can leave and the index is heavy in internet names. Onshore is cushioned by state buying. A larger offshore drop points to foreign selling; onshore leading lower points to a domestic or policy cause. During Golden Week (Oct 1–7) onshore is closed while HK trades, so the two closes have different dates.

Same session: in the Europe line, `^FTSE` was replaced by `^FCHI` (CAC 40), `^IBEX` (IBEX 35) and `FTSEMIB.MI` (FTSE MIB). See [[project_intermarket_macro_scenarios]] and [[user_em_china_view]].
