---
name: generating-bbnt
description: This skill should be used when the user says "/generating-bbnt", "tạo BBNT cho khách hàng", "điền biên bản nghiệm thu", "generate BBNT tháng này", "chuẩn bị nghiệm thu SEO", "tôi cần tạo biên bản nghiệm thu", "fill BBNT mẫu với kết quả", or pastes client contract info + KPI results and asks to create an acceptance document. Nhận 4 inputs bắt buộc (khung BBNT mẫu, thông tin khách hàng và hợp đồng, kết quả KPIs, tài liệu nghiệm thu từ khoá + thứ hạng), validate tài liệu nghiệm thu cho từng KPI, điền vào mẫu và xuất Google Doc theo tên chuẩn SEONGON. Dùng khi chuẩn bị bàn giao nghiệm thu SEO theo đợt cho khách hàng.
---

# generating-bbnt

Điền đầy đủ thông tin khách hàng, hợp đồng, kết quả KPIs và tài liệu nghiệm thu vào BBNT mẫu SEONGON, sau đó xuất nội dung Google Doc mới theo tên chuẩn.

## Khi nào dùng

User nói một trong các pattern:
- `/generating-bbnt`
- "tạo BBNT cho khách hàng [tên]"
- "điền biên bản nghiệm thu đợt [XX]"
- "generate BBNT tháng này"
- "chuẩn bị nghiệm thu SEO cho [tên khách hàng]"
- "fill BBNT mẫu với kết quả"

KHÔNG dùng skill này khi:
- User chỉ muốn xem hoặc kiểm tra kết quả KPIs (không cần tạo document)
- User muốn phân tích điều khoản hợp đồng SEO (dùng `/analyzing-seo-contracts`)

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| BBNT template | Mẫu SEONGON chuẩn (xem Step 1) | User cung cấp link template khác |
| Năm trong tên file | 2026 | User chỉ định năm khác |
| Ngôn ngữ BBNT | Tiếng Việt | User yêu cầu tiếng Anh |

## Input schema

| Param | Type | Required | Note |
|---|---|---|---|
| bbnt_template | Google Doc URL | yes | Mặc định dùng mẫu SEONGON ở Step 1 nếu không cung cấp |
| client_info | text paste | yes | Tên KH, số HĐ, ngày ký HĐ, người đại diện 2 bên, số đợt nghiệm thu |
| kpi_results | text paste | yes | Danh sách KPI cam kết trong HĐ → kết quả thực tế đạt được |
| acceptance_docs | text paste | yes | Danh sách từ khoá + thứ hạng hiện tại (tài liệu nghiệm thu) |

## Pipeline — 5 bước

Theo thứ tự, không skip.

### Step 1 — Validate input

Kiểm tra đủ 4 inputs: bbnt_template, client_info, kpi_results, acceptance_docs.

Nếu thiếu bất kỳ input nào: liệt kê rõ input nào thiếu, giải thích input đó cần chứa gì, và dừng — KHÔNG tự proceed với input chưa đủ.

Template mặc định SEONGON: https://docs.google.com/document/d/1L3iWdncSGqvhdtAbezpF8L-ifCmAc8oU3JFCM5TeT5M/edit

### Step 2 — Parse thông tin đầu vào

Trích xuất các trường sau từ client_info và kpi_results. Tham khảo `assets/bbnt-placeholder-map.md` để biết mapping đầy đủ.

Từ client_info:
- Tên khách hàng (dùng để đặt tên file output — viết hoa toàn bộ)
- Số đợt nghiệm thu → format 2 chữ số zero-padded (01, 02, ...)
- Số hợp đồng
- Ngày lập biên bản → format DD/MM/YYYY
- Người đại diện SEONGON + chức vụ
- Người đại diện khách hàng + chức vụ

Từ kpi_results: danh sách KPI dạng [Tên KPI] | [Cam kết] | [Thực tế] cho mỗi mục.

Nếu bất kỳ trường bắt buộc nào không đọc được từ input: hỏi user trực tiếp, không tự suy đoán.

### Step 3 — Validate KPI documentation

Với mỗi KPI trong kpi_results, kiểm tra acceptance_docs có đủ tài liệu nghiệm thu không:

| Loại KPI | Tài liệu cần có trong acceptance_docs |
|---|---|
| Thứ hạng từ khoá | Danh sách từ khoá + thứ hạng cụ thể |
| Số từ khoá top N | Đếm đủ số từ khoá đạt ngưỡng trong danh sách |
| Organic traffic | Số liệu traffic hoặc xuất báo cáo GSC/GA4 |
| KPI khác | Bằng chứng cụ thể tương ứng |

Lập danh sách kết quả mapping:
- KPI có tài liệu đầy đủ → ghi nhận, tiếp tục
- KPI thiếu tài liệu → thêm vào "KPIs cần bổ sung"

**Decision point**: Nếu có ≥1 KPI thiếu tài liệu, dừng và hỏi user:
1. Bổ sung tài liệu và tiếp tục, hoặc
2. Đánh dấu KPI đó là "Chưa xác nhận" và vẫn tạo BBNT

KHÔNG tự quyết — chờ user confirm trước khi sang Step 4.

### Step 4 — Điền BBNT

Điền toàn bộ nội dung BBNT theo mapping trong `assets/bbnt-placeholder-map.md`.

Gán trạng thái từng KPI:
- **Đạt**: có tài liệu nghiệm thu + kết quả đạt ngưỡng cam kết trong HĐ
- **Không đạt**: có tài liệu nghiệm thu + kết quả không đạt ngưỡng
- **Chưa xác nhận**: thiếu tài liệu hoặc user confirm bỏ qua

Khi điền bảng từ khoá và thứ hạng: liệt kê đầy đủ từ acceptance_docs, nhóm theo trạng thái (đạt / chưa đạt ngưỡng cam kết).

Self-check trước khi sang Step 5: search "[" trong toàn bộ nội dung đã điền. Nếu còn placeholder dạng [TÊN] chưa điền → điền trước khi xuất.

### Step 5 — Xuất output + báo cáo

Đặt tên file theo convention:
```
SEONGON | {TÊN KHÁCH HÀNG} - BBNT ĐỢT {XX} | SEO 2026
```

Hướng dẫn user tạo Google Doc:
1. Copy toàn bộ nội dung BBNT đã điền bên dưới
2. Mở Google Drive → tạo Google Doc mới
3. Đặt tên file đúng convention trên
4. Paste nội dung vào

Xuất báo cáo theo format bên dưới.

## Output naming convention

`SEONGON | {TÊN KHÁCH HÀNG} - BBNT ĐỢT {XX} | SEO 2026`

- TÊN KHÁCH HÀNG: viết hoa toàn bộ, đúng theo tên trong hợp đồng (vd: VINAMILK, TIGER BEER)
- XX: 2 chữ số zero-padded (01, 02, 10, 11)
- Năm: lấy từ hợp đồng nếu có, mặc định 2026

## Output format

```
BBNT generated — {tên file đầy đủ}

KPIs: {X}/{Y} có đủ tài liệu nghiệm thu
KPIs thiếu tài liệu: {danh sách, hoặc "Không có"}

---
{Nội dung BBNT đầy đủ đã điền — sẵn sàng copy vào Google Doc}
---
```

## Tiêu chí chất lượng (self-check)

Trước khi xuất output:
- [ ] Không còn placeholder dạng [TÊN] nào trong nội dung (search "[" để verify)
- [ ] Mỗi KPI có trạng thái rõ: Đạt / Không đạt / Chưa xác nhận
- [ ] Mỗi KPI thứ hạng từ khoá có ≥1 từ khoá dẫn chứng từ acceptance_docs
- [ ] Tên file đúng convention (kiểm tra TÊN KHÁCH HÀNG viết hoa, XX đúng 2 chữ số)
- [ ] Ngày tháng đúng format DD/MM/YYYY

## Anti-patterns

- KHÔNG tạo BBNT khi thiếu bất kỳ 1 trong 4 inputs bắt buộc — BAD: tiếp tục với 3/4 inputs và tự điền phần còn thiếu
- KHÔNG tự điền kết quả KPI không có trong kpi_results — BAD: đánh "Đạt" cho KPI mà kpi_results không đề cập đến
- KHÔNG bỏ qua KPI thiếu tài liệu mà không hỏi user — BAD: đánh "Đạt" cho KPI không có dẫn chứng trong acceptance_docs
- KHÔNG đặt tên file sai convention — BAD: "BBNT Vinamilk T5" thay vì "SEONGON | VINAMILK - BBNT ĐỢT 05 | SEO 2026"

## Common pitfalls

1. **KPI thiếu bằng chứng nhưng vẫn "Đạt"** — Nếu không có từ khoá matching trong acceptance_docs, đánh "Chưa xác nhận" và báo user, không tự điền để cho xong
2. **Sai số đợt nghiệm thu** — Đọc kỹ client_info, không suy đoán từ ngày tháng hay lịch sử BBNT trước
3. **Placeholder còn sót** — Step 4 có self-check, nhưng nếu BBNT mẫu có placeholder format khác (vd: <<TÊN>>, {TÊN}) thì search cả 3 dạng

## Skill files

| File | Purpose | Khi nào load |
|---|---|---|
| `assets/bbnt-placeholder-map.md` | Mapping placeholder BBNT mẫu SEONGON → input fields | Step 2 và Step 4 khi điền thông tin |
