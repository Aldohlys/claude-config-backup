# Save to Memory

Review the current session and persist anything worth keeping to the file-based memory of the **current project**, following the memory rules in the system prompt.

**INSTRUCTIONS FOR CLAUDE:**

This command is project-agnostic. It operates on the current session's memory directory
(`~/.claude/projects/<current-project-key>/memory/`), so it works in any project, not just RApplication.

## Steps

1. **Scan the session** for durable, non-obvious facts worth remembering across sessions. Classify each as one of:
   - `user` — who the user is (role, expertise, preferences)
   - `feedback` — guidance on how to work (corrections or confirmed approaches); include the **why**
   - `project` — ongoing work, goals, constraints not derivable from code/git; convert relative dates to absolute
   - `reference` — pointers to external resources, or non-obvious behaviors discovered by investigation

2. **Filter out** what must NOT be saved:
   - Anything the repo already records (code structure, past fixes, git history, CLAUDE.md, CHANGELOG)
   - Anything that only matters to this one conversation
   - Conversational explanations, trivia, or one-off lookups
   - If asked to remember something the repo already records, save instead what was *non-obvious* about it.

3. **For each fact to keep:**
   - Check for an existing memory file that already covers it → **update** it rather than duplicating.
   - Otherwise write a new file `<slug>.md` with frontmatter (`name`, `description`, `metadata.type`).
   - Filename ≡ `name:` ≡ `[[link]]` slug, all snake_case; the one-line summary goes in `description:`.
   - For `feedback`/`project`, follow the body with **Why:** and **How to apply:** lines.
   - Link related memories with `[[other-slug]]`.

4. **Update the index:** add/update a one-line pointer in `MEMORY.md` (`- [Title](file.md) — hook`). Never put memory content in MEMORY.md itself.

5. **Delete** any memory that this session proved wrong.

6. **Report** a concise list of what was created / updated / skipped (and why skipped).

If nothing is worth saving, say so plainly rather than inventing entries.
