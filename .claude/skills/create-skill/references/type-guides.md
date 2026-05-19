# Type Guides — Q&A scripts + section structure

## Contents
- Reference skill
- Task: Research
- Task: Workflow
- Task: Generator
- Task: Analyzer
- Task: Voice/Style
- Task: Discovery

Mỗi mục có 3 phần:
1. **Câu hỏi cho user** (đặt 1 message duy nhất ở Step 3)
2. **Section structure** (cấu trúc body SKILL.md đặc thù cho dạng)
3. **Eval focus** (3 scenario tập trung vào gì)

---

## Reference skill

### Câu hỏi cho user

```
Reference skill - Claude áp dụng song song với task khác. Trả lời 3 câu:

1. Domain skill phụ trách là gì? (vd: brand voice, BigQuery schemas, company NDA template)
2. Khi nào Claude nên consult skill này? (trigger conditions ngầm — domain context nào trong conversation)
3. Có file/asset gì cần Claude tham khảo không? (vd: brand-voice.md, schema files, policy docs)
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả domain]

## Khi nào áp dụng
[Trigger conditions chi tiết — context nào của conversation cần consult skill]

## Core principles / Domain knowledge
[Kiến thức chính skill cung cấp — ngắn, ≤300 từ. Detail dài → references/]

## Reference files
[Pointers tới references/*.md, mỗi file có 1 dòng mô tả purpose]

## Anti-patterns
[Cách KHÔNG áp dụng skill này — vd: "Không dùng tone friendly cho CEO communications"]
```

### Eval focus

3 scenarios để test:
1. **Golden**: User task khớp domain → Claude consult skill, áp dụng đúng
2. **Edge**: User task chạm nhẹ domain → Claude vẫn nhận ra cần consult
3. **Anti**: User task NGOÀI domain → Claude KHÔNG over-apply skill này

**Special verification cho Reference**: Reference skill không có visible output riêng — output user nhận là deliverable của task chính. Pass criteria "Skill triggered" verify bằng 1 trong 3:
- Ask Claude post-task: "Did you consult [skill-name]?"
- Check reasoning trace nếu expose
- Indirect: verify output style match rules trong skill (weak signal, không verify cứng)

---

## Task: Research

### Câu hỏi cho user

```
Research skill - input topic → output báo cáo có nguồn. Trả lời 4 câu:

1. Output format: markdown / PDF / slides / khác?
2. Target length: bao nhiêu từ/trang?
3. Nguồn ưu tiên: loại nguồn nào? (vd: McKinsey, government data, academic, trade publications)
4. Vietnam context có bắt buộc 1 chương riêng không? (default: có nếu skill cho team SEONGON)
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả output type + target audience]

## Khi nào dùng
[Trigger phrases]

## Default settings
| Setting | Default |
|---|---|
| Output format | <markdown/PDF/slides> |
| Length | <X từ / Y trang> |
| Language | <VN/EN> |
| Citation style | <numbered/inline> |

## Pipeline N bước

### Step 1 — Clarify scope (optional)
### Step 2 — Spawn research agent (background)
[Brief template trong assets/research-brief-template.md nếu có]
### Step 3 — Skeleton output trong khi agent chạy
### Step 4 — Receive findings, patch placeholders
### Step 5 — Complete sections + citations
### Step 6 — Render output
### Step 7 — Verify + report

## Source priority
1. <highest priority sources>
2. <medium>
3. <fallback>

## Anti-patterns
- KHÔNG bịa số liệu — mọi stat phải có citation
- KHÔNG suy đoán không có chứng cứ
- KHÔNG dùng nguồn >2 năm cho data thay đổi nhanh

## Citation format
[Format chuẩn cho citation]
```

### Eval focus

1. **Golden**: Topic phổ biến (vd: "AI in finance") → output đúng format, ≥X từ, có citation
2. **Edge**: Topic niche / không có nhiều public data → skill biết note "data sparse" thay vì bịa
3. **Anti**: Topic vi phạm policy (vd: medical advice) → skill từ chối hoặc redirect

---

## Task: Workflow

### Câu hỏi cho user

```
Workflow skill - chuỗi bước có deliverable end-to-end. Trả lời 4 câu:

1. Bước cuối deliver gì? (file, report, deployed app, PR, ...)
2. Skill phụ thuộc tool/file/script gì trong repo? (liệt kê absolute paths)
3. Decision points: bước nào skill nên tự dừng để hỏi user vs auto-proceed?
4. Recovery: nếu step giữa fail, skill xử lý thế nào?
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả end-to-end outcome]

## Khi nào dùng
[Trigger phrases + tiền điều kiện]

## Tiền điều kiện
- [ ] <repo state cần có>
- [ ] <tool installed>
- [ ] <env var set>

## Pipeline N bước

### Step 1 — <name>
**Decision point**: <khi nào skip / khi nào proceed>
**Tool**: <command hoặc tool dùng>
**Exit condition**: <khi nào step coi là done>

### Step 2 — <name>
[...]

## Decision points
[Bảng tổng hợp các decision points]

| Step | Hỏi user khi | Auto-proceed khi |
|---|---|---|
| ... | ... | ... |

## Recovery
[Mỗi failure mode → recovery action]

## Output
[Format report cuối pipeline]

## Anti-patterns
- KHÔNG skip validation steps để fast-track
- KHÔNG auto-deploy nếu chưa có user confirm
```

### Eval focus

1. **Golden**: Repo đúng tiền điều kiện → pipeline chạy hết, output đúng format
2. **Edge**: Tiền điều kiện thiếu 1 item → skill detect + báo user + dừng
3. **Anti**: User input ambiguous ở decision point → skill HỎI thay vì đoán

---

## Task: Generator

### Câu hỏi cho user

```
Generator skill - tạo artifact mới (file/code/config). Trả lời 4 câu:

1. Output là gì? (file type + naming convention)
2. Input params skill nhận: bắt buộc gì, optional gì?
3. Có template/skeleton có sẵn không? (path + format)
4. Validation cuối: skill check gì để confirm output đúng?
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả output artifact]

## Khi nào dùng
[Trigger phrases]

## Input schema
| Param | Type | Required | Default | Note |
|---|---|---|---|---|
| name | string | yes | - | <constraint> |
| ... | ... | ... | ... | ... |

## Pipeline

### Step 1 — Validate input
### Step 2 — Choose template
[Trỏ tới assets/templates/ — chọn template nào theo input]
### Step 3 — Fill placeholders
### Step 4 — Generate file(s)
### Step 5 — Self-validate
[Check gì để confirm output đúng]

## Output naming convention
[Pattern: `<prefix>-<slug>.<ext>` hoặc tương tự]

## Templates
[Liệt kê assets/templates/*.* — mỗi cái 1 dòng mô tả khi nào dùng]

## Anti-patterns
- KHÔNG overwrite file có sẵn nếu chưa user confirm
- KHÔNG generate output nếu input invalid
```

### Eval focus

1. **Golden**: Input đầy đủ, hợp lệ → output đúng schema, file parse OK
2. **Edge**: Input thiếu optional param → skill dùng default đúng
3. **Anti**: Input invalid → skill từ chối với error message rõ, không generate junk

---

## Task: Analyzer

### Câu hỏi cho user

```
Analyzer skill - input artifact → output insight có score. Trả lời 5 câu:

1. Input: artifact gì? (URL, file, code, repo)
2. Output: report format gì + có score không (0-100/pass-fail/severity)?
3. Data sources: skill lấy data từ đâu? (API, file system, web fetch, ...)
4. Fallback: nếu primary source unavailable, degrade thế nào?
5. Pre-delivery review: skill verify findings thế nào trước khi report?
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả: input → score/insight]

## Khi nào dùng
[Trigger phrases]

## Source detection
[Bước đầu: detect data sources nào available]

## Analysis framework
### Section 1: <name>
**Sources**: <preference order với confidence weighting>
**Scoring**: <good/warning/critical thresholds>

### Section 2: <name>
[...]

## Scoring rubric
| Factor | Weight | Source preference | Confidence |
|---|---|---|---|
| ... | ... | ... | ... |

**Data sufficiency gate**: nếu <N factors có data → output INSUFFICIENT DATA thay vì score giả

## Fallback cascade
1. Primary source available → confidence 1.0
2. Secondary → confidence 0.85
3. ...
4. Always-available baseline → confidence 0.5

## Pre-delivery review (MANDATORY)
- [ ] Mỗi claim có source label
- [ ] Distinguish "not found" vs "not crawled" vs "below threshold"
- [ ] Cross-check internal consistency
- [ ] Score chỉ output nếu ≥<N> factors có data

## Output format
[Report structure: summary score → sections → critical issues → recommendations]

## Anti-patterns
- KHÔNG output numeric score khi data insufficient
- KHÔNG present inferred data as fact
- KHÔNG skip pre-delivery review
```

### Eval focus

1. **Golden**: Artifact đầy đủ data → score đúng, recommendations specific
2. **Edge**: Artifact thiếu data → skill output INSUFFICIENT DATA hoặc partial score với confidence note
3. **Anti**: Artifact malformed → skill detect + error rõ, không hallucinate findings

---

## Task: Voice/Style

### Câu hỏi cho user

```
Voice skill - review/rewrite content theo voice. Trả lời 4 câu:

1. Voice mô tả ngắn: <tone> + <register> + <audience>
2. Cho 2 cặp BEFORE-AFTER: 1 bad → 1 good (CỤ THỂ, không vague)
3. Anti-patterns: liệt kê words/structures CẤM dùng
4. Output format khi revise: full rewrite / diff inline / suggestions list?
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả voice + audience]

## Khi nào dùng
[Trigger phrases — vd: "/<skill> review my draft", "check voice của bài này"]

## Voice fingerprint
- **Tone**: <descriptor>
- **Register**: <descriptor>
- **Audience**: <who reads>
- **Non-negotiables**: <hard rules>

## Examples (Before → After)

### Example 1: <type of issue>
**Before**: <bad sample>
**Issue**: <why bad>
**After**: <good sample>
**Why fixed**: <explain>

### Example 2-5: [...]

## Anti-patterns (banned/avoid)

### Banned words
- <word 1> — vì sao
- <word 2> — vì sao

### Banned structures
- <structure 1> — vì sao
- <structure 2> — vì sao

### Banned tones
- <tone 1>

## Pipeline khi review

### Step 1 — Read draft
### Step 2 — Identify violations (check banned words/structures/tones)
### Step 3 — Rewrite hoặc suggest changes
### Step 4 — Output theo format đã định

## Output format
[Spec format cụ thể]

## Anti-patterns (cho skill, không phải cho voice)
- KHÔNG sugarcoat critique — direct là core của voice
- KHÔNG override user's intent — chỉ sửa voice, không thay nội dung
```

### Eval focus

1. **Golden**: Draft có ≥3 voice violations → skill detect tất cả + rewrite đúng
2. **Edge**: Draft already on-voice → skill confirm OK, không over-edit
3. **Anti**: Draft có technical content phức tạp → skill chỉ sửa voice, KHÔNG thay đổi technical accuracy

---

## Task: Discovery / Interactive

### Câu hỏi cho user

```
Discovery skill - Q&A có hướng dẫn để khám phá. Trả lời 4 câu:

1. Goal khám phá: skill giúp user làm rõ gì? (vd: requirements, decision criteria, problem framing)
2. Question tree: liệt kê 3-5 câu hỏi opening, mỗi câu có 2-3 nhánh follow-up
3. Stop condition: khi nào skill kết thúc Q&A?
4. Output cuối: summary / plan / recommendation / decision matrix?
```

### Section structure

```markdown
# <skill-name>

[1-2 câu mô tả goal exploration]

## Khi nào dùng
[Trigger phrases]

## Discovery approach
**Style**: <Socratic / directive / mixed>
**Pace**: <bao nhiêu câu hỏi per turn>
**Tone**: <descriptor>

## Question tree

### Opening questions (Q1-Q5)
1. <opening Q1>
   - If answer A → follow-up: <Q>
   - If answer B → follow-up: <Q>
2. <opening Q2>
   - [...]

### Deepening questions (sau khi opening rõ)
[...]

## Stop condition
Skill kết thúc khi:
- [ ] User đã trả lời ≥<N> câu hỏi key
- [ ] Skill có đủ info để generate output
- [ ] User explicit nói "đủ rồi" / "synthesize đi"

## Output cuối

[Format: summary / plan / decision matrix / recommendation]

## Anti-patterns
- KHÔNG hỏi >3 câu trong 1 turn (overwhelming)
- KHÔNG advance sang deepening trước khi opening rõ
- KHÔNG output recommendation generic — phải specific dựa trên Q&A
```

### Eval focus

1. **Golden**: User cooperative, trả lời chi tiết → skill ra recommendation specific
2. **Edge**: User trả lời ngắn / "không biết" → skill rephrase câu hỏi hoặc đưa option choices
3. **Anti**: User cố push qua Q&A nhanh → skill vẫn ensure đủ info trước khi synthesize, không cut corner

---

## Cross-type rules

Áp dụng cho MỌI dạng:

1. **Frontmatter description** luôn third-person, ≥5 trigger phrases, ≤1024 chars
2. **Body** luôn imperative form, ≤500 lines
3. **EVALS.md** bắt buộc 3 scenarios theo "Golden / Edge / Anti" pattern
4. **References 1-level** từ SKILL.md
5. **Forward slashes** trong path
6. **No emoji** (trừ khi user explicit yêu cầu)
