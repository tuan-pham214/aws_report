# Đưa báo cáo lên GitHub và xuất bản bằng GitHub Pages

## 1. Tạo repository trên GitHub

1. Đăng nhập GitHub và mở <https://github.com/new>.
2. Đặt tên repository, ví dụ `aws-internship-report`.
3. Chọn **Public** nếu tài khoản GitHub Free cần xuất bản Pages công khai.
4. Không chọn tạo sẵn README, `.gitignore` hoặc license vì thư mục báo cáo đã có đầy đủ tệp.
5. Chọn **Create repository**.

## 2. Chọn GitHub Actions làm nguồn xuất bản

1. Mở repository vừa tạo.
2. Chọn **Settings → Pages**.
3. Trong **Build and deployment**, đặt **Source** thành **GitHub Actions**.

## 3. Ghi đúng danh tính người tạo commit

Mở Terminal tại thư mục báo cáo và chạy:

```bash
cd /Users/phamtuan/Downloads/fcaj-hcmut-template-main
git config user.name "TEN_HIEN_THI_GITHUB_CUA_BAN"
git config user.email "EMAIL_DA_XAC_MINH_TREN_GITHUB"
```

Có thể dùng địa chỉ `noreply` trong **GitHub → Settings → Emails** nếu không muốn công khai email thật.

## 4. Tạo commit và push bằng tài khoản của bạn

```bash
git add .
git commit -m "Publish AWS internship report"
git remote add origin https://github.com/USERNAME/TEN_REPOSITORY.git
git push -u origin main
```

Thay `USERNAME` và `TEN_REPOSITORY` bằng thông tin repository vừa tạo. Tài khoản xác thực khi chạy `git push` sẽ là người push; tên và email cấu hình ở bước 2 sẽ là tác giả commit.

## 5. Theo dõi quá trình xuất bản

1. Mở tab **Actions** và chọn workflow **Build and deploy Hugo report**.
2. Chờ cả hai bước `build` và `deploy` chuyển sang màu xanh.

Địa chỉ trang web sẽ có dạng:

```text
https://USERNAME.github.io/TEN_REPOSITORY/
```

Workflow tự nhận đúng đường dẫn repository, vì vậy không cần sửa `baseURL` khi đổi tên repository.

## 6. Cập nhật báo cáo sau này

Sau khi chỉnh sửa nội dung, chính bạn chạy:

```bash
git add .
git commit -m "Update internship report"
git push
```

GitHub Pages sẽ tự dựng lại trang web sau mỗi lần push lên nhánh `main`.
