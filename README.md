# CO3133 — Group Website · Hướng dẫn chỉnh sửa

Đây là site tĩnh (HTML/CSS thuần, không cần build) gồm 4 trang:

| File | Nội dung |
|---|---|
| `index.html` | Trang chủ: thông tin môn học, thông tin nhóm, danh sách assignment, AI disclosure chung |
| `assignment1.html` | Chi tiết Assignment 1 |
| `assignment2.html` | Chi tiết Assignment 2 |
| `assignment3.html` | Chi tiết Assignment 3 |
| `style.css` | Toàn bộ style, dùng chung cho cả 4 trang |

Mở file `.html` bằng bất kỳ trình soạn thảo nào (VS Code, Notepad++...) để sửa. Không cần cài gì thêm, mở trực tiếp file `.html` bằng trình duyệt là xem được ngay.

---

## 1. Quy tắc quan trọng: class `placeholder`

Trong file có rất nhiều chỗ được đánh dấu bằng `class="placeholder"`, ví dụ:

```html
<td class="placeholder">Hoàng Xuân Bách</td>
```

Class này khiến chữ **hiển thị mờ + nghiêng** (xem trong `style.css` dòng ~232) để nhắc "đây là dữ liệu mẫu, chưa phải dữ liệu thật". 

**Quy tắc: sau khi bạn thay nội dung bằng thông tin thật, phải xóa `class="placeholder"` đi**, nếu không chữ sẽ vẫn bị mờ dù bạn đã điền đúng thông tin.

Ví dụ trước và sau khi sửa:

```html
<!-- Trước (còn placeholder, chữ mờ) -->
<td class="placeholder">[Tên bạn]</td>

<!-- Sau (đã điền + xóa class, chữ rõ) -->
<td>Nguyễn Văn A</td>
```

Nếu ô đó có kèm class khác (vd `mono` để hiện font số MSSV), chỉ xóa phần `placeholder`, giữ lại class còn lại:

```html
<!-- Trước -->
<td class="mono placeholder">2352082</td>

<!-- Sau -->
<td class="mono">2352082</td>
```

---

## 2. Cách gắn link (GitHub, repo...) cho đúng

Một lỗi rất hay gặp là gõ cả link vào phần chữ hiển thị thay vì vào `href`. Link phải nằm trong thuộc tính `href`, còn giữa `<a>...</a>` chỉ nên là chữ hiển thị.

```html
<!-- SAI: link không hoạt động vì href="#" -->
<a href="#" class="placeholder">username[https://github.com/username]</a>

<!-- ĐÚNG -->
<a href="https://github.com/username" target="_blank" rel="noopener">username</a>
```

- `target="_blank"` → mở link ở tab mới.
- `rel="noopener"` → thêm cho an toàn, nên giữ nguyên.

Áp dụng tương tự cho nút "Group code repository" trong `index.html`:

```html
<div class="repo-callout">
  <span class="label">Group code repository</span>
  <a class="button" href="https://github.com/ten-nhom/ten-repo" target="_blank" rel="noopener">Xem repo trên GitHub</a>
</div>
```

---

## 3. Danh sách những chỗ cần điền

### `index.html`
- [ ] `Group [XX]` ở thanh menu trên cùng (`<a class="brand">`) → đổi thành số nhóm thật, vd `Group 07`.
- [ ] Bảng thành viên (`<table class="members">`): họ tên, MSSV, vai trò/đóng góp, link GitHub — nhớ xóa `placeholder` sau khi điền.
- [ ] Nút "Group code repository" → điền link repo thật.
- [ ] 3 tiêu đề `[Assignment N title]` trong phần "Assignment index" → đổi thành tên đề tài thật của từng assignment.
- [ ] Phần "AI usage disclosure" (4 mục: Tools used, Where AI was used, Where AI was NOT used, Verification statement) → điền theo thực tế nhóm đã dùng AI như thế nào.
- [ ] `Group [XX]` ở footer cuối trang.

### `assignment1.html`, `assignment2.html`, `assignment3.html` (lặp lại cấu trúc giống nhau)
- [ ] `[Assignment N Title]` ở đầu trang.
- [ ] Mục "Problem statement" → thay `[Replace with the assignment's problem statement / objectives.]`.
- [ ] Mục "Method" → thay `[Describe your model, data, and experimental setup here.]`.
- [ ] Mục "Results" → thay `[Add tables, charts, or figures summarizing your results.]` (có thể chèn bảng số liệu, ảnh biểu đồ...).
- [ ] Mục "AI disclosure" riêng cho assignment đó (Where AI was used / NOT used).
- [ ] `Group [XX]` ở footer.

---

## 4. Chèn hình ảnh / biểu đồ (nếu cần cho phần Results)

Đặt file ảnh (vd `results1.png`) cùng thư mục với các file `.html`, sau đó thêm vào chỗ cần:

```html
<img src="results1.png" alt="Kết quả training Assignment 1" style="max-width:100%; border-radius:8px; margin-top:12px;">
```

---

## 5. Thêm/bớt thành viên trong bảng

Mỗi thành viên là một dòng `<tr>...</tr>` trong `<tbody>` của `index.html`. Copy nguyên một dòng `<tr>` rồi sửa nội dung để thêm thành viên; xóa cả dòng `<tr>` để bớt thành viên.

---

## 6. Xem trước kết quả

Không cần server, chỉ cần double-click `index.html` để mở bằng trình duyệt (Chrome/Edge/Firefox) là xem được toàn bộ trang, kể cả sau khi sửa. Sửa xong, lưu file (Ctrl+S), rồi reload lại trình duyệt (F5) để thấy thay đổi.
