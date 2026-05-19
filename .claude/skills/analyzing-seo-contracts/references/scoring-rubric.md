# Scoring Rubric — Risk Levels cho SEO Contract Analysis

Dùng file này ở Step 2 để assign risk level nhất quán cho từng hạng mục.

## Contents
- Định nghĩa 3 mức risk
- Rubric theo hạng mục
- Data sufficiency gate
- Bảng tham chiếu nhanh

---

## Định nghĩa 3 mức risk

| Level | Ý nghĩa | Hành động khuyến nghị |
|---|---|---|
| **Critical** | Điều khoản vắng mặt hoàn toàn, hoặc có nội dung gây bất lợi nghiêm trọng cho client | Phải bổ sung / đàm phán lại trước khi ký |
| **Warning** | Điều khoản tồn tại nhưng mơ hồ, thiếu định lượng, hoặc thiếu cân bằng nhẹ | Nên làm rõ hoặc bổ sung trước khi ký |
| **OK** | Điều khoản rõ ràng, đo được, cân bằng hai bên | Không cần hành động |

---

## Rubric theo hạng mục

### Hạng mục 1: KPIs + Phương pháp đo lường

| Condition | Risk |
|---|---|
| Có KPI cụ thể + baseline + tool đo + tần suất báo cáo | OK |
| Có KPI nhưng thiếu 1-2 trong: baseline, tool, tần suất | Warning |
| KPI định tính không đo được ("cải thiện SEO chung") | Warning |
| Hoàn toàn không có KPI | Critical |

### Hạng mục 2: Timeline nghiệm thu + Lịch thanh toán

| Condition | Risk |
|---|---|
| Có milestone dates + điều kiện kích hoạt + ngày thanh toán cụ thể | OK |
| Có lịch thanh toán nhưng ngày mơ hồ ("sau khi nghiệm thu" không có SLA) | Warning |
| Thanh toán 100% upfront không có milestone nghiệm thu | Warning |
| Không có lịch thanh toán hoặc không có timeline hợp đồng | Critical |

### Hạng mục 3: Điều khoản phạt

| Condition | Risk |
|---|---|
| Phạt cân bằng 2 chiều + có cap + có quy trình xác nhận vi phạm | OK |
| Phạt tồn tại nhưng thiếu cap hoặc quy trình xác nhận | Warning |
| Phạt chỉ áp dụng 1 chiều (bất lợi cho client) | Critical |
| Không có điều khoản phạt nào | Warning |

### Hạng mục 4: Hồ sơ nghiệm thu

| Condition | Risk |
|---|---|
| Có danh sách deliverable rõ + review period ≥3 ngày + quy trình dispute | OK |
| Có deliverable nhưng thiếu review period hoặc quy trình dispute | Warning |
| Review period <3 ngày làm việc | Warning |
| Không có danh sách deliverable / điều kiện nghiệm thu | Critical |

### Hạng mục 5: Điều khoản chấm dứt hợp đồng sớm

| Condition | Risk |
|---|---|
| Có điều kiện terminate + notice period hợp lý (14-30 ngày) + quy trình thanh toán proportional | OK |
| Có terminate nhưng notice period >60 ngày hoặc phạt >30% giá trị còn lại | Warning |
| Terminate chỉ có lợi cho 1 bên | Critical |
| Không có điều khoản terminate | Critical |

### Hạng mục 6: Trách nhiệm khi thuật toán thay đổi

| Condition | Risk |
|---|---|
| Có quy trình xử lý algorithm updates + SLA response + cơ chế điều chỉnh KPI | OK |
| Đề cập algorithm updates nhưng không có SLA hoặc quy trình rõ | Warning |
| Không đề cập algorithm updates | Warning |
| Bên dịch vụ hoàn toàn miễn trách mọi trường hợp kể cả do black-hat | Critical |

### Hạng mục 7: Quyền sở hữu dữ liệu và content

| Condition | Risk |
|---|---|
| Client sở hữu toàn bộ content + data + tài khoản tool | OK |
| Client sở hữu nhưng có giới hạn nhỏ (vd: không dùng case study) | Warning |
| Bên dịch vụ giữ quyền sở hữu một phần content/data | Critical |
| Không đề cập quyền sở hữu sau hợp đồng | Warning |

### Hạng mục 8: Bảo mật / NDA

| Condition | Risk |
|---|---|
| Có NDA rõ ràng + thời hạn ≥2 năm + áp dụng cả subcontractors | OK |
| Có NDA nhưng thiếu thời hạn hoặc không áp dụng subcontractors | Warning |
| Thời hạn bảo mật <1 năm | Warning |
| Không có điều khoản bảo mật nào | Warning |

---

## Data sufficiency gate

Nếu có <4 hạng mục có đủ dữ liệu để phân tích (vd: hợp đồng quá ngắn, chỉ 1 trang), output:

```
INSUFFICIENT DATA — Hợp đồng không đủ nội dung để phân tích đầy đủ 8 hạng mục.
Chỉ có [X] hạng mục có thể đánh giá.
Phân tích partial được cung cấp dưới đây với note rõ các hạng mục thiếu.
```

Không output tổng risk score nếu <6/8 hạng mục có data.

---

## Bảng tham chiếu nhanh

| Hạng mục | Bắt buộc (Critical nếu thiếu) | Khuyến nghị (Warning nếu thiếu) |
|---|---|---|
| KPIs | Có KPI đo được | Baseline + tool đo |
| Timeline thanh toán | Lịch thanh toán có ngày cụ thể | Điều kiện kích hoạt |
| Điều khoản phạt | — | Tồn tại và cân bằng |
| Hồ sơ nghiệm thu | Danh sách deliverable | Review period + dispute |
| Chấm dứt sớm | Có điều khoản terminate | Notice period hợp lý |
| Thuật toán thay đổi | — | Đề cập và có quy trình |
| Quyền sở hữu | Client sở hữu data + content | Áp dụng cả sau terminate |
| Bảo mật / NDA | — | Có NDA ≥2 năm |
