# EVALS — generating-bbnt

3 scenarios để test skill có hoạt động đúng không. Chạy ở phiên Claude Code mới (fresh context).

---

## Eval 1: Golden path — đủ 4 inputs, tất cả KPIs có tài liệu

**User input**:
```
/generating-bbnt

BBNT mẫu: https://docs.google.com/document/d/1L3iWdncSGqvhdtAbezpF8L-ifCmAc8oU3JFCM5TeT5M/edit

Thông tin khách hàng:
- Tên KH: Vinamilk
- Số HĐ: SEO-2026-045
- Ngày ký HĐ: 01/01/2026
- Đợt nghiệm thu: 05
- Ngày lập BBNT: 19/05/2026
- Đại diện SEONGON: Nguyễn Văn A — Giám đốc dự án
- Đại diện KH: Trần Thị B — Marketing Manager

KPI kết quả:
- KPI 1: Top 10 cho 30 từ khoá chính → Đạt 32 từ khoá
- KPI 2: Top 3 cho 10 từ khoá thương hiệu → Đạt 10 từ khoá

Tài liệu nghiệm thu:
Từ khoá: "sữa tươi vinamilk" - thứ hạng #2
Từ khoá: "sữa chua vinamilk" - thứ hạng #4
[... 30+ từ khoá đạt top 10, 10 từ khoá thương hiệu đạt top 3]
```

**Expected behavior**:
1. Skill triggered `generating-bbnt`
2. Validate đủ 4 inputs — PASS, không hỏi thêm
3. Parse đúng: tên KH = VINAMILK, đợt = 05, ngày = 19/05/2026
4. Step 3 validate KPI documentation: cả 2 KPI có từ khoá đủ → PASS
5. Điền BBNT hoàn chỉnh, không còn placeholder trống
6. Xuất output với tên file: "SEONGON | VINAMILK - BBNT ĐỢT 05 | SEO 2026"

**Pass criteria**:
- [ ] Skill triggered đúng (không gọi skill khác)
- [ ] Tên file đúng convention: SEONGON | VINAMILK - BBNT ĐỢT 05 | SEO 2026
- [ ] Cả 2 KPIs trạng thái "Đạt"
- [ ] Không còn placeholder dạng [TÊN] trong output
- [ ] Có hướng dẫn tạo Google Doc ở cuối

---

## Eval 2: Edge case — KPI có trong hợp đồng nhưng thiếu tài liệu nghiệm thu

**User input**:
```
/generating-bbnt

BBNT mẫu: [link mẫu]

Thông tin khách hàng:
- Tên KH: Tiger Beer Vietnam
- Số HĐ: SEO-2026-031
- Đợt nghiệm thu: 03
- Ngày lập BBNT: 19/05/2026
- Đại diện SEONGON: Lê Văn C — Account Manager
- Đại diện KH: Phạm Thị D — Digital Manager

KPI kết quả:
- KPI 1: Top 10 cho 20 từ khoá → Đạt 21 từ khoá
- KPI 2: Organic traffic tăng 30% so với baseline → chưa có số liệu

Tài liệu nghiệm thu:
Từ khoá: "tiger beer" - #1
Từ khoá: "bia tiger lon" - #3
[20 từ khoá đầy đủ]
```

**Expected behavior**:
1. Skill triggered `generating-bbnt`
2. Step 3 detect: KPI 1 có đủ tài liệu, KPI 2 (organic traffic) thiếu số liệu trong acceptance_docs
3. **Dừng tại Step 3** — KHÔNG tự sang Step 4
4. Hỏi user rõ ràng: "KPI 2 (Organic traffic tăng 30%) chưa có số liệu trong tài liệu nghiệm thu. Bạn muốn: (1) Bổ sung số liệu traffic, hoặc (2) Đánh dấu KPI này là 'Chưa xác nhận' và tiếp tục?"
5. Sau khi user chọn → tiếp tục tạo BBNT với trạng thái KPI 2 = "Chưa xác nhận"

**Pass criteria**:
- [ ] Skill dừng tại Step 3, KHÔNG generate BBNT ngay
- [ ] Liệt kê đúng KPI nào thiếu tài liệu (KPI 2, không phải KPI 1)
- [ ] Hỏi user 2 lựa chọn rõ ràng
- [ ] Sau user confirm → tạo BBNT với KPI 2 = "Chưa xác nhận" (không tự đánh "Đạt")

---

## Eval 3: Anti-pattern — chỉ cung cấp 2/4 inputs bắt buộc

**User input**:
```
/generating-bbnt

Tôi cần tạo BBNT cho khách hàng Heineken, đợt 2.

KPI kết quả:
- Top 10 cho 25 từ khoá → Đạt 28 từ khoá
```

**Expected behavior**:
1. Skill triggered `generating-bbnt`
2. Step 1 detect: thiếu `client_info` đầy đủ (không có số HĐ, ngày, người đại diện) và thiếu hoàn toàn `acceptance_docs`
3. **Dừng ngay tại Step 1** — KHÔNG tự điền phần thiếu
4. Liệt kê rõ inputs còn thiếu và mỗi input cần chứa gì:
   - `client_info`: cần số HĐ, ngày ký HĐ, ngày lập BBNT, đại diện 2 bên
   - `acceptance_docs`: danh sách từ khoá + thứ hạng hiện tại
5. KHÔNG generate BBNT dù một phần

**Pass criteria**:
- [ ] Skill dừng tại Step 1, KHÔNG generate output BBNT
- [ ] Liệt kê đúng 2 inputs thiếu với mô tả cụ thể cần gì
- [ ] KHÔNG tự bịa thông tin để fill inputs trống
- [ ] User biết chính xác cần cung cấp thêm gì để chạy lại skill

---

## Cách chạy evals

1. Mở phiên Claude Code MỚI ở repo có skill installed
2. Copy User input từng scenario, paste vào Claude Code
3. So response với Expected behavior
4. Tick Pass criteria items

Nếu fail Eval 1 (golden) → kiểm tra pipeline Step 4-5.
Nếu fail Eval 2 (edge) → kiểm tra decision point tại Step 3.
Nếu fail Eval 3 (anti) → kiểm tra validation cứng tại Step 1.
