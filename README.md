# Hands-on Lab: Kiểm thử API với Postman & Newman CLI

> **Hệ thống kiểm thử (SUT):** eShop Backend (`http://localhost:3000`)  
> **Thời lượng:** ~15–20 phút

---

## Mục tiêu bài Lab

- Thiết lập biến môi trường (`{{baseUrl}}`) và biến động (`{{$timestamp}}`) trong Postman.
- Viết **Checkpoint (Assertion)** bằng Chai.js để thực hiện **Negative Testing** và phát hiện lỗi thiếu kiểm tra đầu vào.
- Áp dụng **Data-Driven Testing (DDT)** với dữ liệu CSV và chạy tự động qua dòng lệnh **Newman CLI**.

---

## Cấu trúc thư mục

```text
postman-lab/
├── README.md                                  # Đề bài & quy định nộp bài
├── SETUP.md                                   # Hướng dẫn cài đặt môi trường & chạy SUT
├── ANSWERS_TEMPLATE.md                        # Mẫu file Markdown trả lời câu hỏi nộp bài
├── data/
│   └── login_test_data.csv                    # Dữ liệu DDT cho Phần 2 (4 test cases)
├── postman/
│   └── eShop_Lab_Env.postman_environment.json # Environment mẫu để import vào Postman
└── sut/
    ├── api_specification.md                   # Đặc tả API của eShop SUT
    └── backend/                               # Mã nguồn Backend Express + SQLite
```

---

## Bắt đầu nhanh

1. Hoàn tất cài đặt, import Environment và khởi chạy SUT theo **[SETUP.md](SETUP.md)**.
2. Tạo Collection mới tên **`eShop_HandsOnLab`** trong Postman để bắt đầu làm bài.

---

## Phần 1: Negative Testing & Checkpoints (`POST /api/register`)

**Bối cảnh:** Theo đặc tả `sut/api_specification.md`, API đăng ký `POST /api/register` yêu cầu 3 trường `name`, `email`, `password`. Hãy kiểm tra xem hệ thống có chặn lại khi **thiếu `password`** hay không.

### Yêu cầu
1. Tạo request **`1_Register_MissingPassword`** (`POST {{baseUrl}}/api/register`).
2. Nhập Body (`raw → JSON`) **cố tình bỏ trường `password`** (dùng `{{$timestamp}}` để email không bị trùng):
   ```json
   {
     "name": "Sinh Vien Test",
     "email": "lab_{{$timestamp}}@eshop.com"
   }
   ```
3. Tự viết **3 Checkpoints** trong tab **`Scripts` → `After response`**:
   - **CP1:** Mã trạng thái HTTP phải là `400`.
   - **CP2:** Phản hồi JSON phải có thông báo lỗi chứa từ khóa `"password"` (không phân biệt hoa/thường).
   - **CP3:** Thời gian phản hồi (`responseTime`) phải nhỏ hơn `350` ms.
4. Bấm **Send**, quan sát **Test Results** và **chụp ảnh kết quả** (kèm mã HTTP trả về).

### Gợi ý
- `pm.test("Tên", function () { ... });`
- `pm.response.to.have.status(400);` | `const res = pm.response.json();` | `pm.expect(pm.response.responseTime).to.be.below(350);`
- Nếu checkpoint báo **FAIL (đỏ)** dù kỳ vọng của bạn hợp lý, **không sửa test cho xanh** — đó chính là bằng chứng bạn đã tìm ra bug của API.

---

## Phần 2: Data-Driven Testing & Tự động hóa với Newman CLI (`POST /api/login`)

**Bối cảnh:** Để kiểm thử nhiều kịch bản đăng nhập (đúng, sai mật khẩu, tài khoản không tồn tại, thiếu mật khẩu) mà không phải tạo nhiều request rời rạc, ta dùng **1 request duy nhất** kết hợp file `data/login_test_data.csv` và chạy tự động bằng **Newman CLI**.

> **Tài khoản mẫu trong SUT:** `test@eshop.com` / `Test1234!`

### Dữ liệu đầu vào (`data/login_test_data.csv`)

| `testCase` | `email` | `password` | `expectedStatus` | `expectedSuccess` | `expectedError` |
|---|---|---|:---:|:---:|---|
| `TC01_Valid_User` | `test@eshop.com` | `Test1234!` | `200` | `true` | *(rỗng)* |
| `TC02_Wrong_Password` | `test@eshop.com` | `WrongPass!` | `401` | `false` | `Invalid email or password` |
| `TC03_User_NotFound` | `ghost_user_99@eshop.com` | `Test1234!` | `401` | `false` | `Invalid email or password` |
| `TC04_Missing_Password` | `test@eshop.com` | *(rỗng)* | `400` | `false` | `required` |

### Yêu cầu
1. Tạo request **`2_Login_DDT`** (`POST {{baseUrl}}/api/login`) trong cùng Collection.
2. Tham số hóa Body (`raw → JSON`) bằng biến CSV:
   ```json
   {
     "email": "{{email}}",
     "password": "{{password}}"
   }
   ```
3. Tự viết **3 Checkpoints động** trong tab **`Scripts` → `After response`** sử dụng `pm.iterationData.get("...")`:
   - **CP1:** Kiểm tra HTTP status khớp với `expectedStatus`.
   - **CP2:** Nếu `expectedSuccess` là `true` → kiểm tra response có trường `token` dạng chuỗi không rỗng; ngược lại → kiểm tra thông báo lỗi chứa `expectedError`.
   - **CP3:** Thời gian phản hồi `< 350` ms.
4. **Export Collection** (định dạng **Collection v2.1**) lưu trực tiếp vào **thư mục gốc của repo** (`postman-lab/<MSSV>_eShop_Lab.postman_collection.json`).
5. Mở Terminal tại thư mục gốc repo (`postman-lab/`) và chạy lệnh **Newman CLI** để thực thi toàn bộ kịch bản kèm dữ liệu CSV và xuất báo cáo HTML:
   ```bash
   newman run <MSSV>_eShop_Lab.postman_collection.json -e postman/eShop_Lab_Env.postman_environment.json -d data/login_test_data.csv -r cli,htmlextra --reporter-htmlextra-export results/report.html
   ```
   *(Lưu ý: Nếu chạy lại nhiều lần và gặp lỗi `403 Tài khoản đã bị khóa`, hãy khởi động lại server bằng `npm start` trong `sut/backend` để reset DB).*

### Gợi ý
- Giá trị đọc từ CSV qua `pm.iterationData.get(...)` nên ép kiểu tường minh: `Number(pm.iterationData.get("expectedStatus"))` và `String(pm.iterationData.get("expectedSuccess")) === "true"`.

---

## Câu hỏi thu hoạch (Trả lời vào file Markdown)

Sao chép file [ANSWERS_TEMPLATE.md](ANSWERS_TEMPLATE.md) thành `<MSSV>_answers.md` và trả lời ngắn gọn 4 câu sau:

1. **Phân tích Bug Phần 1:** API `/api/register` đã trả về mã HTTP và nội dung gì khi thiếu `password`? Nếu đưa lên môi trường thật, lỗ hổng này gây ra rủi ro gì?
2. **Phân tích kết quả Phần 2:** Trong 4 test case của `login_test_data.csv`, test case nào bị **FAIL**? Nguyên nhân nằm ở test script của bạn hay ở code backend (`sut/backend/server.js`)?
3. **Quan sát trạng thái hệ thống:** Vì sao khi chạy lệnh Newman lần thứ 2 liên tiếp mà không reset database, `TC01_Valid_User` có thể bị trả về `403`? Điều này nói lên bài học gì khi thiết kế bộ test tự động (*Test Isolation / Cleanup*)?
4. **Mở rộng & CI/CD:** Vì sao Newman CLI lại trả về `Exit Code 1` khi có ít nhất 1 checkpoint FAIL? Ý nghĩa của cơ chế này trong pipeline CI/CD là gì?

---

## Quy định nộp bài

Nén thành tệp **`<MSSV>.zip`** gồm đầy đủ **4 thành phần** sau:

1. **File Collection đã export:** `<MSSV>_eShop_Lab.postman_collection.json` (đầy đủ 2 request và test script).
2. **File Markdown trả lời câu hỏi:** `<MSSV>_answers.md` (theo mẫu [ANSWERS_TEMPLATE.md](ANSWERS_TEMPLATE.md)).
3. **Báo cáo Newman HTML:** `results/report.html` (xuất từ cờ `--reporter-htmlextra-export`).
4. **Ảnh chụp minh chứng (2 ảnh):**
   - `part1_register.png`: Ảnh màn hình Postman sau khi Send `1_Register_MissingPassword` (thấy rõ Body, Response và tab Test Results).
   - `part2_newman.png`: Ảnh màn hình Terminal hiển thị bảng kết quả chạy lệnh `newman run ...`.
