---
name: analyzing-seo-contracts
description: This skill should be used when the user says "/analyzing-seo-contracts [link]", "phân tích hợp đồng SEO này", "review hợp đồng dịch vụ SEO", "đọc hợp đồng và tóm tắt KPI + thanh toán", "check điều khoản hợp đồng", "tóm tắt và đánh giá rủi ro hợp đồng SEO", or when the user shares a Google Docs link or PDF link to an SEO service contract. Fetches the contract from a public URL, extracts all key clauses across 8 categories, and delivers a structured markdown report with risk ratings (Critical/Warning/OK). Use when onboarding a new client contract, reviewing renewal terms, or auditing an existing SEO agreement for liabilities.
---

# analyzing-seo-contracts

Đọc hợp đồng dịch vụ SEO từ Google Docs hoặc PDF link, trích xuất điều khoản theo 8 hạng mục, và output báo cáo markdown có risk rating cho từng mục.

## Khi nào dùng

User nói một trong các pattern:
- `/analyzing-seo-contracts [Google Docs URL hoặc PDF URL]`
- "phân tích hợp đồng SEO này: [link]"
- "review hợp đồng dịch vụ SEO"
- "đọc hợp đồng và tóm tắt KPI + thanh toán"
- "check điều khoản hợp đồng, đánh giá rủi ro"
- "tóm tắt hợp đồng SEO cho tôi"

KHÔNG dùng skill này khi:
- Input là hợp đồng lao động, hợp đồng mua bán hàng hóa, hoặc hợp đồng không liên quan dịch vụ SEO/digital marketing
- User chỉ hỏi "hợp đồng SEO cần có gì?" (trả lời thẳng, không cần chạy pipeline)
- Link không public hoặc yêu cầu đăng nhập (báo user thay vì cố fetch)

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| Ngôn ngữ report | Tiếng Việt | User yêu cầu English |
| Risk threshold Critical | Điều khoản hoàn toàn vắng mặt hoặc có lợi 1 chiều | — |
| Risk threshold Warning | Điều khoản mơ hồ, thiếu định lượng | — |
| Thiếu thông tin | Ghi "Không có" | — |

## Pipeline — 5 bước

Theo thứ tự, không skip.

### Step 1 — Fetch và extract văn bản hợp đồng

Nhận URL từ user. Phân biệt loại link:
- **Google Docs**: Fetch nội dung qua URL (append `/export?format=txt` nếu public). Nếu link yêu cầu đăng nhập → báo user và dừng.
- **PDF**: Fetch file và extract text. Nếu PDF scan (hình ảnh, không có text layer) → báo user "PDF này là ảnh scan, cần OCR trước khi phân tích".

Sau khi extract, xác nhận ngắn: "Đã đọc hợp đồng — [X] trang / [X] từ. Bắt đầu phân tích."

### Step 2 — Phân tích theo 8 hạng mục

Đọc `references/seo-contract-clauses.md` để biết chính xác các sub-item cần tìm trong từng hạng mục.

Với mỗi hạng mục, xác định:
1. Nội dung điều khoản (trích dẫn hoặc tóm tắt ngắn)
2. Risk level theo `references/scoring-rubric.md`
3. Lưu ý cụ thể nếu có vấn đề

**8 hạng mục phân tích:**

1. KPIs + Phương pháp đo lường
2. Timeline nghiệm thu + Lịch thanh toán
3. Điều khoản phạt
4. Hồ sơ nghiệm thu
5. Điều khoản chấm dứt hợp đồng sớm
6. Trách nhiệm khi thuật toán thay đổi
7. Quyền sở hữu dữ liệu và content
8. Bảo mật / NDA

Với mục nào không tìm thấy trong hợp đồng: ghi "Không có" và tự động assign **Critical** nếu mục đó nằm trong danh sách bắt buộc (xem `references/scoring-rubric.md`), hoặc **Warning** nếu mục khuyến nghị.

### Step 3 — Pre-delivery review

Trước khi output, self-check:
- [ ] Mỗi hạng mục đều có risk level (Critical/Warning/OK)
- [ ] Mọi trích dẫn claim "điều khoản X có nội dung Y" phải có thể tra lại trong văn bản gốc
- [ ] Không tự suy diễn rủi ro không có căn cứ trong văn bản
- [ ] Phân biệt "điều khoản vắng mặt" (Critical) vs "điều khoản mơ hồ" (Warning) — hai trường hợp khác nhau
- [ ] Bảng tổng hợp có đủ 8 hạng mục

### Step 4 — Render output

Dùng `assets/report-template.md` làm khung. Điền đủ:
1. Header: tên hợp đồng (nếu nhận ra được), ngày phân tích, tóm tắt 2 câu
2. Bảng tổng hợp 8 hạng mục với risk level
3. Mục "Lưu ý quan trọng" — liệt kê chỉ các điểm Critical và Warning, kèm giải thích cụ thể
4. Mục "Các mốc thời gian chính" — timeline dạng bullet theo thứ tự thời gian
5. Mục "Khuyến nghị" — tối đa 5 bullet actionable

### Step 5 — Báo cáo kết quả

Output report theo format trong `assets/report-template.md`.

### Step 6 — Gap Analysis (nếu có framework chuẩn)

Chỉ chạy bước này khi user yêu cầu so sánh hoặc file `assets/geo-framework-standard.md` tồn tại trong skill.

Đọc `assets/geo-framework-standard.md`. So sánh từng hạng mục đã phân tích ở Step 2 với chuẩn framework:

**Phân loại mỗi hạng mục:**
- **Đạt chuẩn**: điều khoản tương đương hoặc mạnh hơn framework
- **Gap nhỏ**: có điều khoản nhưng yếu hơn (vd: SLA khác, ngưỡng khác)
- **Gap lớn**: thiếu hoàn toàn điều khoản có trong framework
- **Khác loại**: hợp đồng dùng cơ chế khác scope (SEO truyền thống vs GEO) — không phải gap mà là khác phạm vi dịch vụ
- **Bổ sung**: hợp đồng có điều khoản không có trong framework

**Output bảng gap analysis** thêm vào sau phần Khuyến nghị:

```markdown
## Gap Analysis — So sánh với hợp đồng khung chuẩn GEO 2026

| Hạng mục | Hợp đồng phân tích | Framework chuẩn | Kết quả |
|---|---|---|---|
| 1. KPIs + Đo lường | [tóm tắt] | [tóm tắt chuẩn] | [Đạt / Gap nhỏ / Gap lớn / Khác loại] |
...

### Gaps cần xử lý ưu tiên
- **Gap lớn**: [liệt kê + đề xuất bổ sung]
- **Gap nhỏ**: [liệt kê + đề xuất điều chỉnh]

### Điểm hợp đồng mạnh hơn framework
- [liệt kê nếu có]
```

Kết thúc report bằng:

```
---
Phân tích hoàn tất. Nếu bạn muốn so sánh với framework hợp đồng khác,
cung cấp file template và tôi sẽ chạy lại gap analysis.
```

## Tiêu chí chất lượng (self-check)

Trước khi report:
- [ ] 8 hạng mục đều có entry (không bỏ trống)
- [ ] Risk level nhất quán với định nghĩa trong scoring-rubric.md
- [ ] Không có claim nào không có căn cứ trong văn bản hợp đồng
- [ ] Timeline bullet theo thứ tự thời gian
- [ ] Khuyến nghị actionable, không generic

## Anti-patterns

- KHÔNG assign risk level theo cảm tính — mọi Critical/Warning phải khớp rubric
- KHÔNG bịa điều khoản không có trong văn bản
- KHÔNG bỏ qua hạng mục nào dù hợp đồng ngắn
- KHÔNG output report nếu pre-delivery review chưa pass
- KHÔNG đánh giá "hợp đồng này hợp pháp hay không" — đây là phân tích điều khoản, không phải tư vấn pháp lý

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `references/seo-contract-clauses.md` | Checklist sub-items cần tìm trong từng hạng mục | Step 2 |
| `references/scoring-rubric.md` | Định nghĩa Critical/Warning/OK cho từng hạng mục | Step 2 |
| `assets/report-template.md` | Khung markdown output | Step 4 |
| `assets/geo-framework-standard.md` | Điều khoản chuẩn GEO 2026 để so sánh gap analysis | Step 6 |

## Sample timing

- Step 1 fetch: 30-60 giây
- Step 2 phân tích: 2-4 phút
- Step 3-4 review + render: 1-2 phút
- Step 5 report: 30 giây
- Step 6 gap analysis (nếu có): 2-3 phút thêm
- Total: **4-10 phút**
