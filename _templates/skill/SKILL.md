---
name: "your-skill-name"
description: "Describe what this skill does, when it should be used, and what makes it distinct."
---

# Your Skill Name

## When to use
- Describe the task types this skill is for.
- Describe the user signals that should trigger it.
- Describe the boundaries of the skill.

## Resource map
- `references/...`: Add focused reference docs when needed.
- `scripts/...`: Add deterministic helper scripts when repeated logic is needed.
- `assets/...`: Add templates or output assets only when they are part of the result.

## Workflow
1. Load only the references needed for the current task.
2. Prefer deterministic scripts for fragile or repetitive steps.
3. Keep outputs stable and easy to verify.
4. State important fallbacks explicitly.

## Validation
- Check that outputs are structurally correct.
- Check that filenames and paths are stable.
- State any skipped steps or missing tooling.
