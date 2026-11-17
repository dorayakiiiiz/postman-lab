# Setup Guide

## 1. Yêu cầu tiên quyết

- **Node.js** (v18 trở lên): Kiểm tra bằng lệnh `node -v` trong Terminal. Nếu chưa có, tải bản LTS tại https://nodejs.org/.
- **Postman Desktop**: Tải tại https://www.postman.com/downloads/.

---

## 2. Khởi chạy eShop SUT Backend (`http://localhost:3000`)

Mở Terminal tại thư mục gốc của repo (`postman-lab/`) và chạy:
```bash
cd sut/backend
npm install
npm start
```

**Kiểm tra server đã sống:** Mở trình duyệt truy cập `http://localhost:3000/api/categories`. Nếu thấy trả về mảng JSON 3 danh mục (`Điện thoại`, `Laptop`, `Phụ kiện`) là thành công.

> **Mẹo reset dữ liệu sạch:** Backend sẽ khóa tạm thời tài khoản `test@eshop.com` trong 3 phút nếu đăng nhập sai liên tiếp 2 lần (`403 Tài khoản đã bị khóa`). Khi chạy lại test nhiều lần, chỉ cần dừng server (`Ctrl + C`) và chạy lại `npm start` để khởi tạo lại database sạch.

---

## 3. Cấu hình Postman

1. Mở **Postman Desktop**.
2. Chọn **Import** (góc trên bên trái) → chọn file `postman/eShop_Lab_Env.postman_environment.json`.
3. Ở góc trên bên phải của Postman, chuyển dropdown Environment từ *No Environment* sang **`eShop_Lab_Env`** (chứa sẵn `baseUrl = http://localhost:3000`).

---

## 4. Cài đặt Newman CLI (Bắt buộc cho Phần 2)

Mở Terminal và cài đặt `newman` cùng bộ xuất báo cáo HTML (chỉ cần cài 1 lần):

```bash
npm install -g newman newman-reporter-htmlextra
```

Kiểm tra cài đặt thành công:
```bash
newman -v
```
