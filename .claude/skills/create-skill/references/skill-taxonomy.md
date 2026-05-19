# Skill Taxonomy — 2 trục phân loại

## Contents
- Trục 1: Reference vs Task (theo Anthropic dichotomy)
- Trục 2: 6 dạng Task skill
- Hybrid skills: chọn dạng PRIMARY
- Decision tree

## Trục 1 — Reference vs Task

Anthropic chia skill làm 2 loại lớn theo cách Claude SỬ DỤNG skill:

### Reference skill

**Đặc trưng**: Claude áp dụng kiến thức/style trong khi làm task KHÁC. Skill cung cấp context (schema, brand voice, policy, pattern), không tạo deliverable riêng.

**Trigger**: explicit như mọi skill — qua `/skill-name`, qua mention rõ ("dùng brand-guidelines khi viết landing page này"), hoặc Claude tự match description khi context conversation chạm domain. KHÔNG có chuyện "tự chạy ngầm" — mọi skill đều cần 1 trigger.

**Khác Task skill ở deliverable**: skill kết thúc, user KHÔNG nhận artifact mới riêng từ skill. Skill chỉ inject knowledge vào task chính (vd: landing page user đang build) để task chính ra output tốt hơn.

**Ví dụ thực tế**:
- `brand-guidelines` — voice/style brand. User trigger: "dùng brand-guidelines khi viết bài này" → Claude consult skill → áp dụng khi viết. Output user nhận = bài viết (do task chính), không phải file riêng từ skill.
- `bigquery-schema-reference` — table schemas. User trigger: "viết SQL để pull MRR" → Claude tự match description (vì context chạm BigQuery) → consult schema → output SQL. Schema file KHÔNG được copy ra cho user.
- `company-policies` — internal rules. User trigger: "review process onboarding của tôi có vi phạm policy không" → Claude consult policy → output đánh giá. Policy file vẫn ở skill folder.
- `api-reference` — endpoint docs. User trigger: "build integration với Stripe" → Claude consult endpoints → output integration code.

**Cấu trúc điển hình**:
```
brand-guidelines/
├── SKILL.md          # Overview + voice rules
└── references/
    ├── tone.md       # Tone per audience
    ├── words.md      # Banned/preferred words
    └── examples.md   # Before-after samples
```

**KHÔNG có**: pipeline numbered steps, output deliverable cụ thể, eval scenarios cho task hoàn chỉnh.

### Task skill

**Đặc trưng**: Claude chạy chuỗi bước CỤ THỂ và tạo ra DELIVERABLE RIÊNG (file mới, report, audit, code, revised draft). User nhận artifact độc lập từ skill.

**Trigger**: như Reference — `/skill-name [args]`, mention rõ, hoặc description match. Cách trigger giống nhau.

**Khác Reference**: skill kết thúc, user nhận artifact MỚI. Skill có pipeline numbered, có decision points, có output format spec.

**Ví dụ thực tế**:
- `macro-research` — User trigger: "/macro-research AI in finance" → skill ra 1 PDF mới (artifact riêng).
- `audit-webapp` — User trigger: "/audit-webapp" → skill ra 1 audit report (artifact riêng).
- `git-commit-viet` — User trigger: "commit this" → skill ra 1 commit message (artifact riêng).

**Cấu trúc điển hình**:
```
task-skill/
├── SKILL.md          # Pipeline N bước
├── EVALS.md          # 3 test scenarios
├── references/       # (optional) load when needed
├── assets/           # (optional) templates dùng trong output
└── scripts/          # (optional) deterministic code
```

### Phân biệt Reference vs Task — 1 câu hỏi quyết định

> "Sau khi skill chạy xong, user có nhận được artifact MỚI ĐỘC LẬP từ skill không?"

- **Có** (file mới, report mới, code mới, draft mới) → **Task skill**
- **Không**, output cuối user nhận là của task khác (skill chỉ inject knowledge để task khác ra output tốt hơn) → **Reference skill**

**Cùng trigger cách giống nhau** — đều qua `/skill-name`, mention rõ, hoặc description match. Khác chỉ ở deliverable.

## Trục 2 — 6 dạng Task skill

Áp dụng SAU KHI đã xác định là Task skill. Chọn 1 dạng PRIMARY.

### 1. Research

**Pattern**: input = topic → output = báo cáo có nguồn citation.

**Đặc trưng**:
- Spawn research agent (background) hoặc Web tool
- Tổng hợp nhiều nguồn
- Output có citation explicit
- Pipeline tương đối dài (1-3 giờ)

**Ví dụ**: macro-research (40+ trang), /research, vietnamese-seo-researcher

**Skill cần có**:
- Source priority list (ưu tiên loại nguồn nào)
- Citation format
- Output template (markdown/PDF/slide)
- Anti-pattern: "không bịa nguồn", "không suy đoán không có chứng cứ"

### 2. Workflow

**Pattern**: chuỗi bước có deliverable end-to-end, thường multi-stage.

**Đặc trưng**:
- 5-10 bước numbered
- Có decision points giữa chừng
- Phụ thuộc tool/file trong repo
- Output là 1 deliverable tổng hợp (audit report, deployment, migration)

**Ví dụ**: audit-webapp, /init, /security-review, deployment workflows

**Skill cần có**:
- Pipeline numbered, mỗi step có exit condition
- Decision points rõ ràng (when to ask user vs auto-proceed)
- Report template cuối pipeline
- Recovery instructions khi step fail

### 3. Generator

**Pattern**: tạo artifact mới (file, code, config) từ spec.

**Đặc trưng**:
- Input thường là spec/parameters
- Output là 1+ file mới
- Có template + placeholder fill logic
- Validation cuối cùng

**Ví dụ**: create-skill (chính nó), scaffold scripts, mcp-builder, artifacts-builder

**Skill cần có**:
- Input schema (params skill nhận)
- Template files trong `assets/`
- Naming convention cho output
- Self-validation (file parse OK, schema correct)

### 4. Analyzer

**Pattern**: input = artifact (URL, code, file) → output = insight có score/rating.

**Đặc trưng**:
- Multi-source data gathering
- Scoring rubric (0-100, pass/fail, severity tiers)
- Pre-delivery review (verify findings)
- Có thể có fallback cascade khi data thiếu

**Ví dụ**: seo-backlinks, /security-review, audit-webapp (overlap với Workflow), performance-profiling

**Skill cần có**:
- Source detection (data nào available)
- Scoring rubric explicit (thresholds)
- Fallback cascade (data thiếu thì degrade gracefully)
- "Insufficient data" gate (không output score giả)
- Pre-delivery review checklist

### 5. Voice/Style

**Pattern**: review hoặc rewrite content theo voice/style định trước.

**Đặc trưng**:
- Input = draft text
- Output = revised text + diff explanation
- Có before/after examples bắt buộc
- Anti-patterns explicit (cấm dùng từ X)

**Ví dụ**: voice-checker, git-commit-viet, brand-guidelines (overlap Reference)

**Skill cần có**:
- 2-5 cặp before/after examples (good vs bad)
- Anti-patterns list (banned words, banned structures)
- Tone parameters (audience, register)
- Diff format khi revise

### 6. Discovery / Interactive

**Pattern**: Q&A có hướng dẫn để KHÁM PHÁ requirement, không có deliverable cố định.

**Đặc trưng**:
- Conversational, multi-turn
- Mỗi turn skill quyết định câu hỏi tiếp theo
- Output cuối có thể là spec/plan/decision (không phải file)
- Khó eval — eval scenario thường là "quality of questions asked"

**Ví dụ**: brainstorming, /viet-strategist, requirements-gathering

**Skill cần có**:
- Question tree (nhánh câu hỏi theo answer)
- Stop condition (khi nào kết thúc Q&A)
- Output format cuối (summary/plan/recommendation)
- Tone guideline (Socratic vs directive)

## Hybrid skills — chọn PRIMARY

Nhiều skill thực tế là hybrid 2-3 dạng. Quy tắc:

1. **Chọn dạng deliver giá trị CHÍNH** làm primary.
2. Mention dạng phụ trong body skill để Claude hiểu nuance.

**Ví dụ hybrid**:

| Skill | Hybrid | Primary |
|---|---|---|
| `macro-research` | Research + Workflow | **Research** (nguồn + citation là core) |
| `audit-webapp` | Workflow + Analyzer | **Workflow** (multi-step audit là core, score là phụ) |
| `git-commit-viet` | Voice + Generator | **Voice** (style enforcement là core) |
| `seo-backlinks` | Analyzer + Reference | **Analyzer** (scoring là core) |
| `brand-guidelines` | Reference + Voice | **Reference** (Claude áp dụng nhiều domain) |

## Decision tree

```
Skill tạo ra artifact MỚI ĐỘC LẬP cho user không?
│
├── KHÔNG (chỉ inject knowledge vào task khác) → Reference skill
│
└── CÓ → Task skill — phân tiếp theo input/output:
    │
    ├── Input = topic/keyword → Research
    │
    ├── Input = spec/params → Generator
    │
    ├── Input = artifact có sẵn (URL/file/code) →
    │   │
    │   ├── Output rating/insight → Analyzer
    │   ├── Output revised version → Voice/Style
    │   └── Output multi-step deliverable → Workflow
    │
    └── Input = open-ended question, no fixed deliverable → Discovery
```

## Cách dùng taxonomy này

Trong Step 2 của pipeline create-skill:
1. Đọc answer của user ở Step 1 (NHẬN gì → TRẢ gì + 2 ví dụ)
2. Dùng decision tree để classify
3. Nếu hybrid, chọn primary
4. Pass classification → Step 3 (đọc `type-guides.md` mục tương ứng)
