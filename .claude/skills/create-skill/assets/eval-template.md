# EVALS.md TEMPLATE — 3 scenarios for testing new skill

Pattern: Golden / Edge / Anti
- Golden: happy path, input chuẩn → output chuẩn
- Edge: ambiguous/incomplete input → skill graceful
- Anti: input mà skill nên TỪ CHỐI hoặc clarify, không cứ làm

Cách dùng EVALS.md sau khi skill được generate:
1. Mở phiên Claude Code MỚI (skill chưa load context)
2. Gõ User input của từng scenario
3. So output Claude với Expected behavior
4. Pass nếu match Pass criteria

Replace [BRACKETED] placeholders trước khi commit.

---

# EVALS — [SKILL_NAME]

3 scenarios để test skill có hoạt động đúng không. Chạy ở phiên Claude Code mới (fresh context).

## Eval 1: Golden path

**Tên scenario**: [SHORT_DESCRIPTOR — vd: "Standard input, full output"]

**Precondition** (chỉ dùng cho Workflow/Analyzer skills — bỏ section này nếu không áp dụng):
- [REPO_STATE_REQUIRED — vd: "Next.js project có clean git tree", "FastAPI app với requirements.txt"]
- [TOOL_INSTALLED — vd: "railway CLI authenticated", "Chrome headless available"]
- [ENV_VAR_SET — vd: "ANTHROPIC_API_KEY in .env"]

**User input**:
```
[EXACT_PROMPT_USER_GÕ]
```

**Expected behavior**:
1. Claude triggers skill `[SKILL_NAME]` (visible trong skills list hoặc context)
2. [SPECIFIC_BEHAVIOR_1 — vd: "Hỏi 1 câu clarifying duy nhất, không 3 câu"]
3. [SPECIFIC_BEHAVIOR_2 — vd: "Generate output đúng format X với Y components"]
4. [SPECIFIC_BEHAVIOR_3 — vd: "Self-validate trước khi report"]
5. Output có đầy đủ: [LIST_REQUIRED_COMPONENTS]

**Pass criteria**:
- [ ] Skill triggered đúng (không gọi nhầm skill khác)
- [ ] Output match format spec trong SKILL.md
- [ ] [QUANTITATIVE_CRITERION — vd: "≥40 trang", "≥3 sections", "≤5 phút"]
- [ ] [QUALITATIVE_CRITERION — vd: "Citations đầy đủ", "Voice đúng", "Imperative form"]

---

## Eval 2: Edge case

**Tên scenario**: [SHORT_DESCRIPTOR — vd: "Input thiếu key parameter" / "Ambiguous topic"]

**User input**:
```
[PROMPT_VỚI_AMBIGUOUS_HOẶC_INCOMPLETE_INPUT]
```

**Expected behavior**:
1. Claude triggers skill `[SKILL_NAME]`
2. Detect ambiguity / missing data / edge condition
3. [GRACEFUL_HANDLING — vd: "Hỏi user 1 câu clarifying" hoặc "Áp dụng default + note rõ"]
4. KHÔNG fabricate / hallucinate / fake data để fill gap
5. Output có note rõ về assumption hoặc data gap

**Pass criteria**:
- [ ] Skill detect được edge condition (không cứ làm như normal input)
- [ ] [SPECIFIC_GRACEFUL_BEHAVIOR — vd: "Note 'data unavailable for X' thay vì bịa"]
- [ ] Output partial vẫn useful, không error out hoàn toàn
- [ ] User có actionable next step (vd: "Cung cấp thêm info Y để re-run")

---

## Eval 3: Anti-pattern (skill nên từ chối / redirect)

**Tên scenario**: [SHORT_DESCRIPTOR — vd: "Input vi phạm policy" / "Out-of-scope request" / "Conflicting goals"]

**User input**:
```
[PROMPT_MÀ_SKILL_KHÔNG_NÊN_FULFILL_NHƯ_REQUESTED]
```

**Expected behavior**:
1. Claude triggers skill `[SKILL_NAME]` HOẶC redirect tới skill khác phù hợp hơn
2. Detect anti-pattern condition: [SPECIFIC_REASON]
3. KHÔNG cứ generate output theo input — phải clarify/redirect/reject
4. Output thay thế: [WHAT_SKILL_SHOULD_DO_INSTEAD — vd: "Suggest reframe", "Redirect tới /<other-skill>", "Explain why không phù hợp"]
5. Tone professional, không lecture

**Pass criteria**:
- [ ] Skill KHÔNG generate output naive theo input (chứng tỏ có guardrail)
- [ ] Có explanation rõ tại sao không proceed
- [ ] [REDIRECT_OR_REFRAME_SUGGESTION — vd: "Suggest dùng /other-skill thay thế"]
- [ ] User hiểu cần làm gì tiếp

---

## Cách chạy evals

### Manual test (recommended cho non-tech)

1. Mở phiên Claude Code MỚI ở repo có skill installed
2. Verify skill xuất hiện: gõ `/skills` (nếu CC hỗ trợ) hoặc check context
3. Copy User input từng scenario, paste vào CC
4. So Claude's response với Expected behavior
5. Tick Pass criteria items

### Notes

- Chạy ở fresh context để test triggering thực tế (không bias bởi context đã có skill)
- Test với cùng input 2-3 lần để check consistency
- Nếu fail Eval 1 (golden) → skill có vấn đề core, debug pipeline
- Nếu fail Eval 2 (edge) → thiếu graceful handling, add to anti-patterns section
- Nếu fail Eval 3 (anti) → thiếu guardrail, strengthen "Khi nào KHÔNG dùng" section

### Verify triggering cho Reference skill (special case)

Reference skill KHÔNG có visible output riêng — output user nhận là deliverable của task chính. Để verify skill triggered đúng, dùng 1 trong 3 cách:

1. **Ask Claude post-task**: "Did you consult [skill-name] skill khi làm task này?"
2. **Check Claude's reasoning trace** nếu phiên CC expose extended thinking
3. **Indirect signal**: verify output style/content KHỚP rules trong skill (vd: nếu skill cấm hype words, check output không có hype)

Cách (3) là indirect — Claude có thể coincidentally output đúng style mà KHÔNG consult skill. Dùng (1) hoặc (2) để verify cứng.

### Iteration

Sau mỗi lần edit SKILL.md, chạy lại 3 evals. Eval-driven development ensures skill không drift over time.

---

## Eval results log (optional)

Track results qua các version:

| Date | Skill version | Eval 1 | Eval 2 | Eval 3 | Notes |
|---|---|---|---|---|---|
| YYYY-MM-DD | v0.1 | PASS | PASS | FAIL | Eval 3 fail — added guardrail |
| YYYY-MM-DD | v0.2 | PASS | PASS | PASS | All pass |
