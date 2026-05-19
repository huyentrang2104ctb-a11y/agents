# Validation Checklist

Run mọi item trước khi commit skill mới. Hard gates BẮT BUỘC pass 100%. Content gates SHOULD pass — nếu fail, justify trong report.

## Contents

- [Hard gates (auto-verifiable)](#hard-gates-auto-verifiable)
  - [Frontmatter](#frontmatter)
  - [Description quality](#description-quality)
  - [Body length](#body-length)
  - [Body structure](#body-structure)
  - [Bundled resources](#bundled-resources-nếu-có)
  - [Eval scenarios](#eval-scenarios)
- [Content gates (manual review)](#content-gates-manual-review)
  - [Khi nào dùng](#khi-nào-dùng-section)
  - [Default settings](#default-settings)
  - [Pipeline](#pipeline-cho-task-skills)
  - [Anti-patterns section](#anti-patterns-section)
  - [Voice](#voice)
- [Type-specific gates](#type-specific-gates)
- [Anti-pattern sample-check](#anti-pattern-checks-cứng--sample-check-trước-commit)
- [Failure recovery](#failure-recovery)
- [Final validation report format](#final-validation-report-format)

## Hard gates (auto-verifiable)

### Frontmatter
- [ ] File `SKILL.md` exists ở `.claude/skills/<name>/` hoặc `~/.claude/skills/<name>/`
- [ ] Frontmatter parse OK (YAML hợp lệ — không indentation lỗi, không stray quote)
- [ ] Field `name` exists, lowercase + hyphens + numbers ONLY
- [ ] Field `name` ≤64 ký tự
- [ ] Field `name` KHÔNG chứa "anthropic" hoặc "claude" (reserved words)
- [ ] Field `name` KHÔNG chứa XML tags
- [ ] Field `name` MATCH tên folder skill
- [ ] Field `description` exists, non-empty
- [ ] Field `description` ≤1024 ký tự
- [ ] Field `description` KHÔNG chứa XML tags

### Description quality
- [ ] Third-person form ("This skill should be used when...") — KHÔNG "I can help" / "Use this when..."
- [ ] Có ≥5 trigger phrases CỤ THỂ (không vague)
- [ ] Bao gồm cả *what* (skill làm gì) + *when* (trigger conditions)
- [ ] (Default) Có cả VN + EN trigger phrases

### Body length
- [ ] Body ≤500 dòng
- [ ] Body 1500-2000 từ ideal (cảnh báo nếu <800 hoặc >3000)

### Body structure
- [ ] Có ≥1 H2 heading "Khi nào dùng" (hoặc tương đương)
- [ ] Có ≥1 H2 heading về pipeline/quy trình (trừ Reference skill)
- [ ] Mọi reference link đi tới file 1-level từ SKILL.md (không nested)
- [ ] Mọi path dùng forward slash (`/`), KHÔNG backslash (`\`)

### Bundled resources (nếu có)
- [ ] Mọi file referenced trong SKILL.md TỒN TẠI thật ở đúng path
- [ ] `references/*.md` có TOC nếu file >100 dòng
- [ ] `scripts/*` executable (chmod +x cho .sh, .py)
- [ ] `assets/*` có header comment giải thích purpose

### Eval scenarios
- [ ] `EVALS.md` exists ở root skill folder
- [ ] Có ĐÚNG 3 scenarios (Golden / Edge / Anti)
- [ ] Mỗi scenario có: User input + Expected behavior + Pass criteria

## Content gates (manual review)

### Khi nào dùng section
- [ ] Liệt kê CỤ THỂ patterns user nói
- [ ] Có "KHÔNG dùng skill này khi..." section (negative trigger)

### Default settings
- [ ] Có table với cột "Setting / Default / Override khi"
- [ ] Mọi default đều có rationale ngầm hoặc explicit

### Pipeline (cho Task skills)
- [ ] Mỗi step có name rõ ràng
- [ ] Mỗi step ≤200 từ
- [ ] Imperative form ("Tạo X", "Hỏi user Y") — KHÔNG "Bạn nên" / "You should"
- [ ] Decision points explicit (khi nào hỏi user vs auto)
- [ ] Recovery actions cho failure modes

### Anti-patterns section
- [ ] Có ≥3 anti-patterns explicit
- [ ] Mỗi anti-pattern có example BAD (không chỉ rule trừu tượng)

### Voice
- [ ] No emoji (trừ user explicit cho phép)
- [ ] No hype words ("revolutionary", "đột phá", "cách mạng", "ngoạn mục")
- [ ] No hedge phrases ("có thể", "khả năng cao") — replace bằng direct statement
- [ ] Consistent terminology (chọn 1 từ + dùng xuyên suốt — vd: "skill" hoặc "kỹ năng", không mix)

## Type-specific gates

### Reference skill
- [ ] KHÔNG có pipeline numbered steps (Reference skill không có procedure)
- [ ] Có "Khi nào áp dụng" rõ — domain context conversation nào trigger
- [ ] Có ≥1 reference file trong `references/` (knowledge tách ra)

### Research skill
- [ ] Có "Source priority" section — liệt kê loại nguồn ưu tiên
- [ ] Có citation format spec
- [ ] Có anti-pattern "KHÔNG bịa số liệu"

### Workflow skill
- [ ] Có "Tiền điều kiện" checklist
- [ ] Có "Decision points" table
- [ ] Có "Recovery" section per failure mode

### Generator skill
- [ ] Có "Input schema" table với required/optional/default
- [ ] Có "Output naming convention" rõ
- [ ] Có self-validation step ở cuối pipeline

### Analyzer skill
- [ ] Có "Source detection" bước đầu
- [ ] Có "Scoring rubric" với weight + threshold
- [ ] Có "Data sufficiency gate" (không output score giả khi data thiếu)
- [ ] Có "Pre-delivery review" mandatory checklist
- [ ] Có "Fallback cascade" (degrade gracefully)

### Voice/Style skill
- [ ] Có ≥2 cặp BEFORE-AFTER examples
- [ ] Có "Banned words" list
- [ ] Có "Banned structures" list
- [ ] Có output format spec khi revise

### Discovery skill
- [ ] Có "Question tree" với opening + branching
- [ ] Có "Stop condition" explicit
- [ ] Có output cuối format spec

## Anti-pattern checks (cứng — sample-check trước commit)

Sample 5-10 sentences từ body và check:

### Second person violations
❌ Pattern: "Bạn nên...", "You should...", "You can use...", "Hãy bạn..."
✅ Fix: "Tạo X", "Use Y", "Add Z"

### Vague trigger violations (description)
❌ Pattern: "Helps with X", "Provides Y", "Makes Z easier", "For when working with..."
✅ Fix: 'This skill should be used when the user asks to "<specific phrase 1>", "<specific phrase 2>"...'

### Time-sensitive violations
❌ Pattern: "Trong 2026...", "Sau Q2 2026...", "As of May 2026..."
✅ Fix: Wrap trong `## Old patterns` section nếu cần historical context, hoặc remove

### Path violations
❌ Pattern: `scripts\helper.py`, `references\guide.md`
✅ Fix: `scripts/helper.py`, `references/guide.md`

### Hype violations
❌ Pattern: "đột phá", "cách mạng", "revolutionary", "game-changing", "ngoạn mục"
✅ Fix: Direct statement — "produces X output", "achieves Y in Z minutes"

### Emoji violations
❌ Pattern: 🎯 ✨ 🚀 ⚡ trong headings hoặc body
✅ Fix: Remove (trừ user explicit cho phép)

## Failure recovery

Nếu hard gate fail, action cụ thể:

| Gate fail | Recovery |
|---|---|
| Frontmatter parse lỗi | Re-read frontmatter, fix YAML syntax (indentation, quotes, colons) |
| Name >64 chars hoặc invalid chars | Đề xuất 3 candidate gerund-form ngắn hơn, ask user choose |
| Description >1024 chars | Trim trigger phrases ít quan trọng, giữ ≥5 phrases cụ thể nhất |
| Body >500 lines | Tách section dài nhất ra `references/<topic>.md`, replace bằng pointer 1 dòng |
| Reference nested >1 level | Move target file từ subfolder lên root references/, update link |
| EVALS.md missing | Generate 3 scenarios từ template `assets/eval-template.md` |
| File reference broken | Tạo file stub hoặc remove reference từ SKILL.md |

Nếu content gate fail (không bắt buộc pass):
- Note trong report cuối: "Content gate X failed — reason: Y. Skill vẫn ship được nhưng nên revisit."

## Final validation report format

```
Validation report — <skill-name>

Hard gates: <N>/<total> passed
- PASS: <gate 1>
- PASS: <gate 2>
- FAIL: <gate 3> — <reason> — <recovery action taken>

Content gates: <N>/<total> passed (informational)
- PASS: ...
- WARN: <gate X> — <reason> — <action: ship as-is / revisit later>

Type-specific gates (<type>): <N>/<total> passed
- PASS: ...

Anti-pattern sample-check: <N>/10 sentences clean
- 1 violation: <type> at line <N> — <fix applied>

Status: READY TO SHIP | NEEDS FIX
```
