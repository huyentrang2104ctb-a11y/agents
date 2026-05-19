# Anti-patterns — 12 lỗi phổ biến + cách fix

Tổng hợp từ Anthropic best practices doc + skill repos thực tế (anthropics/skills, claude-seo, awesome-claude-skills). Đọc khi user hỏi "tại sao skill X không trigger / không hoạt động đúng" hoặc khi skill mới fail validation.

## Contents
- 1. Description quá vague
- 2. Second person trong body
- 3. Nhồi mọi thứ vào SKILL.md
- 4. Nested references
- 5. Time-sensitive info
- 6. Windows-style paths
- 7. Mix scope (project vs user) confusion
- 8. Skipping eval scenarios
- 9. Reserved word trong name
- 10. Description >1024 chars
- 11. Offering too many options
- 12. "Voodoo constants" trong scripts

---

## 1. Description quá vague

**Why bad**: Anthropic warning rõ — *"Claude has a tendency to UNDERTRIGGER skills"*. Description vague → Claude không nhận ra cần dùng skill → skill bị bỏ qua dù task hợp.

❌ **Bad**:
```yaml
description: Helps with PDFs.
```
```yaml
description: A skill for working with documents.
```
```yaml
description: Use this skill when you need it.
```

✅ **Good**:
```yaml
description: This skill should be used when the user asks to "extract text from PDF", "merge PDFs", "fill PDF form", "rotate PDF pages", "phân tích PDF", or works with .pdf files. Extracts text/tables, fills forms, merges documents using pdfplumber and pypdf.
```

**Rule**:
- Third-person form ("This skill should be used when...")
- Liệt kê ≥5 trigger phrases CỤ THỂ (cả VN + EN)
- Bao gồm cả *what* skill làm + *when* nên trigger

---

## 2. Second person trong body

**Why bad**: Anthropic style guide — body skill viết cho Claude (instance khác), không phải cho user. Imperative form ngắn hơn, ít ambiguous, consistent.

❌ **Bad**:
```markdown
## Step 1
You should start by reading the configuration file. Then you can validate the input. You might want to use grep to search for patterns.
```

✅ **Good**:
```markdown
## Step 1
Read the configuration file. Validate the input. Use grep to search for patterns.
```

❌ **Bad (Vietnamese)**:
```markdown
Bạn nên hỏi user về scope trước. Sau đó bạn có thể tạo file template. Nếu bạn muốn rebrand, hãy chỉnh CSS variables.
```

✅ **Good (Vietnamese)**:
```markdown
Hỏi user về scope. Tạo file template. Để rebrand, chỉnh CSS variables.
```

---

## 3. Nhồi mọi thứ vào SKILL.md

**Why bad**: SKILL.md load mỗi lần skill trigger. Body 5000+ từ → context bloat, đẩy useful info ra khỏi window.

❌ **Bad**:
```
my-skill/
└── SKILL.md  (8,000 từ — pipeline + advanced patterns + edge cases + API ref + examples)
```

✅ **Good** — apply progressive disclosure 3 tầng:
```
my-skill/
├── SKILL.md  (1,800 từ — overview + pipeline core)
├── references/
│   ├── advanced-patterns.md   (3,500 từ — load khi cần)
│   ├── edge-cases.md          (2,000 từ — load khi gặp edge)
│   └── api-reference.md       (4,000 từ — load khi cần API detail)
└── examples/
    └── working-example.sh
```

**Rule**: SKILL.md ≤500 lines / 1500-2000 từ. Mọi detail dài tách `references/`.

---

## 4. Nested references

**Why bad**: Anthropic rõ — Claude `head -100` partial-read khi follow nested chain. Nested → bỏ sót info.

❌ **Bad**:
```markdown
# SKILL.md
For details, see [advanced.md](references/advanced.md)

# references/advanced.md
For more, see [details.md](references/details.md)

# references/details.md
[Actual content here]
```

✅ **Good** — flat 1-level:
```markdown
# SKILL.md
- **Basic**: [content trong SKILL.md]
- **Advanced patterns**: See [advanced.md](references/advanced.md)
- **Detailed examples**: See [examples.md](references/examples.md)
- **API reference**: See [api-ref.md](references/api-ref.md)
```

**Rule**: Mọi reference link 1 level từ SKILL.md. Reference file KHÔNG link tiếp tới reference khác.

---

## 5. Time-sensitive info

**Why bad**: Sẽ outdated nhanh. Skill build hôm nay với "Sau Q2 2026..." → 6 tháng sau wrong.

❌ **Bad**:
```markdown
If you're doing this before August 2026, use the v1 API.
After August 2026, use the v2 API.
```

✅ **Good** — wrap historical context:
```markdown
## Current method
Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2026-08)</summary>

The v1 API used: `api.example.com/v1/messages`. Endpoint no longer supported.
</details>
```

**Rule**: Time-sensitive info → "Old patterns" collapsed section, hoặc remove.

---

## 6. Windows-style paths

**Why bad**: Backslash `\` gây lỗi trên Unix systems. Forward slash `/` cross-platform OK.

❌ **Bad**:
```markdown
Run `scripts\helper.py` to extract data.
See `references\guide.md` for details.
```

✅ **Good**:
```markdown
Run `scripts/helper.py` to extract data.
See `references/guide.md` for details.
```

**Rule**: Forward slash kể cả khi skill target Windows. Anthropic best practice cứng.

**Exception**: PowerShell-specific commands trong `scripts/*.ps1` có thể dùng backslash native cho Windows path. Nhưng path TRONG SKILL.md body luôn forward slash.

---

## 7. Mix scope (project vs user) confusion

**Why bad**: User nghĩ skill chạy mọi project nhưng thực ra chỉ project hiện tại (hoặc ngược lại). Bug khó debug.

❌ **Bad**: Skill installed ở `.claude/skills/` (project scope) nhưng SKILL.md ghi "available everywhere".

✅ **Good** — explicit về scope:
```markdown
## Scope
This skill is **project-scoped** — only available when Claude Code runs in this repo.

To make it user-scoped (available everywhere), copy folder to `~/.claude/skills/`.
```

**Rule khi create-skill**: Hỏi user scope ở Step 1, default Project. Mention rõ trong report cuối.

---

## 8. Skipping eval scenarios

**Why bad**: Skill build xong không có cách test → drift over time. User update SKILL.md sau 1 tháng không biết liệu vẫn hoạt động đúng không.

❌ **Bad**:
```
my-skill/
└── SKILL.md  (no EVALS.md)
```

✅ **Good**:
```
my-skill/
├── SKILL.md
└── EVALS.md  (3 scenarios: Golden / Edge / Anti)
```

**Rule**: 3 eval scenarios MINIMUM. Pattern Golden / Edge / Anti:
- **Golden**: Happy path — input chuẩn, output chuẩn
- **Edge**: Ambiguous/incomplete input — skill phải graceful
- **Anti**: Input mà skill nên TỪ CHỐI hoặc clarify, không cứ làm

---

## 9. Reserved word trong name

**Why bad**: Anthropic constraint cứng — skill name không chứa "anthropic" hoặc "claude". Skill sẽ reject ở registration.

❌ **Bad**:
```yaml
name: claude-helper
name: anthropic-tool
name: my-claude-skill
```

✅ **Good**:
```yaml
name: code-reviewer
name: research-assistant
name: pdf-processor
```

**Rule**: Validate cứng ở Step 4. Reject + đề xuất tên mới nếu user input chứa reserved word.

---

## 10. Description >1024 chars

**Why bad**: Anthropic limit cứng — description >1024 chars sẽ truncate hoặc reject.

❌ **Bad**:
Description với 30+ trigger phrases + paragraph dài về philosophy + danh sách features chi tiết → 1500+ chars.

✅ **Good**:
Description tập trung — what + when + 5-8 trigger phrases CỐT LÕI nhất. Detail dài → SKILL.md body, không phải description.

**Rule**: Validate length ở Step 5. Trim trigger phrases ít quan trọng nếu vượt limit.

---

## 11. Offering too many options

**Why bad**: Anthropic best practice — "Don't present multiple approaches unless necessary". User confused, Claude indecisive.

❌ **Bad**:
```markdown
For PDF extraction, you can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or pdfminer, or...
```

✅ **Good** — provide default + escape hatch:
```markdown
Use `pdfplumber` for text extraction:

```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For scanned PDFs requiring OCR, use `pdf2image` with `pytesseract` instead.
```

**Rule**: Mỗi decision point → 1 default + 1 alternative cho edge case. Không liệt kê 5 options.

---

## 12. "Voodoo constants" trong scripts

**Why bad**: Anthropic best practice (Ousterhout's law) — config values không có rationale = magic numbers. Claude không biết khi nào điều chỉnh.

❌ **Bad**:
```python
TIMEOUT = 47  # Why 47?
MAX_RETRIES = 5  # Why 5?
BUFFER_SIZE = 8192  # Why 8192?
```

✅ **Good**:
```python
# HTTP requests typically complete within 30s
# Longer timeout accounts for slow upstream APIs
TIMEOUT = 30

# Three retries balances reliability vs latency
# Most intermittent failures resolve by retry 2
MAX_RETRIES = 3

# 8KB buffer matches default OS page size for efficient I/O
BUFFER_SIZE = 8192
```

**Rule**: Mỗi config value có comment 1 dòng giải thích WHY value đó.

---

## Sample anti-pattern check (cứng)

Trước khi commit skill mới, sample 5-10 sentences từ SKILL.md body. Check:

| Check | Pattern bị flag | Fix |
|---|---|---|
| Second person | "bạn nên", "you should", "you can", "hãy bạn" | → imperative form |
| Vague description | "helps with", "for working with", "provides" | → specific trigger phrases |
| Time-sensitive | "trong 2026", "after Q2", "as of <date>" | → "Old patterns" section |
| Windows path | `\\` trong markdown body | → forward slash |
| Hype | "đột phá", "revolutionary", "ngoạn mục", "game-changing" | → direct statement |
| Emoji | 🎯 ✨ 🚀 trong headings | → remove |
| Hedge | "có thể", "khả năng", "perhaps", "might be" | → direct statement với data |

Sample-check fail >1 violation → re-write section, re-check.

---

## When to consult this file

Đọc anti-patterns khi:
- Validation gate fail (Step 8 của pipeline)
- User report skill không trigger / behavior wrong
- Claude tự debug skill mới generate

KHÔNG cần load anti-patterns mỗi lần create-skill chạy. Chỉ load khi gặp issue cần fix.
