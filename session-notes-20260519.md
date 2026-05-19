# Session Notes — 2026-05-19
# Xây dựng skill phân tích hợp đồng SEO + Phân tích hợp đồng thực tế BTMH

---

## Mục tiêu ban đầu

Tạo Claude Code skill tự động hóa việc review hợp đồng dịch vụ SEO:
- Input: link Google Docs hoặc PDF hợp đồng
- Output: bảng tổng hợp KPI + timeline nghiệm thu/thanh toán, risk rating, lưu ý điều khoản phạt, các mốc quan trọng

---

## Skill đã tạo: `analyzing-seo-contracts`

**Vị trí sau khi move**: `~/.claude/skills/analyzing-seo-contracts/` (global — hoạt động trong mọi project)

### Cấu trúc files

```
~/.claude/skills/analyzing-seo-contracts/
├── SKILL.md                          — pipeline 6 bước
├── EVALS.md                          — 3 test scenarios
├── references/
│   ├── seo-contract-clauses.md       — checklist sub-items cho 8 hạng mục
│   └── scoring-rubric.md             — định nghĩa Critical/Warning/OK
└── assets/
    ├── report-template.md            — khung markdown output
    └── geo-framework-standard.md     — điều khoản chuẩn GEO 2026 (dùng cho gap analysis)
```

### Pipeline 6 bước

| Bước | Nội dung |
|---|---|
| Step 1 | Fetch hợp đồng từ URL (Google Docs export txt / PDF text extract) |
| Step 2 | Phân tích 8 hạng mục theo `seo-contract-clauses.md` + assign risk theo `scoring-rubric.md` |
| Step 3 | Pre-delivery self-check (không bịa, không bỏ hạng mục nào) |
| Step 4 | Render report theo `report-template.md` |
| Step 5 | Output report |
| Step 6 | Gap analysis — so sánh với `geo-framework-standard.md` nếu user yêu cầu |

### 8 hạng mục phân tích

1. KPIs + Phương pháp đo lường
2. Timeline nghiệm thu + Lịch thanh toán
3. Điều khoản phạt
4. Hồ sơ nghiệm thu
5. Điều khoản chấm dứt hợp đồng sớm
6. Trách nhiệm khi thuật toán thay đổi
7. Quyền sở hữu dữ liệu và content
8. Bảo mật / NDA

### Risk levels

| Level | Ý nghĩa |
|---|---|
| Critical | Điều khoản vắng mặt hoặc gây bất lợi nghiêm trọng — phải sửa trước khi ký |
| Warning | Điều khoản mơ hồ, thiếu định lượng, thiếu cân bằng — nên làm rõ |
| OK | Điều khoản rõ ràng, đo được, cân bằng |

### Cách trigger skill

```
/analyzing-seo-contracts [Google Docs URL hoặc PDF URL]
phân tích hợp đồng SEO này: [link]
check điều khoản hợp đồng, đánh giá rủi ro
```

---

## Hợp đồng thực tế đã phân tích: BTMH × SEONGON

**Hợp đồng số**: 78/2026/BMTH-SEONGON  
**Bên A**: Công ty TNHH Bảo Tín Mạnh Hải (trang sức vàng)  
**Bên B**: SEONGON Thịnh Vượng  
**Dịch vụ**: SEO AI Max cho baotinmanhhai.vn — 12 tháng  
**Giá trị**: 1.823.715.000 VNĐ (đã VAT, đã chiết khấu 5%)  
**Trạng thái**: DRAFT — ngày ký để trống, có comments nội bộ [a][b][c][d]

### Kết quả phân tích (0 Critical / 6 Warning / 2 OK)

| # | Hạng mục | Risk | Vấn đề chính |
|---|---|---|---|
| 1 | KPIs + Đo lường | Warning | Thiếu baseline; mẫu số AIO thay đổi được; tool đo (Semrush) không đo được ChatGPT brand mention |
| 2 | Timeline nghiệm thu + Thanh toán | Warning | Mốc ngày cụ thể (30/9, 31/12, 31/3) nhưng ngày ký hợp đồng còn trống |
| 3 | Điều khoản phạt | Warning | Cliff penalty tại 80% không tuyến tính; phạt Bên B đơn phương hủy nhưng Bên A chỉ bị lãi suất ngân hàng |
| 4 | Hồ sơ nghiệm thu | OK | Có danh sách deliverable, review period 5 ngày LV, có quy trình dispute |
| 5 | Chấm dứt hợp đồng sớm | OK | Cân bằng 2 chiều: 20 ngày notice + 8% penalty; vi phạm: 15 ngày notice |
| 6 | Trách nhiệm thuật toán | Warning | Algorithm update = BKK (đúng), nhưng không có SLA response, không phân biệt black-hat |
| 7 | Quyền sở hữu dữ liệu | Warning | Không có điều khoản về quyền sở hữu content và data sau terminate |
| 8 | Bảo mật / NDA | Warning | NDA có nhưng thiếu thời hạn và không có điều khoản cho subcontractors |

### KPIs chi tiết trong hợp đồng BTMH

| Nhóm từ khóa | Số lượng |
|---|---|
| Vàng tích trữ | 523 keywords |
| Hotkey | 68 keywords |
| Vàng 24K | 384 keywords |
| AIO (AI Overviews) | Tỷ lệ % — để trống template |
| Traffic | 6 triệu clicks organic/năm |

**Tools đo lường**: Semrush (keyword ranking) + Google Search Console (traffic)  
**Nghiệm thu**: 3 đợt — 30/9/2026 (20% KPI), 31/12/2026 (50%), 31/3/2027 (100%)  
**Thanh toán**: 30% tạm ứng + 3 đợt nghiệm thu (SLA 7 ngày LV/đợt)

---

## Hợp đồng khung GEO 2026 — đã lưu làm reference

**File**: `~/.claude/skills/analyzing-seo-contracts/assets/geo-framework-standard.md`  
**Nguồn**: SEONGON — HỢP ĐỒNG DỊCH VỤ GEO 2026 (CÓ ONPAGE).txt

### Điểm khác biệt chính so với BTMH

| Hạng mục | GEO 2026 Framework | BTMH (SEO AI Max) |
|---|---|---|
| Loại dịch vụ | GEO thuần (ChatGPT + AIO) | SEO truyền thống + AIO |
| KPIs | Brand Mention ChatGPT %, URL Citation AIO %, Brand Mention AIO % | Keyword rankings + Traffic + AIO |
| Tool đo | SE Ranking + SERPUPDATE | Semrush + GSC |
| SLA thanh toán | 5 ngày làm việc/đợt | 7 ngày làm việc/đợt |
| Bảo hành | 1 tháng | 2 tháng |
| Phạt Bên A hủy | 8% giá trị chưa thực hiện | Chỉ lãi suất Vietcombank |
| Onpage đặc thù GEO | llms.txt, GPT-Bot robots.txt, Share to AI, JS/SSR, AI Bot log | Không có |

### Gap analysis BTMH vs GEO Framework

**Gap lớn (cần bổ sung vào BTMH trước khi ký):**
- Điều khoản phạt Bên A khi đơn phương hủy (hiện chỉ phạt lãi suất, framework là 8%)
- Quyền sở hữu content/data sau terminate (cả 2 bên đều thiếu)

**Gap nhỏ:**
- SLA thanh toán (7 ngày vs. 5 ngày trong framework)
- Thiếu quy trình dispute trung gian khi không đồng ý nghiệm thu

**BTMH mạnh hơn framework:**
- Có Phụ lục 04 báo cáo hàng tuần/tháng (framework GEO không có)
- Bảo hành 2 tháng (framework 1 tháng)

**Khác loại (không phải gap):**
- KPI và tool đo khác vì scope dịch vụ khác (SEO AI Max vs GEO thuần)

---

## Bài học kỹ thuật trong phiên

### Google Docs fetch
- WebFetch không thể authenticate Google Docs dù link public → HTTP 401
- Giải pháp: user paste text trực tiếp hoặc dùng export link `.../export?format=txt` cho docs thực sự public

### Skills management
- Skills trong `.claude/skills/` của project → chỉ hoạt động trong project đó
- Skills trong `~/.claude/skills/` → hoạt động toàn cục mọi project
- Sau khi move vào `~/.claude/skills/`: restart Claude Code để skills xuất hiện trong `/skills` list

### Skill đã move global

```
~/.claude/skills/
├── analyzing-seo-contracts/   — phân tích hợp đồng SEO/GEO
└── create-skill/              — tạo skill mới theo chuẩn Anthropic
```

---

## Files chính được tạo/cập nhật trong phiên

| File | Mô tả |
|---|---|
| `~/.claude/skills/analyzing-seo-contracts/SKILL.md` | Pipeline 6 bước (thêm Step 6 gap analysis) |
| `~/.claude/skills/analyzing-seo-contracts/assets/geo-framework-standard.md` | Điều khoản chuẩn GEO 2026 — mới tạo |
| `~/.claude/skills/analyzing-seo-contracts/references/scoring-rubric.md` | Rubric Critical/Warning/OK |
| `~/.claude/skills/analyzing-seo-contracts/references/seo-contract-clauses.md` | Checklist 8 hạng mục |
| `~/.claude/skills/analyzing-seo-contracts/assets/report-template.md` | Template markdown output |
| `~/.claude/skills/analyzing-seo-contracts/EVALS.md` | 3 test scenarios |
