# EVALS — renaming-drive-files

3 scenarios để test skill có hoạt động đúng không. Chạy ở phiên Claude Code mới (fresh context).

---

## Eval 1: Golden path

**Tên scenario**: Danh sách 2 file hợp lệ, đủ tên mới

**Precondition**:
- `gdrive` CLI đã cài tại `/opt/homebrew/bin/gdrive`
- Đã authenticated: `gdrive account list` hiển thị account
- User có quyền edit trên cả 2 file

**User input**:
```
/renaming-drive-files

https://docs.google.com/spreadsheets/d/1dGK78jY5FDm3-JACH3CgJbBer1o91WKL2Zkmvfac728/edit
SEONGON | BTMH - Plan nội bộ | SEO 2026

https://docs.google.com/spreadsheets/d/1DPM5BerK_gmm1AyTNWQ3ZM4S_2UEnq_vEZdjKBwU11s/edit
SEONGON | BTMH - CONTENT | SEO 2026
```

**Expected behavior**:
1. Claude triggers skill `renaming-drive-files`
2. Extract FILE_ID: `1dGK78jY5FDm3-JACH3CgJbBer1o91WKL2Zkmvfac728` và `1DPM5BerK_gmm1AyTNWQ3ZM4S_2UEnq_vEZdjKBwU11s`
3. Hiển thị bảng preview với 2 hàng, hỏi "Xác nhận để tiếp tục?"
4. Sau khi user gõ "yes", chạy `gdrive files rename` cho từng file
5. Trả về bảng kết quả với cột: Tên mới / Link file / Trạng thái = "Đã đổi tên"

**Pass criteria**:
- [ ] Skill triggered đúng (không nhầm skill khác)
- [ ] Preview table hiển thị trước khi rename — không rename luôn
- [ ] Cả 2 file có trạng thái "Đã đổi tên" trong output
- [ ] Link file dùng format `https://drive.google.com/file/d/{FILE_ID}`
- [ ] Không có file bị SKIP trong output

---

## Eval 2: Edge case

**Tên scenario**: Input có 1 URL hợp lệ và 1 URL không nhận dạng được

**User input**:
```
/renaming-drive-files

https://docs.google.com/spreadsheets/d/1dGK78jY5FDm3-JACH3CgJbBer1o91WKL2Zkmvfac728/edit
SEONGON | BTMH - Plan nội bộ | SEO 2026

https://dropbox.com/sh/abc123/somefile
SEONGON | BTMH - Báo cáo | SEO 2026
```

**Expected behavior**:
1. Claude triggers skill `renaming-drive-files`
2. Nhận dạng URL Dropbox không phải Google Drive
3. Đánh dấu URL Dropbox là SKIP với lý do "không nhận dạng được Google Drive URL"
4. Preview table chỉ hiển thị 1 file hợp lệ + note về file bị skip
5. Sau confirm, chỉ rename 1 file hợp lệ
6. Output bảng 1 hàng "Đã đổi tên" + phần "File bị skip" liệt kê URL Dropbox với lý do

**Pass criteria**:
- [ ] Skill detect được URL không phải Google Drive (không cứ rename)
- [ ] File Dropbox xuất hiện trong "File bị skip" với lý do rõ ràng
- [ ] File Google Drive hợp lệ vẫn được rename thành công
- [ ] User có thể thấy file nào bị bỏ qua và tại sao

---

## Eval 3: Anti-pattern (skill nên từ chối / redirect)

**Tên scenario**: User chỉ paste link mà không cung cấp tên mới

**User input**:
```
/renaming-drive-files

https://docs.google.com/spreadsheets/d/1dGK78jY5FDm3-JACH3CgJbBer1o91WKL2Zkmvfac728/edit
https://docs.google.com/spreadsheets/d/1DPM5BerK_gmm1AyTNWQ3ZM4S_2UEnq_vEZdjKBwU11s/edit
```

**Expected behavior**:
1. Claude triggers skill `renaming-drive-files`
2. Detect: không có tên mới tương ứng với các URL
3. KHÔNG tự đặt tên hay đoán tên mới
4. KHÔNG chạy lệnh rename
5. Hỏi user cung cấp tên mới: "Vui lòng cung cấp tên mới cho từng file. Format: [URL] rồi [tên mới] trên dòng tiếp theo."

**Pass criteria**:
- [ ] Skill KHÔNG rename bất kỳ file nào
- [ ] Có thông báo rõ rằng thiếu tên mới
- [ ] User biết cần cung cấp thêm gì để re-run
- [ ] Không hallucinate tên file hay tự đặt tên theo format nào đó

---

## Cách chạy evals

1. Mở phiên Claude Code mới ở repo này
2. Copy User input từng scenario, paste vào Claude Code
3. So Claude's response với Expected behavior
4. Tick Pass criteria items

**Notes**:
- Chạy ở fresh context để test triggering thực tế
- Eval 1 cần gdrive authenticated và có quyền edit file thật
- Eval 2 và 3 không cần gdrive — test logic parsing và guardrail

## Eval results log

| Date | Skill version | Eval 1 | Eval 2 | Eval 3 | Notes |
|---|---|---|---|---|---|
| 2026-05-19 | v1.0 | - | - | - | Initial version |
