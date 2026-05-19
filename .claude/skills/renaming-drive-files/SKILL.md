---
name: renaming-drive-files
description: This skill should be used when the user says "/renaming-drive-files", "đổi tên file Drive", "rename các file Google Drive này", "tôi có danh sách link + tên mới cần đổi", "đổi tên file theo danh sách", "rename Drive files theo list", or pastes a list of Google Drive URLs with corresponding new names. Nhận danh sách cặp (URL, tên mới) do user cung cấp, đổi tên từng file trên Google Drive qua gdrive CLI, trả về bảng gồm tên mới và link file.
---

# renaming-drive-files

Nhận danh sách cặp (Google Drive URL, tên mới) từ user, đổi tên từng file qua `gdrive` CLI, trả về bảng kết quả với tên đã đổi và link file.

## Khi nào dùng

User nói một trong các pattern:
- `/renaming-drive-files [paste danh sách url + tên mới]`
- "đổi tên file Drive theo danh sách này"
- "rename các file Google Drive này"
- "tôi có danh sách link + tên mới cần đổi"
- "rename Drive files theo list"

KHÔNG dùng skill này khi:
- User muốn đổi tên file local (không phải Google Drive)
- User chỉ muốn xem danh sách file, không cần rename
- Link không phải Google Drive (Dropbox, OneDrive, S3...)
- User chưa cung cấp tên mới — không tự detect hay đoán tên

## Default settings

| Setting | Default | Override khi |
|---|---|---|
| PATH gdrive | `/opt/homebrew/bin` prepend | gdrive có sẵn trong PATH hệ thống |
| Xử lý file lỗi | SKIP + ghi lý do, tiếp tục các file còn lại | User yêu cầu dừng hẳn khi có lỗi |
| Encoding tên file | Giữ nguyên chuỗi user cung cấp | Không normalize |

## Tiền điều kiện

- [ ] `gdrive` CLI đã cài và authenticated — kiểm tra: `export PATH="/opt/homebrew/bin:$PATH" && gdrive account list`
- [ ] User có quyền edit trên tất cả file trong danh sách
- [ ] User đã cung cấp tên mới cho mỗi file (không để trống)

## Pipeline — 3 bước

Theo thứ tự, không skip.

### Step 1 — Parse input, hiển thị preview

Đọc toàn bộ input của user. Với mỗi cặp (URL, tên mới):

Extract FILE_ID theo pattern:
- `docs.google.com/spreadsheets/d/{FILE_ID}/...` → lấy `{FILE_ID}`
- `docs.google.com/document/d/{FILE_ID}/...` → lấy `{FILE_ID}`
- `drive.google.com/file/d/{FILE_ID}/...` → lấy `{FILE_ID}`
- `drive.google.com/open?id={FILE_ID}` → lấy `{FILE_ID}`

Nếu URL không match → đánh dấu SKIP với lý do "không nhận dạng được Google Drive URL".
Nếu tên mới để trống → đánh dấu SKIP với lý do "thiếu tên mới".

Hiển thị bảng preview:

```
Preview — sẽ đổi tên các file sau:

| # | FILE_ID | Tên mới |
|---|---|---|
| 1 | {FILE_ID_1} | {TÊN_MỚI_1} |
| 2 | {FILE_ID_2} | {TÊN_MỚI_2} |

Xác nhận để tiếp tục? (yes/no)
```

**Decision point**: LUÔN dừng, chờ user confirm "yes" trước khi chạy lệnh rename.

### Step 2 — Rename từng file

Sau khi user confirm, chạy lần lượt cho mỗi file:

```bash
export PATH="/opt/homebrew/bin:$PATH" && gdrive files rename {FILE_ID} "{TÊN_MỚI}"
```

Ghi nhận kết quả từng file: thành công hoặc error message.
Nếu 1 file fail → SKIP file đó, tiếp tục các file còn lại. Không dừng toàn bộ pipeline.

### Step 3 — Output bảng kết quả

Trả về bảng tổng hợp sau khi xử lý xong tất cả file.

## Output format

```
| # | Tên mới | Link file | Trạng thái |
|---|---|---|---|
| 1 | {TÊN_MỚI_1} | https://drive.google.com/file/d/{FILE_ID_1} | Đã đổi tên |
| 2 | {TÊN_MỚI_2} | https://drive.google.com/file/d/{FILE_ID_2} | Đã đổi tên |

File bị skip (nếu có):
- {URL_GỐC} — lý do: {error message}
```

Link file trong bảng dùng format: `https://drive.google.com/file/d/{FILE_ID}`

## Tiêu chí chất lượng (self-check)

Trước khi báo hoàn thành:
- [ ] Tất cả FILE_ID đã được extract đúng từ URL
- [ ] Đã hiển thị preview và nhận confirm từ user trước khi rename
- [ ] Số hàng trong bảng kết quả khớp số file đã xử lý
- [ ] File bị SKIP có lý do cụ thể trong output
- [ ] Không tự đặt tên khác với tên user cung cấp

## Anti-patterns

- KHÔNG tự đặt tên file — dùng đúng tên user cung cấp, không normalize hay chỉnh sửa
- KHÔNG rename bất kỳ file nào trước khi user confirm ở Step 1
- KHÔNG bỏ qua file lỗi mà không báo — luôn list vào phần "File bị skip"

## Common pitfalls

1. **gdrive không có trong PATH** — Claude Code bash không tự load Homebrew PATH. Luôn prepend `export PATH="/opt/homebrew/bin:$PATH"` trước mỗi lệnh `gdrive`.
2. **FILE_ID sai cho Google Sheets** — Sheets URL có thêm `?gid=...` sau FILE_ID, phải cắt đúng tại `/edit` hoặc `/view`.
3. **File trong "Shared with me"** — `gdrive files rename` vẫn hoạt động, nhưng `gdrive files move` sẽ fail. Skill này chỉ rename nên không bị ảnh hưởng.
