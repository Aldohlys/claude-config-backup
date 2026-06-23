---
name: feedback_memory_slug_convention
description: "Memory-store convention — filename, frontmatter name:, and [[links]] must all be the SAME snake_case slug; description: holds the human summary, not name:"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b7c72d7e-945c-4cbf-8075-4bd8bf2373ec
---

In this memory store, keep three things identical for every memory file: the **filename** (minus `.md`), the frontmatter **`name:`** field, and any **`[[link]]`** that points to it. All snake_case (underscores), e.g. `project_fx_risk`.

**Why:** A prior session left the store inconsistent — `name:` fields held descriptive sentences (e.g. `name: User is colorblind`) or kebab-case variants, while filenames and most links used snake_case. `[[links]]` resolve by this slug, so the mismatch produced effectively-dead links. Normalized 2026-06-14: all ~106 files set to `name:` ≡ filename, all links converted to underscore form.

**How to apply:**
- New memory: `name:` = the filename slug, snake_case. Put the one-line human summary in `description:` (that's what recall ranks on), NOT in `name:`.
- Don't write kebab-case `name:` slugs even though the base spec says "kebab-case" — match the filename instead so filename ≡ name ≡ link.
- Two legacy exceptions use hyphenated filenames (and matching hyphen names): [[gcloud-vm]] and [[r-partial-matching]]. Leave them; their links already match.
- An unmatched `[[name]]` is allowed as a forward-marker for a memory not yet written — not an error.
