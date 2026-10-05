# Báo cáo Bài 2: Tái cấu trúc lịch sử Commit bằng Interactive Rebase

## 1. Giới thiệu

Tài liệu này báo cáo quy trình tái cấu trúc lịch sử commit cục bộ bằng công cụ **Interactive Rebase** (`git rebase -i`). Mụch đích nhằm dọn dẹp các commit thử nghiệm vụn vặt, gộp nhiều commit nhỏ thành một commit có ý nghĩa và xóa bỏ các commit chứa file rác trước khi push lên remote repository.

---

## 2. Quy trình thực hiện

### Bước 1: Khởi tạo dữ liệu thử nghiệm

Tạo lần lượt 4 commit trên nhánh làm việc hiện tại để giả lập lịch sử commit vụn vặt:

```bash
# Commit 1: Tạo auth.js
echo "// Module Auth" > auth.js
git add auth.js
git commit -m "feat: khoi tao module auth"

# Commit 2: Sửa auth.js
echo "// Fix typo in Auth" >> auth.js
git add auth.js
git commit -m "fix typo"

# Commit 3: Thêm hàm cho auth.js
echo "// Utility functions" >> auth.js
git add auth.js
git commit -m "adds utility functions"

# Commit 4: Tạo file rác temp.txt
echo "temp data" > temp.txt
git add temp.txt
git commit -m "add temp file for debug"
```

Kiểm tra lịch sử commit ban đầu bằng `git log --oneline`:

```text
a4b3c2d (HEAD -> main) add temp file for debug
f3e2d1c adds utility functions
e2d1c0b fix typo
d1c0b9a feat: khoi tao module auth
```

---

### Bước 2: Thực hiện Interactive Rebase

Mở giao diện tương tác Interactive Rebase lùi lại 4 commit tính từ `HEAD`:

```bash
git rebase -i HEAD~4
```

### Bước 3: Cấu hình lệnh Rebase trong trình soạn thảo

Trong giao diện trình soạn thảo mở ra, thay đổi các từ khóa ở đầu mỗi dòng theo yêu cầu:
- **Keep & Squash**: Giữ commit đầu tiên (`pick`), gộp 2 commit tiếp theo vào commit đầu (`squash` / `s`).
- **Drop**: Xóa bỏ hoàn toàn commit chứa file rác `temp.txt` (`drop` / `d`).

**Cấu hình thực tế:**

```text
pick d1c0b9a feat: khoi tao module auth
squash e2d1c0b fix typo
squash f3e2d1c adds utility functions
drop a4b3c2d add temp file for debug

# Rebase 8f9e0d1..a4b3c2d onto 8f9e0d1 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# s, squash <commit> = use commit, but meld into previous commit
# d, drop <commit> = remove commit
```

Lưu và đóng trình soạn thảo lại.

---

### Bước 4: Đổi thông điệp commit (Reword)

Ngay sau khi lưu cấu hình, Git sẽ mở tiếp trình soạn thảo để nhập thông điệp mới cho commit vừa gộp. 

Xóa các dòng thông điệp cũ và sửa lại thành:
```text
feat: hoan thien module authentication
```

Lưu và hoàn tất quá trình rebase.

---

## 3. Kiểm tra kết quả

1. **Xem lại lịch sử commit sau khi Rebase:**
   ```bash
   git log --oneline
   ```
   *Kết quả mong đợi:* Lịch sử commit trở nên sạch sẽ, chỉ còn duy nhất 1 commit mới chứa toàn bộ thay đổi hoàn chỉnh.
   ```text
   b7a6c5d (HEAD -> main) feat: hoan thien module authentication
   8f9e0d1 Initial commit
   ```

2. **Kiểm tra sự tồn tại của file rác:**
   ```bash
   ls -la
   ```
   *Kết quả mong đợi:* Tệp `temp.txt` đã bị xóa bỏ hoàn toàn khỏi working directory, chỉ còn lại `auth.js`.

---

## 4. Tổng kết bài học
- **Interactive Rebase (`git rebase -i`)** là công cụ mạnh mẽ để dọn dẹp lịch sử commit cục bộ (local) trước khi chia sẻ code với nhóm.
- **`squash` (s):** Giúp gom nhiều thay đổi nhỏ lẻ thành một tính năng hoàn chỉnh, giúp cây lịch sử (git tree) rõ ràng, dễ theo dõi.
- **`drop` (d):** Giúp loại bỏ hoàn toàn các thay đổi không mong muốn hoặc file thử nghiệm/debug mà không cần tạo thêm commit revert.