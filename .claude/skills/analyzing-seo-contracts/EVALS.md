# EVALS — analyzing-seo-contracts

3 scenarios để test skill có hoạt động đúng không. Chạy ở phiên Claude Code mới (fresh context).

---

## Eval 1: Golden path — Hợp đồng đầy đủ điều khoản

**Tên scenario**: Hợp đồng SEO 12 tháng có đủ 8 hạng mục

**Precondition**:
- Link Google Docs hợp đồng là public (ai có link đều xem được)
- Hợp đồng có ít nhất: KPI, lịch thanh toán, điều khoản phạt, hồ sơ nghiệm thu

**User input**:
```
/analyzing-seo-contracts https://docs.google.com/document/d/[ID]/edit?usp=sharing
```

**Expected behavior**:
1. Skill fetch URL và xác nhận đọc được (báo số trang / số từ)
2. Phân tích 8 hạng mục theo thứ tự, không skip
3. Assign risk level theo scoring-rubric.md — không phải cảm tính
4. Output markdown report theo assets/report-template.md với đủ 5 sections
5. Kết thúc bằng offer gap analysis nếu có template khung

**Pass criteria**:
- [ ] Skill triggered đúng (không trả lời generic)
- [ ] Output có bảng tổng hợp 8 hàng, mỗi hàng có Risk Level rõ
- [ ] Mục "Lưu ý quan trọng" phân biệt Critical vs Warning
- [ ] Mục "Các mốc thời gian" có ≥2 bullet nếu hợp đồng có timeline
- [ ] Không có claim nào không có căn cứ trong văn bản

---

## Eval 2: Edge case — Hợp đồng thiếu nhiều điều khoản

**Tên scenario**: Hợp đồng khung sơ sài, chỉ có 1-2 trang

**User input**:
```
phân tích hợp đồng SEO này cho tôi: https://docs.google.com/document/d/[ID_SHORT_CONTRACT]/edit
```

**Expected behavior**:
1. Skill fetch và xác nhận hợp đồng ngắn
2. Phân tích từng hạng mục — với hạng mục không tìm thấy trong văn bản, ghi "Không có" và assign risk tương ứng
3. Nếu <4 hạng mục có data → output cảnh báo INSUFFICIENT DATA trước khi report
4. Không bịa hoặc suy diễn điều khoản không có trong văn bản
5. Mục "Lưu ý quan trọng" có nhiều Critical do hợp đồng thiếu điều khoản

**Pass criteria**:
- [ ] Mỗi hạng mục thiếu được ghi "Không có" rõ ràng (không bỏ qua)
- [ ] "Không có" + hạng mục bắt buộc → assign Critical đúng
- [ ] Không hallucinate điều khoản không có trong văn bản
- [ ] Mục khuyến nghị nêu cụ thể điều khoản nào cần bổ sung

---

## Eval 3: Anti-pattern — Input không phải hợp đồng SEO

**Tên scenario**: User đưa hợp đồng lao động hoặc hợp đồng mua hàng

**User input**:
```
/analyzing-seo-contracts https://docs.google.com/document/d/[ID_LABOR_CONTRACT]/edit
```
(Link trỏ tới hợp đồng lao động, không phải hợp đồng dịch vụ SEO)

**Expected behavior**:
1. Skill fetch và đọc nội dung
2. Detect rằng đây không phải hợp đồng dịch vụ SEO/digital marketing
3. Không cố phân tích theo 8 hạng mục SEO — sẽ không có data hoặc data sai context
4. Báo rõ: "Đây là [loại hợp đồng], không phải hợp đồng dịch vụ SEO. Skill này chỉ phân tích hợp đồng dịch vụ SEO/digital marketing."
5. Offer: "Nếu bạn muốn tôi phân tích hợp đồng này theo framework khác, hãy mô tả yêu cầu."

**Pass criteria**:
- [ ] Skill KHÔNG cứ chạy 8 hạng mục SEO lên hợp đồng sai loại
- [ ] Có explanation rõ tại sao không phù hợp
- [ ] Không error out — vẫn offer next step
- [ ] User hiểu cần làm gì tiếp

---

## Cách chạy evals

1. Mở phiên Claude Code MỚI ở repo có skill installed
2. Copy User input từng scenario, paste vào CC (thay [ID] bằng link thật)
3. So Claude's response với Expected behavior
4. Tick Pass criteria items

**Nếu fail Eval 1** (golden) → skill có vấn đề core, kiểm tra pipeline trong SKILL.md
**Nếu fail Eval 2** (edge) → thiếu graceful handling, bổ sung vào Anti-patterns
**Nếu fail Eval 3** (anti) → thiếu guardrail, bổ sung vào "Khi nào KHÔNG dùng"

## Eval results log

| Date | Skill version | Eval 1 | Eval 2 | Eval 3 | Notes |
|---|---|---|---|---|---|
| 2026-05-19 | v0.1 | — | — | — | Initial release |
