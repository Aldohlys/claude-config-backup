---
name: project-bot-forward-test
description: "BOT forward test (bot_fwd) — simulates every BOT/BOT-/CT signal of bot_daily with call, spread, stock fallback; where code, tables, report live; decisions; open checks and review date"
metadata:
  node_type: memory
  type: project
  originSessionId: 19e73193-152c-4f28-8ad3-65c915a3c317
  modified: 2026-10-08T17:46:18.186Z
---

Built 2026-10-07/08. Code `RApplication/RStudies/reports/bot_fwd/` (Python, ib_async direct), run after every `bot_daily` by `NewTrading/scripts/2_run_bot_daily.bat` (log `RApplication/logs/bot_fwd.log`). Design + decisions + implementation notes: `NewTrading/Strategies/Breakouts/bot_forward_test_proposal_20261007.md` (§8 decisions, §9 implementation, §9.4 decisions of 10-08).

- Tables `bot_fwd_signal/position/contract/mark/result/underlying` in mydb.db; weekly report `NewTrading/Reports/bot_fwd_<Monday>.md`.
- Backfilled 2026-09-21 → 10-07 on 10-08 (182 signals entered, 384 positions). 09-21 is the earliest bot_daily file with `tradable`; files 09-25..09-29 09:21 have no `atr` (ATR14 computed instead).
- Decisions: 1 lot; every tier appearance = independent trade (headline = run_day 1); stock fallback; bid-ask limit 20%; monthly up to 49 DTE when none in 28-42; P0 has no `no_asym` exit (P7 adds it); spread = long ask / short mid; skip entries already through the stop.
- Stock sizing floor (2026-10-08): risk/share and 1R = max(ask - stop, 0.5 ATR) (`STOCK_MIN_RISK_ATR`), exit stop unchanged. Fixed DUOL 09-28 (+42 R, 1R \$88 -> +4.5 R) and HYG 09-28 (1R \$9); 10 existing stock rows resized in place, backup CSV of old rows was in session scratchpad.
- `bot_fwd_signal.tier_reason` (2026-10-08): daily_trend / weekly_trend_hold / counter_trend — keeps paused-trend BOT- rows separable in the BOT- control at the review. Former COUNTER-TREND rows are tier WATCH + reason counter_trend and still simulated (`REASONS_SIMULATED`), the hidden "bought the falling knife" baseline; report groups = `C.GROUPS` (tier/reason). First read 10-08 (72 signals): 5 sessions later lower on average, up 20/66; a one-day rebound on the signal bar did not improve R — test multi-day confirmation (higher low, reclaim) at the ~11-02 review.
- BOT- tier is a control group by design (few winners expected); keep it simulated, report it apart from BOT, never "fix" it.
- Recorded, not gated: `entry_ext_atr` (entry vs signal price; bot_daily px = last completed session, entry later — SNPS 10-01 entered +4.9 ATR past signal, beyond target) and `entry_asym`. Whether to skip extended entries is the user's call once data exists.

**Why:** measure scanner list + mechanical exits, kept apart from the real BOT book (TODO S-4: different population).

**How to apply:**
- Live path verified 2026-10-08 17:13 (16 `[live]` entries). From that evening bot_daily writes `Reports/bot_daily_<date>_<hhmm>.xlsx` and intake reads its Detail sheet; older runs are still read from CSV + `bot_daily_detail/` sidecar (xlsx preferred when both exist for one stamp). Open check: the 10-09 17:10 task is the first run producing only the xlsx — verify intake logs entries for 10-09.
- Don't run bot_daily by hand just to test something: every bot_daily file is a forward-test entry source, so an extra morning run becomes the session's first run and moves that day's entries (priced from history at open + 30 min instead of live at 17:10).
- First real review ~2026-11-02, when the September entries (Oct 23-30 expiries) are all closed; before that closed results are mostly early stops.
- IBKR has no daily option bars (hourly BID/ASK used); first history request per option contract 6-17 s — keep ≤5 in flight. Don't kill a running bot_fwd process: TWS refused API connections afterwards (2026-10-07). Long runs: start detached; background watchers get reaped under memory pressure.

Related: [[project-bot-book-design]], [[project-bot-universe-scanner]]
