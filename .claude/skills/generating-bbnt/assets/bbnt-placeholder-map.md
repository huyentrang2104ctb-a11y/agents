<!-- Purpose: Mapping từng placeholder trong BBNT mẫu SEONGON → input field tương ứng.
     Nguồn mẫu: https://docs.google.com/document/d/1L3iWdncSGqvhdtAbezpF8L-ifCmAc8oU3JFCM5TeT5M/edit
     Cập nhật file này nếu BBNT mẫu thêm placeholder mới. -->

# BBNT Placeholder Map

Dùng tại Step 2 (parse) và Step 4 (điền) của skill `generating-bbnt`.

## Thông tin chung

| Placeholder trong mẫu | Nguồn input | Trường cụ thể |
|---|---|---|
| [TÊN KHÁCH HÀNG] | client_info | Tên công ty khách hàng (viết hoa) |
| [SỐ HĐ] | client_info | Số hợp đồng dịch vụ SEO |
| [NGÀY KÝ HĐ] | client_info | Ngày ký hợp đồng (DD/MM/YYYY) |
| [ĐỢT NGHIỆM THU] | client_info | Số đợt, 2 chữ số (01, 02, ...) |
| [NGÀY LẬP BBNT] | client_info | Ngày lập biên bản (DD/MM/YYYY) |
| [ĐẠI DIỆN SEONGON] | client_info | Họ tên người đại diện bên SEONGON |
| [CHỨC VỤ SEONGON] | client_info | Chức vụ người đại diện SEONGON |
| [ĐẠI DIỆN KHÁCH HÀNG] | client_info | Họ tên người đại diện bên khách hàng |
| [CHỨC VỤ KHÁCH HÀNG] | client_info | Chức vụ người đại diện khách hàng |

## Bảng KPIs

Điền lặp lại cho từng KPI trong kpi_results:

| Placeholder trong mẫu | Nguồn input | Trường cụ thể |
|---|---|---|
| [TÊN KPI N] | kpi_results | Tên chỉ tiêu KPI (vd: "Số từ khoá top 10") |
| [KPI CAM KẾT N] | kpi_results | Giá trị cam kết trong hợp đồng |
| [KPI THỰC TẾ N] | kpi_results | Kết quả thực tế đạt được |
| [TRẠNG THÁI N] | kpi_results + acceptance_docs | Đạt / Không đạt / Chưa xác nhận |
| [GHI CHÚ KPI N] | kpi_results | Ghi chú thêm nếu có (optional) |

## Bảng từ khoá nghiệm thu

Điền từ acceptance_docs, mỗi hàng 1 từ khoá:

| Placeholder trong mẫu | Nguồn input | Trường cụ thể |
|---|---|---|
| [TỪ KHOÁ N] | acceptance_docs | Từ khoá cần nghiệm thu |
| [THỨ HẠNG HIỆN TẠI N] | acceptance_docs | Thứ hạng thực tế (vd: #3, #7) |
| [THỨ HẠNG CAM KẾT N] | kpi_results | Ngưỡng thứ hạng cam kết (vd: top 10) |
| [URL TRANG RANK N] | acceptance_docs | URL trang đang rank (nếu có) |
| [TRẠNG THÁI TỪ KHOÁ N] | computed | Đạt / Chưa đạt dựa trên so sánh thực tế vs cam kết |

## Ghi chú về trạng thái KPI

Quy tắc gán trạng thái tự động:
- **Đạt**: kết quả thực tế ≥ cam kết trong hợp đồng + có tài liệu dẫn chứng
- **Không đạt**: kết quả thực tế < cam kết + có tài liệu dẫn chứng
- **Chưa xác nhận**: thiếu dữ liệu trong acceptance_docs, hoặc user xác nhận bỏ qua validation

## Cập nhật map này

Nếu BBNT mẫu thay đổi (thêm/xóa section): cập nhật bảng mapping tương ứng.
Nếu placeholder trong mẫu dùng format khác (vd: <<TÊN>>, {TÊN}): thêm ghi chú format ở đầu bảng tương ứng.
