<!--
SKILL.md TEMPLATE — base skeleton

Copy entire file (skip this comment block) → fill placeholders [BRACKETED].
References sections optional based on type — see type-guides.md for per-type structure.

Placeholders convention:
- [SKILL_NAME] = lowercase-gerund-form name (matches folder)
- [DESCRIPTION] = ≤1024 chars, third-person, ≥5 trigger phrases
- [DOMAIN] = what skill is about
- [TRIGGER_*] = specific phrases user says
- [N] = numeric value to fill
-->
---
name: [SKILL_NAME]
description: This skill should be used when the user asks to "[TRIGGER_PHRASE_1]", "[TRIGGER_PHRASE_2_VN]", "[TRIGGER_PHRASE_3]", "[TRIGGER_PHRASE_4_VN]", "[TRIGGER_PHRASE_5]", or [CONTEXT_TRIGGER]. [WHAT_SKILL_DOES_1_SENTENCE]. [WHEN_TO_USE_1_SENTENCE].
---

# [SKILL_NAME]

[1-2 sentence description of what skill does and target audience]

## Khi nào dùng

User nói một trong các pattern:
- `/[SKILL_NAME] [argument-hint]`
- "[TRIGGER_PHRASE_1]"
- "[TRIGGER_PHRASE_2]"
- "[TRIGGER_PHRASE_3]"

KHÔNG dùng skill này khi:
- [NEGATIVE_TRIGGER_1]
- [NEGATIVE_TRIGGER_2]

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| [SETTING_1] | [DEFAULT_1] | [OVERRIDE_CONDITION_1] |
| [SETTING_2] | [DEFAULT_2] | [OVERRIDE_CONDITION_2] |
| [SETTING_3] | [DEFAULT_3] | [OVERRIDE_CONDITION_3] |

## Pipeline — [N] bước

[For Reference skills: replace this section with "Core principles" or "Domain knowledge" — no numbered steps]

Theo thứ tự, không skip.

### Step 1 — [STEP_NAME]

[Imperative instruction. What to do. Exit condition.]

[Optional: tool/command]
```bash
[COMMAND_IF_ANY]
```

### Step 2 — [STEP_NAME]

[Imperative instruction.]

**Decision point**: [WHEN_TO_ASK_USER vs AUTO_PROCEED]

### Step 3 — [STEP_NAME]

[Continue pattern...]

### Step N — [FINAL_STEP]

[Output format. Self-validation. Report to user.]

## Output format

```
[OUTPUT_TEMPLATE]
```

## Tiêu chí chất lượng (self-check)

Trước khi report SKILL READY:
- [ ] [QUALITY_CHECK_1]
- [ ] [QUALITY_CHECK_2]
- [ ] [QUALITY_CHECK_3]
- [ ] [QUALITY_CHECK_4]
- [ ] [QUALITY_CHECK_5]

## Anti-patterns (cấm)

- KHÔNG [ANTI_PATTERN_1]
- KHÔNG [ANTI_PATTERN_2]
- KHÔNG [ANTI_PATTERN_3]

## Common pitfalls

1. **[PITFALL_1_TITLE]** — [explanation + how to avoid]
2. **[PITFALL_2_TITLE]** — [explanation + how to avoid]
3. **[PITFALL_3_TITLE]** — [explanation + how to avoid]

## Skill files

[Only include if skill has bundled resources]

| File | Purpose | Khi nào load |
|---|---|---|
| `references/[FILE_1].md` | [PURPOSE_1] | [LOAD_CONDITION_1] |
| `references/[FILE_2].md` | [PURPOSE_2] | [LOAD_CONDITION_2] |
| `assets/[FILE_3]` | [PURPOSE_3] | [USED_IN_OUTPUT_DESCRIPTION] |
| `scripts/[FILE_4]` | [PURPOSE_4] | [EXECUTION_DESCRIPTION] |

## Sample timing

[Optional — only for skills with multi-step pipeline]

- Step 1: [X min]
- Step 2-N: [Y min total]
- Total: **[Z min]**

<!--
END OF TEMPLATE

Reminders before commit:
1. Replace ALL [BRACKETED] placeholders
2. Remove sections not applicable to skill type:
   - Reference skill → no Pipeline section
   - Discovery skill → replace Pipeline with Question Tree
   - Simple skill → no Skill files section
3. Validate body ≤500 lines / 1500-2000 words
4. Check imperative form (no "you should" / "bạn nên")
5. Check forward slashes in all paths
6. Check no emoji, no hype words
7. Run validation-checklist.md hard gates
-->
