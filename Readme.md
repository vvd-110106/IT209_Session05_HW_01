# Báo Cáo Thực Hành: Khôi Phục Commit Đã Mất Bằng Git Reflog

## 1. Giới thiệu & Bối cảnh

Trong quá trình làm việc, khi thực hiện lệnh `git reset --hard HEAD~1`, commit gần nhất chứa các thay đổi quan trọng bị xóa khỏi lịch sử làm việc hiện tại (`git log`). Báo cáo này hướng dẫn chi tiết cách sử dụng cơ chế `git reflog` để truy vết và khôi phục lại commit đã bị xóa mà không cần viết lại mã nguồn.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo repository thử nghiệm

Khởi tạo một repository Git mới và tạo commit ban đầu để có nền tảng làm việc:

```bash
git init
git config user.email "hocvien@example.com"
git config user.name "Hoc Vien"

echo "# Du an thu nghiem" > README.md
git add .
git commit -m "Initial commit"
```

**Output:**
```
Initialized empty Git repository in .../reflog_demo/.git/
[master (root-commit) f821966] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

---

### Bước 2: Tạo file tính năng và commit quan trọng

Tạo file `feature.txt` với nội dung quan trọng và thực hiện commit:

```bash
echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
```

**Output:**
```
[master abddcc7] Them tinh nang quan trong
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

Kiểm tra lịch sử commit lúc này (**2 commit** hiển thị bình thường):

```bash
git log --oneline
```

```
abddcc7 Them tinh nang quan trong
f821966 Initial commit
```

---

### Bước 3: Giả lập sự cố — Chạy `git reset --hard HEAD~1`

Đây là thao tác gây ra lỗi: lùi HEAD về 1 commit và xóa sạch working directory:

```bash
git reset --hard HEAD~1
```

**Output:**
```
HEAD is now at f821966 Initial commit
```

Kiểm tra lại `git log` — commit quan trọng **đã biến mất**:

```bash
git log --oneline
```

```
f821966 Initial commit
```

Kiểm tra file `feature.txt` — file **không còn tồn tại**:

```
feature.txt KHONG TON TAI!
```

>  **Vấn đề:** Commit "Them tinh nang quan trong" (hash `abddcc7`) và toàn bộ nội dung file `feature.txt` đã biến mất khỏi `git log` và thư mục làm việc.

---

### Bước 4: Tra cứu Git Reflog để tìm lại commit đã mất

Git Reflog là nhật ký lưu toàn bộ lịch sử di chuyển của `HEAD`, kể cả những hành động không còn trong `git log`. Chạy lệnh:

```bash
git reflog
```

**Output thực tế:**
```
f821966 HEAD@{0}: reset: moving to HEAD~1
abddcc7 HEAD@{1}: commit: Them tinh nang quan trong
f821966 HEAD@{2}: commit (initial): Initial commit
```

**Phân tích kết quả Reflog:**

| Entry | Hash | Hành động | Ý nghĩa |
|-------|------|-----------|---------|
| `HEAD@{0}` | `f821966` | `reset: moving to HEAD~1` | Lệnh reset vừa chạy, HEAD về Initial commit |
| `HEAD@{1}` | `abddcc7` | `commit: Them tinh nang quan trong` |  Đây là commit cần khôi phục! |
| `HEAD@{2}` | `f821966` | `commit (initial): Initial commit` | Commit khởi tạo |

> 💡 **Kết luận:** Hash commit cần khôi phục là **`abddcc7`** (dòng `HEAD@{1}`).

---

### Bước 5: Khôi phục commit đã mất bằng `git reset --hard`

Sử dụng hash tìm được từ reflog để đưa HEAD trở về đúng trạng thái cũ:

```bash
git reset --hard abddcc7
```

**Output:**
```
HEAD is now at abddcc7 Them tinh nang quan trong
```

---

### Bước 6: Kiểm tra kết quả khôi phục

**Kiểm tra `git log`** — commit đã được khôi phục hoàn toàn:

```bash
git log --oneline
```

```
abddcc7 Them tinh nang quan trong
f821966 Initial commit
```

**Kiểm tra nội dung file `feature.txt`** — dữ liệu đã trở về:

```bash
cat feature.txt
```

```
Day la tinh nang quan trong
```

**Kiểm tra `git reflog` cuối cùng** — ghi lại toàn bộ quá trình:

```bash
git reflog
```

```
abddcc7 HEAD@{0}: reset: moving to abddcc7
f821966 HEAD@{1}: reset: moving to HEAD~1
abddcc7 HEAD@{2}: commit: Them tinh nang quan trong
f821966 HEAD@{3}: commit (initial): Initial commit
```

 **Khôi phục thành công!**

---

## 3. Tổng kết

### Các lệnh đã sử dụng

| Lệnh | Mục đích |
|------|---------|
| `git reset --hard HEAD~1` | Lùi HEAD về 1 commit, xóa thay đổi (gây ra sự cố) |
| `git log --oneline` | Xem lịch sử commit ngắn gọn |
| `git reflog` | Xem toàn bộ nhật ký di chuyển HEAD (kể cả commit đã mất) |
| `git reset --hard <hash>` | Khôi phục HEAD về đúng commit theo hash |

### Bài học rút ra

- `git reset --hard` **không xóa dữ liệu vĩnh viễn** — Git vẫn lưu trữ object trong `.git/` trong một thời gian (mặc định 90 ngày).
- **Git Reflog** là "bảo hiểm" cuối cùng — nó ghi lại mọi di chuyển của HEAD, kể cả những thao tác nguy hiểm như reset, rebase.
- Không bao giờ cần viết lại code thủ công khi dùng đúng công cụ của Git.