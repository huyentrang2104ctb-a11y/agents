---
name: create-skill
description: This skill should be used when the user asks to "create a new skill", "tạo skill mới", "build a Claude Code skill", "đóng gói thành skill", "/create-skill [name]", "skill mới cho [task]", or wants to package a recurring workflow into a reusable Claude Code skill at .claude/skills/. Generates production-grade SKILL.md following Anthropic's official best practices — progressive disclosure (3 tiers), third-person pushy description, imperative form body (≤500 lines), gerund-form naming, scripts/references/assets separation. Classifies skills along 2 axes (Reference vs Task by deliverable; if Task, choose 1 of 6 subtypes: Research/Workflow/Generator/Analyzer/Voice/Discovery), generates type-specific templates, and ships 3 eval scenarios (Golden/Edge/Anti) for testing.
---

# create-skill

Tạo Claude Code skill mới đạt chuẩn production. Skill được generate sẽ tuân thủ best practices chính thức của Anthropic: progressive disclosure 3 tầng, description trigger mạnh, body imperative ≤500 dòng, naming gerund form, tách bạch `scripts/`/`references/`/`assets/`.

## Khi nào dùng skill này

User nói một trong các pattern:
- `/create-skill [name]` hoặc `/create-skill` (không argument)
- "tạo skill mới cho [task]"
- "đóng gói quy trình này thành skill"
- "build a Claude Code skill that..."
- "tôi muốn skill làm X mỗi lần tôi gõ /Y"

KHÔNG dùng skill này khi:
- User chỉ muốn 1 prompt 1 lần (prompt thẳng nhanh hơn)
- User muốn sửa skill có sẵn (đọc skill đó + edit, không generate mới)
- User muốn cài skill từ repo có sẵn (clone + copy, không generate)

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Scope | Project (`.claude/skills/<name>/`) | User nói "skill toàn cục" → `~/.claude/skills/<name>/` |
| Ngôn ngữ body | Tiếng Việt | User chỉ nói tiếng Anh trong yêu cầu |
| Description | Mixed VN/EN trigger phrases | User chỉ định 1 ngôn ngữ |
| Name format | gerund (`processing-pdfs`, `analyzing-spreadsheets`) | Skill quá đặc trưng tới mức gerund unnatural |
| Body length | 1500-2000 từ, ≤500 dòng | Skill phức tạp → tách `references/` |

## Pipeline — 8 bước

Theo thứ tự, không skip. Sau mỗi bước trừ Step 1, KHÔNG hỏi user xác nhận — tự quyết theo default rồi report cuối cùng.

### Step 1 — Hỏi 2 concrete examples (HARD GATE)

**Step 1 là HARD GATE, không phải clarifying question optional.** Kể cả khi session có "no-clarify directive" (system reminder yêu cầu không hỏi user), vẫn fail-closed nếu input không pass 3 checks. Mục đích skill là tạo SKILL.md chất lượng — không tạo junk skill từ input flawed.

Hỏi đúng 1 message duy nhất chứa 2 câu:

```
1. Skill này NHẬN gì → TRẢ gì? (1 câu)
2. Cho 2 ví dụ thật về cách user sẽ gọi skill này.
```

Lý do: Anthropic best practice — "Before writing any code, identify 2-3 concrete use cases your skill should enable" và "concrete examples làm rõ functionality hơn description trừu tượng".

**Skip rules** — chỉ skip nếu user input ban đầu PASS cả 3 check:
- [ ] Có description cụ thể (KHÔNG vague kiểu "helps with X", "for working with Y")
- [ ] Có ≥1 ví dụ user sẽ gọi skill thế nào *(với Reference skill, "context condition" satisfy check này — vd: "khi user viết content trong project")*
- [ ] Có description rõ về NHẬN gì → TRẢ gì *(với Reference skill, output có thể là "KNOWLEDGE inject vào task khác", không cần file riêng)*

Nếu fail bất kỳ check nào → hỏi đúng 2 câu trên, KHÔNG cứ proceed với input flawed.

### Step 2 — Classify skill type

Đọc `references/skill-taxonomy.md` để hiểu 2 trục phân loại. Tự classify theo answer của user, không hỏi lại:

**Trục 1 (chính)**: Reference vs Task — phân biệt qua DELIVERABLE, không qua trigger
- **Reference skill**: Claude áp dụng kiến thức/style trong khi làm task KHÁC. User KHÔNG nhận artifact mới độc lập từ skill. Vd: brand-guidelines, schema docs.
- **Task skill**: Claude chạy bước cụ thể tạo ra DELIVERABLE MỚI ĐỘC LẬP (file, report, code, draft). User nhận artifact riêng từ skill. Vd: macro-research, audit-webapp.
- Cả 2 cùng cách trigger (`/skill-name`, mention rõ, hoặc description match). Khác chỉ ở deliverable.

**Trục 2 (chỉ áp dụng nếu Task)**: chọn 1/6 dạng
- **Research** — input topic → output báo cáo có nguồn (vd: macro-research)
- **Workflow** — chuỗi bước có deliverable end-to-end (vd: audit-webapp)
- **Generator** — tạo artifact: file/code/config (vd: create-skill chính nó, scaffold)
- **Analyzer** — input artifact → output insight có score (vd: seo-backlinks, security-review)
- **Voice/Style** — review/rewrite theo voice định trước (vd: voice-checker, git-commit-viet)
- **Discovery** — interactive Q&A → output plan/recommendation/decision matrix (vd: brainstorming, requirements-gathering)

Edge case: nếu hybrid (vd: macro-research = Research + Workflow), chọn dạng PRIMARY (cái deliver giá trị chính), apply template của dạng đó, mention dạng phụ trong body.

### Step 3 — Hỏi câu chuyên biệt theo dạng

Đọc `references/type-guides.md` mục tương ứng với dạng đã classify. Hỏi 2-4 câu chuyên biệt cho dạng đó. Ví dụ:

- Research → "Nguồn ưu tiên?", "Output format (markdown/PDF/slides)?", "Vietnam context bắt buộc không?"
- Workflow → "Bước cuối deliver gì?", "Skill phụ thuộc tool/file gì trong repo?", "Khi nào skill nên tự dừng để hỏi user?"
- Voice → "Cho 2 ví dụ before-after (1 bad → 1 good)", "Anti-patterns cần loại?"

Đặt TẤT CẢ câu hỏi trong 1 message duy nhất. Không hỏi tuần tự nhiều round.

### Step 4 — Đề xuất name + validate

Đề xuất 2-3 candidate names cho skill, ưu tiên gerund form:
- `processing-pdfs` (KHÔNG `pdf-tool` hay `pdf-helper`)
- `analyzing-spreadsheets` (KHÔNG `excel-skill`)
- `reviewing-code` (KHÔNG `code-review`)

**Reference skill exception**: skills naming một domain entity (brand, schema, policy, voice) thường dùng noun-phrase thay vì gerund — vd: `brand-guidelines`, `bigquery-schema-reference`, `company-policies`, `seongon-brand-voice`. Acceptable khi noun-phrase mô tả domain rõ hơn gerund equivalent (vd: `applying-brand-guidelines` clunkier than `brand-guidelines`).

Validate cứng (skill name BẮT BUỘC pass):
- Lowercase letters + numbers + hyphens ONLY
- ≤64 ký tự
- KHÔNG chứa "anthropic" hoặc "claude" (reserved words)
- KHÔNG chứa XML tags
- Match tên folder skill

**Khi user-supplied name FAIL validation** (vd: user input `claude-helper-tool`):
1. State EXPLICITLY gate fail nào (vd: "Reserved word 'claude' detected")
2. Propose 2-3 alternatives gerund-form (hoặc noun-phrase nếu Reference) với 1 dòng rationale per candidate
3. Default pick most descriptive, proceed unless user redirect
4. Re-validate name mới

User chọn 1 trong các candidate hoặc đề xuất tên mới → validate lại.

### Step 5 — Generate frontmatter

Frontmatter chuẩn:

```yaml
---
name: <validated-name>
description: This skill should be used when the user [trigger phrases cụ thể bằng VN+EN, ≥5 phrases]. [What it does — 1 câu]. [When to use — 1 câu].
---
```

Rules cứng:
- Description **third-person**: "This skill should be used when..." (KHÔNG: "Use this when..." / "I can help you...")
- Description **pushy** — Anthropic cảnh báo Claude tendency UNDERTRIGGER skills. Dùng "should be used", liệt kê ≥5 triggers cụ thể.
- Description ≤1024 ký tự
- Triggers bao gồm CẢ tiếng Việt và tiếng Anh (default), trừ khi user override
- Bao gồm cả *what* (skill làm gì) + *when* (khi nào trigger)

**Trigger format khác theo dạng**:
- **Task skill**: triggers là explicit phrases user sẽ gõ ("create a hook", "/macro-research", "tạo skill mới") — vì user phải explicitly invoke để nhận deliverable.
- **Reference skill**: triggers MIX 2 loại:
  - Explicit phrases ("/skill-name", "dùng brand-voice khi viết bài này")
  - Context conditions ("when the user writes content in this project", "when user works in BigQuery context") — để Claude match implicit khi context conversation chạm domain
- Cả 2 dạng đều có thể trigger qua mention rõ. Description format chỉ khác ở việc Reference skill thêm context conditions.

Anti-pattern (bị reject ngay):
- ❌ "Use this skill when working with X" (vague + second person)
- ❌ "Helps with documents" (vague)
- ❌ "I can help you process Excel files" (first person)

### Step 6 — Generate body

1. **Đọc** `assets/skill-template.md` — base skeleton 6 phần.
2. **Đọc** `references/type-guides.md` mục tương ứng với dạng đã classify — per-type refinement.
3. **Apply refinement lên skeleton** → generate body.

**Cấu trúc khác nhau theo dạng**:

| Section trong skeleton | Task skill | Reference skill |
|---|---|---|
| Khi nào dùng | Giữ | Đổi thành "Khi nào áp dụng" + context conditions |
| Default settings | Giữ | Optional — DEFAULT bỏ, chỉ include nếu skill thật sự có settings |
| **Pipeline N bước** | **BẮT BUỘC giữ** | **BẮT BUỘC bỏ — thay bằng "Core principles" + "Reference files"** |
| Tiêu chí chất lượng | Giữ | Optional — DEFAULT bỏ, chỉ include khi có quality criteria đặc biệt |
| Anti-patterns | Giữ | Giữ |

**Rule cho "Optional" sections**: DEFAULT là OMIT cho gọn. Chỉ include khi skill thật sự cần (vd: Reference skill có configuration knobs, Discovery skill có specific quality criteria). Better lean than padded.

Reference skill KHÔNG có pipeline numbered — đó là dấu hiệu phân biệt với Task. Đừng force pipeline vào Reference skill cho có.

**Writing style RULES (cứng — sample-check trước commit)**:
- Imperative form: "Tạo X", "Hỏi user Y", "Validate Z" (KHÔNG: "Bạn nên tạo X", "You should...")
- Body ≤500 dòng / 1500-2000 từ
- Forward slashes trong path kể cả khi target Windows (Anthropic best practice)
- Consistent terminology — chọn 1 từ và dùng xuyên suốt
- KHÔNG time-sensitive info — wrap trong "Old patterns" section nếu cần
- Mọi link `references/*.md` 1 level từ SKILL.md (không nested)

Anti-pattern detail đọc `references/anti-patterns.md` nếu cần.

### Step 7 — Generate bundled resources

**Auto-suggest theo dạng** (KHÔNG hỏi user nếu auto-suggest rõ ràng):

| Dạng | references/ | scripts/ | assets/ |
|---|---|---|---|
| Reference | **Luôn** (knowledge tách ra) | Hiếm | Hiếm |
| Research | Optional (source priority) | Optional (data fetch) | **Luôn** (output templates) |
| Workflow | Optional (advanced patterns) | Optional (validation) | Optional (report templates) |
| Generator | Optional (advanced patterns) | Optional (validation) | **Luôn** (output templates) |
| Analyzer | **Luôn** (scoring rubric detail) | **Luôn** (data fetch + scoring) | Optional (report templates) |
| Voice | **Luôn** (examples + banned lists) | Hiếm | Optional (sample-before/after files) |
| Discovery | Optional (question trees) | Hiếm | Optional (output templates) |

**Auto-create resources mark "Luôn"** cho dạng đã classify. Hỏi user chỉ khi:
- Resource mark "Optional" — hỏi 1 câu: "Skill có cần [resource type] không? (vd: [concrete example])"
- Edge case không match table — hỏi rõ

**No-clarify fallback** (khi session có no-clarify directive):
- Resources mark "Luôn" — auto-create như bình thường
- Resources mark "Optional" — auto-include nếu skill body mention resource cụ thể (vd: "see `scripts/validate.sh`"). Skip nếu skill body không mention.
- Resources mark "Hiếm" — skip mặc định, không hỏi.

**3 thư mục KHÔNG gộp lại** — mỗi mục đích khác:
- `scripts/` — execute, không vào context
- `references/` — Claude `Read` khi cần
- `assets/` — copy/modify ra output

Mỗi resource generate stub với 1 comment đầu file giải thích purpose.

Nếu `references/*.md` >100 dòng, BẮT BUỘC thêm Table of Contents ở đầu file.

### Step 8 — Generate eval scenarios + self-validate

Đọc `assets/eval-template.md`. Generate **3 eval scenarios** cho skill mới — user dùng để test skill có hoạt động đúng không.

Format mỗi scenario:
```markdown
## Eval N: <tên scenario>

**User input**: <prompt user sẽ gõ>
**Expected behavior**:
- <hành vi 1 skill cần làm đúng>
- <hành vi 2>
- <hành vi 3>
**Pass criteria**: <khi nào coi là pass>
```

3 scenario nên cover: (1) golden path, (2) edge case / ambiguous input, (3) anti-pattern (input mà skill nên TỪ CHỐI hoặc clarify).

Lưu eval vào `<skill-folder>/EVALS.md`.

Sau đó self-validate theo `references/validation-checklist.md`. Hard gates phải pass 100%:
- [ ] Frontmatter parse OK (YAML hợp lệ)
- [ ] Name match folder, lowercase+hyphens, ≤64 chars, không reserved word
- [ ] Description ≤1024 chars, third-person, ≥5 trigger phrases
- [ ] Body ≤500 dòng
- [ ] Body imperative form (sample-check 5 sentences đầu)
- [ ] References 1-level deep
- [ ] Forward slashes in paths
- [ ] EVALS.md có 3 scenario

Nếu fail bất kỳ gate nào → fix → re-validate → mới report.

## Output report

Sau Step 8 thành công, output cho user:

```
Skill created — <path-tới-folder>

Files:
- SKILL.md (X words, Y lines)
- EVALS.md (3 scenarios)
- references/<files> (nếu có)
- assets/<files> (nếu có)
- scripts/<files> (nếu có)

Type: <Reference | Task: Research/Workflow/...>
Scope: <Project | User>

Cách invoke:
  /<skill-name> [argument]

Test skill:
  Mở phiên Claude Code mới, chạy 3 scenario trong EVALS.md.

Lưu ý: Restart Claude Code để skill xuất hiện trong /skills list.
```

KHÔNG output emoji (kể cả ✓ ✗ status markers — dùng "OK" / "FAIL" plain text), KHÔNG hype, KHÔNG "Hope this helps" / "Let me know if you need..." trailing.

## Voice rules (cho skill được generate)

Inherit từ Viet's voice rules ở `/personal/CLAUDE.md`:
- Direct, không hedge
- KHÔNG emoji
- KHÔNG hype words ("đột phá", "cách mạng", "revolutionary")
- Imperative > "you should"
- Frame user as leader/strategist (đặc biệt khi target user là Viet)

Nếu user yêu cầu skill voice khác (vd: cho khoá đào tạo SEONGON cần friendly hơn), override theo yêu cầu cụ thể, default vẫn là Viet's voice.

## Common pitfalls — 4 cái critical nhất

1. **Description quá vague** — "Helps with X" → Claude UNDERTRIGGER skill. Phải có ≥5 trigger phrases CỤ THỂ + third-person form.
2. **Nhồi mọi thứ vào SKILL.md** — body 5000+ từ context bloat. Cứng: ≤500 dòng / 1500-2000 từ. Detail dài → `references/`.
3. **Second person trong body** — "Bạn nên...", "You should...". Sửa: imperative ("Tạo...", "Hỏi user...").
4. **Quên EVALS** — skill không có 3 scenarios test → drift over time. Minimum: Golden/Edge/Anti.

Đọc `references/anti-patterns.md` cho 12 anti-patterns đầy đủ + good/bad examples cho mỗi cái.

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `SKILL.md` | Main pipeline (file này) | Khi skill triggers |
| `references/skill-taxonomy.md` | Reference vs Task + 6 task types | Step 2 |
| `references/type-guides.md` | Q&A scripts + section structure per type | Step 3, Step 6 |
| `references/validation-checklist.md` | Hard gates + content gates | Step 8 |
| `references/anti-patterns.md` | 10+ common mistakes với good/bad examples | Khi user hỏi "tại sao skill X fail" |
| `assets/skill-template.md` | Base SKILL.md skeleton — copy → fill | Step 6 |
| `assets/eval-template.md` | Template 3 eval scenarios | Step 8 |

## Sample timing

- Step 1 hỏi examples: 1 min (nếu user trả lời ngay)
- Step 2 classify: 30 sec
- Step 3 hỏi type-specific: 2-3 min
- Step 4-5 name + frontmatter: 2 min
- Step 6 body: 5-10 min
- Step 7 bundled resources: 3-5 min
- Step 8 eval + validate: 2-3 min

Total: **15-25 phút** từ idea → ship-ready skill có eval.
