# Báo Cáo Thực Hành: Khôi Phục Commit Đã Mất Bằng Git Reflog

## 1. Giới thiệu & Bối cảnh
Trong quá trình làm việc, khi thực hiện lệnh `git reset --hard HEAD~1`, commit gần nhất chứa các thay đổi quan trọng bị xóa khỏi lịch sử làm việc hiện tại (`git log`). Báo cáo này hướng dẫn chi tiết cách sử dụng cơ chế `git reflog` để truy vết và khôi phục lại commit đã bị xóa mà không cần viết lại mã nguồn.

---

## 2. Các bước thực hiện

### Bước 1: Giả lập sự cố mất commit
1. Khởi tạo repository và tạo file thử nghiệm:
   ```bash
   echo "Day la tinh nang quan trong" > feature.txt
   git add .
   git commit -m "Them tinh nang quan trong"