# EVALS — create-skill

3 scenarios để test create-skill có hoạt động đúng không. Chạy ở phiên Claude Code mới (fresh context).

## Eval 1: Golden path — Task skill với type rõ ràng

**Tên scenario**: Tạo Workflow skill từ description đầy đủ

**User input**:
```
/create-skill

Tôi muốn tạo skill tên "deploying-railway" để deploy 1 Next.js app lên Railway.
Pipeline: check git clean → build local → push to Railway → smoke test → report URL.
2 ví dụ: "deploy app này lên Railway" hoặc "/deploying-railway".
```

**Expected behavior**:
1. Skill `create-skill` triggered
2. SKIP Step 1 (user đã cung cấp đủ examples + spec)
3. Classify: Task → Workflow (vì có pipeline + deliverable cụ thể)
4. Hỏi 4 câu chuyên biệt theo Workflow type-guide (deliverable, dependencies, decision points, recovery)
5. Đề xuất 2-3 candidate names gerund-form (eg: `deploying-railway`, `deploying-to-railway`, `railway-deploying`)
6. Generate frontmatter third-person với ≥5 trigger phrases (cả VN + EN)
7. Generate body ≤500 lines, imperative form, có Pipeline 5 bước, Decision points table, Recovery section
8. Tạo EVALS.md với 3 scenarios
9. Self-validate theo validation-checklist hard gates
10. Report: path + files + cách invoke

**Pass criteria**:
- [ ] Skill `create-skill` triggered (không gọi nhầm skill khác như `webapp-deployment`)
- [ ] Classify đúng là Workflow (không Generator hay Research)
- [ ] Frontmatter pass hard gates (lowercase+hyphens, ≤64 chars, no reserved word, description ≤1024 chars third-person)
- [ ] Body ≤500 lines
- [ ] EVALS.md tồn tại với 3 scenarios
- [ ] Final report có path + invoke command

---

## Eval 2: Edge case — Reference skill (no pipeline)

**Tên scenario**: User mô tả skill là "knowledge Claude áp dụng song song" — phải classify là Reference, không Task

**User input**:
```
/create-skill

Tạo skill tên "seongon-brand-voice" — chứa voice guideline của SEONGON.
Khi tôi viết content trong project, Claude tự áp dụng voice này.
Không có command cụ thể, không có file output — chỉ là kiến thức Claude áp dụng.
```

**Expected behavior**:
1. Skill `create-skill` triggered
2. Detect đây là Reference skill (no deliverable, applied alongside)
3. KHÔNG generate Pipeline section (Reference skill không có numbered steps)
4. Generate body với cấu trúc Reference: "Khi nào áp dụng" + "Core principles" + "Reference files" + "Anti-patterns"
5. Hỏi user về domain (voice descriptor, examples, banned words)
6. Tạo references/voice-rules.md, references/examples.md (knowledge tách ra)
7. EVALS.md với 3 scenarios theo Reference focus (golden = Claude consult skill khi viết content; edge = chạm nhẹ domain; anti = task ngoài domain không over-apply)

**Pass criteria**:
- [ ] Classify đúng là Reference (không Task: Voice — phân biệt được nuance)
- [ ] Body KHÔNG có "Pipeline N bước" section
- [ ] Có "Khi nào áp dụng" với trigger conditions ngầm (không phải `/skill-name`)
- [ ] Knowledge dài tách ra references/, không nhồi vào SKILL.md
- [ ] EVALS focus đúng — test Claude tự consult skill khi context khớp domain

---

## Eval 3: Anti-pattern — User input vi phạm constraint

**Tên scenario**: User propose skill name có reserved word + description quá vague

**User input**:
```
/create-skill

Tên: claude-helper-tool
Description: Helps with stuff related to Claude.
Mục đích: làm việc với Claude API.
```

**Expected behavior**:
1. Skill `create-skill` triggered
2. Step 4 validate name → DETECT reserved word "claude" → REJECT
3. Đề xuất 2-3 candidate names alternatives KHÔNG chứa reserved word (vd: `assisting-with-api`, `building-api-integrations`, `processing-api-requests`)
4. Step 5 detect description vague ("Helps with stuff") → REJECT
5. Hỏi user 1 câu clarifying: skill cụ thể làm gì? (extract data? generate code? review prompts?)
6. KHÔNG tự generate skill theo input naive — phải clarify trước
7. Output rõ tại sao reject + alternatives

**Pass criteria**:
- [ ] Skill DETECT name vi phạm reserved word constraint
- [ ] Skill DETECT description vague (không pass quality bar)
- [ ] Skill KHÔNG generate skill với name + description naive — phải clarify trước
- [ ] Có alternatives đề xuất CỤ THỂ (không chỉ "đặt tên khác đi")
- [ ] User hiểu vì sao bị reject + biết cần làm gì tiếp

---

## Cách chạy evals

### Manual test

1. Mở phiên Claude Code MỚI ở workspace có create-skill installed
2. Verify skill xuất hiện trong skills list
3. Copy User input từng scenario, paste vào CC
4. So Claude's response với Expected behavior
5. Tick Pass criteria items

### Notes về test create-skill specifically

- Eval 1 (Golden) test khả năng classify + generate full pipeline cho dạng phổ biến (Workflow)
- Eval 2 (Edge) test khả năng phân biệt Reference vs Task — đây là phân loại quan trọng nhất
- Eval 3 (Anti) test guardrails — không cứ generate khi input invalid

Nếu tất cả 3 eval pass → create-skill ready cho production use cho cả Viet (own workflow) và team SEONGON (đào tạo).

### Iteration

Nếu Eval 1 fail (golden Workflow) → check:
- Classification logic ở Step 2
- Type-guide cho Workflow ở `references/type-guides.md`
- Template skeleton ở `assets/skill-template.md`

Nếu Eval 2 fail (Reference skill bị treat như Task) → strengthen:
- Decision tree ở `references/skill-taxonomy.md`
- Step 2 instruction trong SKILL.md về Reference vs Task

Nếu Eval 3 fail (skill cứ generate junk) → strengthen:
- Validation hard gates ở Step 4 (name) và Step 5 (description)
- `references/anti-patterns.md` mục 1 (vague description) và mục 9 (reserved word)

---

## Eval results log

| Date | Skill version | Eval 1 | Eval 2 | Eval 3 | Notes |
|---|---|---|---|---|---|
| 2026-05-15 | v0.2 | PASS (simulated) | PASS (simulated) | PASS (simulated) | Round 2 audit — pending manual test in fresh CC session |
