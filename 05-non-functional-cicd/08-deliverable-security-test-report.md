# Deliverable 3 Giai đoạn 5: Báo cáo Kiểm thử Bảo mật (Security Test Report) — TechStore

> **Mã tài liệu:** `SEC-TECHSTORE-2026-01` *(Báo cáo mẫu chuẩn Portfolio kết hợp Automated DAST & Manual Logic Pentest)*  
> **Chuẩn đối chiếu:** OWASP Top 10:2025, OWASP API Security Top 10:2023  
> **Công cụ sử dụng:** **OWASP ZAP** (Baseline + OpenAPI Active Scan) & **Burp Suite** (Proxy, Repeater, Turbo Intruder, JWT Editor)

---

## 1. Tóm tắt Điều hành (Executive Summary)

| Mức độ (Severity) | Số lỗi phát hiện (Vòng 1) | Đã khắc phục & Retest Pass (Vòng 2) | Còn mở |
| :---: | :---: | :---: | :---: |
| **Critical** | `1` (`SEC-01` — IDOR xem & hủy đơn người khác) | `1` | `0` |
| **High** | `2` (`SEC-02` — Race Condition Voucher; `SEC-03` — Sửa giá/số lượng âm) | `2` | `0` |
| **Medium** | `2` (`SEC-04` — Thiếu CSP/HSTS Headers; `SEC-05` — Lộ Stack Trace `A10:2025`) | `2` | `0` |
| **Low / Informational** | `1` (`SEC-06` — Cookie thiếu cờ `SameSite=Strict` & `HttpOnly`) | `1` | `0` |
| **TỔNG CỘNG** | **6 Findings** | **6 / 6 Fixed & Verified** | **0** |

- **Kết luận Bảo mật:** Tại Vòng 1, hệ thống **KHÔNG ĐỦ ĐIỀU KIỆN GO-LIVE** do tồn tại 1 lỗ hổng `Critical` (IDOR lộ PII khách hàng) và 2 lỗ hổng `High` (thất thoát tài chính trực tiếp). Sau khi vá và kiểm tra lại (*Confirmation Testing*) bằng cả Burp Repeater lẫn bộ REST Assured Security Regression Tests, toàn bộ **6/6 lỗ hổng đã được đóng hoàn toàn**.

---

## 2. Phạm vi & Phương pháp Kiểm thử (Scope & Methodology)

- **Target:** TechStore Web (`http://localhost:3000`) & REST API (`http://localhost:8080/api/v1/**`).
- **Tài khoản kiểm thử:** `buyer_a@techstore.vn` (`CUSTOMER`), `buyer_b@techstore.vn` (`CUSTOMER`), `admin@techstore.vn` (`ADMIN`), và trạng thái Unauthenticated.
- **Kết hợp 2 giai đoạn:**
  1. **Automated DAST (`OWASP ZAP`):** Chạy `zap-baseline.py` + `zap-api-scan.py` nạp OpenAPI `/v3/api-docs` kèm JWT Bearer token để quét Injection, Misconfiguration, Security Headers và Error Handling.
  2. **Manual Security & Logic Testing (`Burp Suite`):** Kiểm tra sâu những lỗ hổng mà máy quét DAST không thấy được (Authorization IDOR/BOLA, Business Logic Bypass, Race Condition với Turbo Intruder, JWT `alg=none`).

---

## 3. Chi tiết Các Lỗ hổng Phát hiện & Bằng chứng Khắc phục (Findings Detail)

### `[SEC-01]` Critical — Broken Object Level Authorization (IDOR/BOLA) trên API Đơn hàng
- **Phân loại:** **OWASP `A01:2025` (Broken Access Control)** / **API1:2023 (BOLA)** · **CVSS v3.1:** `8.6 (High/Critical)`
- **Phương pháp phát hiện:** Thủ công bằng **Burp Suite Repeater** *(OWASP ZAP không phát hiện được!)*.
- **Mô tả & Bằng chứng tái hiện (PoC):**
  - Đăng nhập bằng `buyer_a` (ID `101`), gửi request xem/hủy đơn hàng của `buyer_b` (ID `102`, Order ID `5042`):
    ```http
    GET /api/v1/orders/5042 HTTP/1.1
    Host: localhost:8080
    Authorization: Bearer <JWT_OF_BUYER_A>
    ```
  - **Actual Response (Vòng 1):** Server trả về **`200 OK`** kèm toàn bộ họ tên, số điện thoại, địa chỉ giao hàng của `buyer_b`! Thậm chí `POST /api/v1/orders/5042/cancel` cũng thành công!
- **Tác động (Impact):** Attacker chỉ cần quét tuần tự `orderId` (`1..100000`) để thu thập 100% thông tin cá nhân (PII) khách hàng TechStore hoặc phá hủy hàng loạt đơn hàng đang chờ xử lý.
- **Khắc phục & Retest (Vòng 2):**
  - Backend bổ sung kiểm tra `if (!order.getUserId().equals(currentUser.getId()) && !currentUser.isAdmin()) throw new AccessDeniedException(...)`.
  - **Retest:** Gửi lại PoC trên trả về **`403 Forbidden`**; đã bổ sung vào `SecurityJwtApiTest` (REST Assured) để chống tái phát trên CI.

---

### `[SEC-02]` High — Race Condition cho phép dùng vượt giới hạn lượt Voucher (`usageLimit = 1`)
- **Phân loại:** **OWASP `A06:2025` (Insecure Design — Concurrency Business Logic Flaw)** · **CVSS:** `7.5`
- **Phương pháp phát hiện:** **Burp Suite Repeater (`Send group in parallel`)** + **Testcontainers Concurrency IT**.
- **Mô tả & PoC:** Khi mã `VIP100K` chỉ còn `usageLimit = 1`, gửi đồng thời 10 request `POST /api/v1/checkout` cùng lúc → Server đọc `usageLimit = 1` vào RAM cho cả 10 luồng trước khi commit → **4 đơn được áp giảm giá thành công**, `usage_limit` trong PostgreSQL tụt xuống `-3`!
- **Khắc phục & Retest:** Chuyển sang câu SQL atomic `UPDATE vouchers SET usage_limit = usage_limit - 1 WHERE code = :code AND usage_limit > 0` + kiểm tra `updatedRows == 1` trong `@Transactional`. Retest 20 request song song: **đúng 1 request `201`, 19 request `409 Conflict`**.

---

### `[SEC-03]` High — Tin tưởng giá tiền / số lượng âm từ Client gửi lên (`Price & Quantity Tampering`)
- **Phân loại:** **OWASP `A06:2025` (Insecure Design)** & **`A05:2025` (Input Validation)** · **CVSS:** `7.8`
- **Mô tả & PoC:** Trong payload `POST /api/v1/checkout` ở bản nháp cũ, gửi `{"productId": 1, "quantity": -2, "clientPrice": 1000}`:
  - Một endpoint phụ tính tổng giỏ hàng tạm (`POST /api/v1/cart/preview`) không chặn `quantity <= 0` làm tổng tiền bị tính âm (`-2.000.000đ`).
- **Khắc phục & Retest:** Backend xóa bỏ hoàn toàn việc đọc giá từ client DTO (luôn lấy `product.getPrice()` từ DB), gắn `@Min(1)` trên DTO + check `if (line.quantity() <= 0)` ở Service. Retest trả **`400 Bad Request`**.

---

### `[SEC-04]` Medium — Thiếu Các HTTP Security Headers Quan trọng
- **Phân loại:** **OWASP `A02:2025` (Security Misconfiguration)**
- **Phương pháp phát hiện:** **OWASP ZAP Baseline Scan** (`Rule 10038`, `10035`, `10021`).
- **Mô tả:** Response từ `http://localhost:3000` và `8080` thiếu các header phòng vệ trình duyệt: `Content-Security-Policy`, `Strict-Transport-Security (HSTS)`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`.
- **Khắc phục & Retest:** Cấu hình `SecurityFilterChain` trong Spring Security (`.headers(h -> h.contentSecurityPolicy(...).frameOptions(DENY))`) và Nginx frontend. ZAP Baseline Scan Vòng 2 báo **0 cảnh báo thiếu header**.

---

### `[SEC-05]` Medium — Rò rỉ Java Stack Trace & Tên Bảng SQL khi Gửi Payload Gãy
- **Phân loại:** **OWASP `A10:2025` (Mishandling of Exceptional Conditions)**
- **Phương pháp phát hiện:** **OWASP ZAP API Scan** + Fuzzing JSON gãy (`{"items": "invalid_string"}`).
- **Mô tả & PoC:** API trả về HTTP `500` kèm toàn bộ `trace` dài 60 dòng tiết lộ phiên bản Spring Boot, Hibernate dialect, tên package `com.techstore.checkout` và tên bảng DB.
- **Khắc phục & Retest:** Cấu hình `server.error.include-stacktrace=never` và chuẩn hóa `@RestControllerAdvice` (`GlobalExceptionHandler`) trả về RFC 7807 JSON gọn sạch (`status`, `errorCode`, `message`) không lộ thông tin nội bộ.

---

### `[SEC-06]` Low — Cookie Refresh Token thiếu cờ `SameSite=Strict`
- **Phân loại:** **OWASP `A04:2025` (Cryptographic Failures / Session Hygiene)**
- **Phương pháp phát hiện:** **OWASP ZAP Passive Scan (`Rule 10054`)**.
- **Khắc phục & Retest:** Thiết lập `ResponseCookie.from("refresh_token", token).httpOnly(true).secure(true).sameSite("Strict")` → Đạt chuẩn, phòng ngừa CSRF.

---

## 4. Bài học Đúc kết: Giá trị khi Kết hợp ZAP DAST + Burp Suite Thủ công

| Nhóm lỗi | OWASP ZAP (Tự động trên CI) | Burp Suite (Manual Pentest) |
| :--- | :---: | :---: |
| Thiếu Security Headers, Cookie flags, Stack Trace (`SEC-04`, `SEC-05`, `SEC-06`) | ✅ **Bắt 100% nhanh trong 2 phút** | Có thấy nhưng tốn công soi từng header |
| Lỗ hổng Logic, IDOR/BOLA, Sửa giá, Race Condition (`SEC-01`, `SEC-02`, `SEC-03`) | ❌ **Mù hoàn toàn!** | ✅ **Bắt chính xác qua Repeater / Turbo Intruder** |

→ **Kết luận cho CI/CD:** Đưa `OWASP ZAP Baseline` + `Trivy SCA` chạy tự động trên Pipeline để dọn sạch các lỗi cấu hình & thư viện; đồng thời biến các PoC tìm được bằng Burp Suite (`SEC-01`, `SEC-02`, `SEC-03`) thành **Regression Security Tests tự động trong REST Assured / Testcontainers**!
