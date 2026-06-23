---
name: Verify "NEVER do X" policy assertions before deferring to them
description: When a memory, CLAUDE.md, or doc says "NEVER commit/touch/use X", check whether the underlying reason still applies. Surface evidence and ask, don't silently honour.
type: feedback
originSessionId: 66254968-8c04-4ad8-bfc3-2b1d56f655a0
---
When a memory entry, CLAUDE.md, or TODO doc says "NEVER do X" (e.g. "NEVER commit config.yml"), don't silently defer. Verify whether the underlying reason still applies, surface the evidence to the user, and let them re-decide.

**Why:** TODO #55 and CLAUDE.md both said `config.yml` must NEVER be committed because of secrets. A full grep + read across all 7 `config.yml` files showed they were entirely secret-free — the actual secrets (`ALERT_EMAIL_PASSWORD` etc.) live in `Renviron.site`, separated by design. The "NEVER commit" rule was a defensive over-generalization that, if I'd silently honoured it, would have left 7 byte-identical configs un-versioned and growing drift. Surfacing the evidence let the user flip the policy in one round-trip and confirmed a follow-up consolidation goal (#57).

Distinct from `feedback_verify_before_claiming_codebase_facts.md` (which is about not asserting code facts without checking). This one is about not deferring to policy facts without checking.

**How to apply:** When you encounter a "never X" instruction:
1. Identify the stated reason (secrets, blast radius, past incident, etc.).
2. Check whether that reason still holds for the current files (read them, grep for the relevant pattern).
3. If the reason no longer applies, present the evidence to the user and ask whether to keep or relax the rule.
4. If the reason DOES still hold, follow the rule.

Never just nod and skip the work the rule is blocking — that's how policies outlive their justification.
