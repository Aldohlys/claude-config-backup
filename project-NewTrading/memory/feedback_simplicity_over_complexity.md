---
name: Simplicity over complexity
description: User strongly prefers simple rules backed by evidence over sophisticated but unvalidated systems
type: feedback
---

When a complex system (regime detection, sigmoid calibration, macro scoring) doesn't pass empirical validation, simplify or remove it. Don't preserve complexity for its own sake.

Example: 12-signal regime system with softmax normalization → replaced by "VIX > 25 = warning banner". The user explicitly chose the simpler approach after seeing the backtest showed r=0.16 at best.

**Why:** The user has a scientific mindset — correlations of 0.16 are "still weak" and don't justify maintaining machinery. Trading decisions should rest on validated edges, not theoretical frameworks.

**How to apply:** Before building complex features, check if a simple rule achieves the same outcome. Always backtest before calibrating. If the data says "no signal", accept it and simplify.
