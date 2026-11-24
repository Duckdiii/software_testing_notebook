# Security Testing: OWASP Top 10 (2021 & 2025), Burp Suite & OWASP ZAP DAST

> **Nguồn tham khảo:** OWASP Top 10 (2021 & 2025), OWASP API Security Top 10 (2023), *Full Stack Testing* (Chương 5 — *Security Testing*, tr. 164–219), OWASP ZAP GitHub Actions Docs (`zaproxy.org`).

*(Lợi thế của bạn: Đã thực hành **Burp Suite**, **OWASP WebGoat** và **OWASP Juice Shop** — đây là điểm khác biệt rất lớn của bạn so với ứng viên QA thông thường!)*

---

## 1. Tư duy Security theo góc nhìn QA/SDET (*Defense in Depth* & *Shift-Left*)

Theo Chương 5 *Full Stack Testing*, bảo mật không thể là bước kiểm tra duy nhất ở cuối dự án trước ngày Go-Live. QA đóng vai trò kết hợp **2 lớp kiểm tra**:
1. **Manual Security / Business Logic Testing (Burp Suite):** Tìm lỗi phân quyền chéo, sửa giá/số lượng, vượt qua các bước quy trình mà máy quét tự động **không bao giờ hiểu được**.
2. **Automated DAST trong CI/CD (OWASP ZAP):** Tự động quét phát hiện lỗi cấu hình (thiếu HTTP Security Headers, rò rỉ stack trace, cookie thiếu `HttpOnly`/`Secure`, Injection cơ bản) mỗi khi deploy lên Staging.

---

## 2. OWASP Top 10 (Cập nhật 2025 vs. 2021) theo Góc nhìn Tester

| Mã OWASP 2025 | Đối chiếu 2021 & Thay đổi chính | Ý nghĩa & Checklist Kiểm thử thực chiến cho QA |
| :--- | :--- | :--- |
| **A01:2025 – Broken Access Control** | Giữ #1; **gộp thêm SSRF** (từ A10:2021). | - **IDOR / BOLA:** Đổi `orderId=101` thành `102` xem có đọc/hủy được đơn của người khác không?<br>- **Privilege Escalation:** Dùng token `CUSTOMER` gọi API `ADMIN`.<br>- **SSRF:** Các trường nhập URL (ảnh đại diện, webhook) thử truyền `http://127.0.0.1:8080/actuator` hoặc `http://169.254.169.254/latest/meta-data/`. |
| **A02:2025 – Security Misconfiguration** | Tăng vọt từ #5 (2021) lên **#2**. | - Mở nhầm `/swagger-ui.html`, `/actuator/env`, `/h2-console`, directory listing trên Staging/Prod.<br>- Thiếu Security Headers (`Content-Security-Policy`, `X-Frame-Options`, `Strict-Transport-Security`, `X-Content-Type-Options`).<br>- CORS cấu hình lỏng lẻo `Access-Control-Allow-Origin: *` kèm `Allow-Credentials: true`. |
| **A03:2025 – Software Supply Chain Failures** | **Mục mới** mở rộng từ A06:2021 (*Vulnerable Components*). | - Dùng **SCA** (`OWASP Dependency-Check`, `Snyk`, `Trivy`, `npm audit`) trong CI để phát hiện thư viện Maven/npm chứa CVE nghiêm trọng (Log4Shell, Jackson cũ...). |
| **A04:2025 – Cryptographic Failures** | Tương đương A02:2021. | - Kiểm tra password/token có bị gửi rõ trên HTTP không; cookie thiếu cờ `Secure`, `HttpOnly`, `SameSite`; JWT dùng secret yếu hoặc lộ thông tin nhạy cảm trong Payload Base64; log server có in password/số thẻ không? |
| **A05:2025 – Injection** | Tương đương A03:2021 (SQLi, XSS, Command Injection). | - Thử `' OR '1'='1`, `<script>alert(1)</script>`, `"><img src=x onerror=alert(1)>`, CSV Formula Injection (`=CMD|' /C calc'!A0`) vào các form tìm kiếm, voucher, tên khách hàng, file CSV export! |
| **A06:2025 – Insecure Design** | Tương đương A04:2021. | - **Lỗi nghiệp vụ (máy quét tự động KHÔNG thấy!):** Sửa giá `price: 0` hoặc `quantity: -5` ở body JSON checkout; áp 1 mã voucher dùng 1 lượt bằng 2 request song song; bỏ qua bước thanh toán VNPAY nhảy thẳng gọi endpoint xác nhận hoàn thành! |
| **A07:2025 – Authentication Failures** | Tương đương A07:2021. | - Brute-force mật khẩu/OTP có bị **Rate Limiting** chặn không? Reset password token có hết hạn và vô hiệu sau 1 lần dùng không? JWT `alg=none` hoặc token cũ sau khi đổi mật khẩu còn dùng được không? |
| **A08:2025 – Software or Data Integrity Failures** | Tương đương A08:2021. | - Kiểu deserialize Java không tin cậy, webhook thanh toán VNPAY/MoMo **không kiểm tra chữ ký HMAC** (attacker tự giả lập webhook báo `PAID`!). |
| **A09:2025 – Security Logging & Alerting Failures** | Tương đương A09:2021. | - Thực hiện 10 lần login sai hoặc 403 IDOR → kiểm tra hệ thống log có ghi nhận IP/UserId/Timestamp (nhưng **KHÔNG** ghi mật khẩu rõ!) để truy vết không. |
| **A10:2025 – Mishandling of Exceptional Conditions** | **Mục mới** thay chỗ SSRF. | - Gửi JSON gãy, số quá giới hạn `int`, giá trị `null` → kiểm tra server có lộ **Java Stack Trace**, tên bảng SQL, hay bị rớt trạng thái lỗi giữa chừng (*Fail Open*) thay vì đóng từ chối an toàn (*Fail Closed*) không. |

---

## 3. Quy trình dùng Burp Suite Hiệu quả cho QA (Không cần biết hack sâu!)

1. **Intercept & Proxy History:** Duyệt qua 1 lượt luồng Checkout/CRM bình thường, sau đó mở tab **HTTP History** rà soát: có endpoint nào lộ `password`, token truyền trên URL Query String (`?token=...`), hoặc có API ngầm không?
2. **Repeater (Vũ khí số 1 cho QA):** Gửi các request nhạy cảm (`POST /checkout`, `GET /orders/101`, `PATCH /deals/205`) sang **Repeater**:
   - Đổi `userId` / `orderId` sang ID của user khác → bắt **IDOR (A01)**.
   - Thay `Authorization` header bằng token user quyền thấp → bắt **Privilege Escalation (A01)**.
   - Sửa `price = 1`, `quantity = -1`, xóa chữ ký webhook → bắt **Insecure Design (A06)**.
3. **Intruder / Turbo Intruder:** Thử brute-force mã OTP 6 số, kiểm tra Rate Limit login (A07), hoặc bắn **20 request song song cùng mili-giây (*Single-packet Attack*)** để săn Race Condition voucher/tồn kho!
4. **JWT Editor Extension:** Giải mã nhanh JWT, thử đổi `role: ADMIN` với `alg: none` hoặc ký lại sai signature.

---

## 4. Tích hợp OWASP ZAP (DAST) vào CI/CD Pipeline

**OWASP ZAP** cung cấp 2 Docker image / GitHub Action chính để tích hợp tự động hóa vào pipeline:

| Action | Cách hoạt động | Dùng khi nào & Ở môi trường nào? |
| :--- | :--- | :--- |
| **`zaproxy/action-baseline`** | Chạy Spider (thu thập link) + **Passive Scan** (chỉ đọc response, **không** bắn payload tấn công phá hoại) trong 1–3 phút. | An toàn tuyệt đối; chạy được trên **mỗi PR / Deploy Staging** để bắt thiếu Security Header, Cookie flags, rò rỉ thông tin (A02, A04, A10). |
| **`zaproxy/action-api-scan`** | Đọc file `openapi.json` / `swagger.yaml` rồi chạy **Active Scan / Fuzzing** (SQLi, XSS, Path Traversal...) vào từng tham số API. | **Chỉ chạy trên môi trường Ephemeral / Staging test** (tuyệt đối không chạy Active Scan lên Production vì sẽ làm rác DB hoặc kích hoạt WAF!). Phải truyền JWT Token qua option `-z` để quét được các endpoint sau đăng nhập! |

### Cấu hình mẫu GitHub Actions (`zaproxy/action-baseline` + Rules lọc nhiễu)
```yaml
- name: OWASP ZAP Baseline Scan
  uses: zaproxy/action-baseline@v0.14.0
  with:
    target: 'http://localhost:8080'
    rules_file_name: '.zap/rules.tsv' # Tùy biến IGNORE/WARN/FAIL tránh false positive chặn nhầm PR
    cmd_options: '-a'
```
File `.zap/rules.tsv` (quy ước `IGNORE` / `WARN` / `FAIL`):
```tsv
# RuleID	Action	Description
10096	IGNORE	Timestamp Disclosure - Unix (false positive phổ biến trên API JSON)
10038	WARN	Content Security Policy (CSP) Header Not Set
40012	FAIL	Cross Site Scripting (Reflected)
```
