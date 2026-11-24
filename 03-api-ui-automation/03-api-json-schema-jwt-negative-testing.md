# Kỹ thuật API Testing Cốt lõi: JSON Schema Validation, Auth (JWT) & Negative Testing

> **Nguồn tham khảo:** JSON Schema Draft 2020-12 / Draft-07, RFC 7519 (JWT), RFC 7807/9457 (Problem Details), OWASP API Security Top 10 (2023), *Agile Testing* (Ch. 10 — API Testing, tr. 205–207).

Một API test chỉ kiểm tra `statusCode == 200` là một test **cực kỳ yếu**: API có thể trả về `200 OK` nhưng thiếu trường dữ liệu, đổi kiểu từ `integer` sang `string`, để lọt dữ liệu của người dùng khác (IDOR/BOLA), hoặc văng Stack Trace `500` khi nhận dữ liệu rác. Bài này tổng hợp 3 kỹ thuật bắt buộc cho mọi QA Automation / SDET.

---

## 1. JSON Schema Validation (Khóa chặt Hợp đồng Dữ liệu)

### Tại sao cần JSON Schema Validation?
- Khi bạn chỉ assert `.body("orderId", notNullValue())`, nếu Backend vô tình đổi `finalTotal` từ số `430000` thành chuỗi `"430000"`, hoặc xóa mất trường `createdAt`, hoặc trả thừa trường nhạy cảm `passwordHash` — test thông thường vẫn pass!
- **JSON Schema Validation** kiểm tra **toàn bộ cấu trúc, kiểu dữ liệu, ràng buộc định dạng (`email`, `date-time`, `uuid`), min/max, enum, trường bắt buộc (`required`) và cấm trường lạ (`additionalProperties: false`)** chỉ với một câu lệnh.

### Ví dụ Schema chuẩn (`src/test/resources/schemas/order-response.schema.json`)
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TechStore OrderResponse",
  "type": "object",
  "required": ["orderId", "status", "subtotal", "discountAmount", "shippingFee", "finalTotal"],
  "additionalProperties": false,
  "properties": {
    "orderId": { "type": "integer", "minimum": 1 },
    "status": {
      "type": "string",
      "enum": ["PENDING_PAYMENT", "CONFIRMED", "PAID", "SHIPPING", "COMPLETED", "CANCELLED"]
    },
    "subtotal": { "type": "integer", "minimum": 0 },
    "discountAmount": { "type": "integer", "minimum": 0 },
    "shippingFee": { "type": "integer", "minimum": 0 },
    "finalTotal": { "type": "integer", "minimum": 0 },
    "voucherCode": { "type": ["string", "null"] }
  }
}
```

### Áp dụng cả cho Error Response (RFC 7807 / 9457 — `error-response.schema.json`)
**Mẹo chuyên nghiệp:** Đừng chỉ validate Schema cho `200/201`! Hãy tạo thêm `error-response.schema.json` để đảm bảo **mọi** mã lỗi `400`, `401`, `403`, `404`, `409` trả về đúng format thống nhất (`timestamp`, `status`, `errorCode`, `message`) và **không bao giờ rò rỉ stack trace**.

### Chạy kiểm chứng trong REST Assured
```java
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;

given()
    .spec(buyerAuthSpec)
    .body(validCheckoutPayload)
.when()
    .post("/checkout")
.then()
    .statusCode(201)
    .body(matchesJsonSchemaInClasspath("schemas/order-response.schema.json"));
```

---

## 2. Kiểm thử Authentication & Authorization với JWT (JSON Web Token)

### A. Cấu trúc JWT & Những gì Tester cần hiểu
Một JWT gồm 3 phần ngăn cách bởi dấu chấm: `Header.Payload.Signature` (mỗi phần là Base64Url).
- **Header:** Thuật toán ký (`alg`: `HS256`, `RS256`) và loại token (`typ`: `JWT`).
- **Payload (Claims):** `sub` (userId/email), `role` (`CUSTOMER`, `ADMIN`), `iat` (thời điểm cấp), `exp` (thời điểm hết hạn). *(Lưu ý: Payload chỉ được **encode Base64**, KHÔNG được mã hóa — ai cũng giải mã đọc được, nên không bao giờ chứa password hay thông tin mật!)*.
- **Signature:** Chữ ký số bảo đảm token không bị chỉnh sửa dọc đường.

### B. Phân biệt `401 Unauthorized` vs. `403 Forbidden` (Câu hỏi phỏng vấn kinh điển!)

| Mã lỗi HTTP | Tên đúng bản chất | Khi nào API phải trả về mã này? | Ví dụ trên TechStore |
| :---: | :--- | :--- | :--- |
| **`401`** | **Unauthenticated** *(Chưa xác thực)* | Thiếu token, token sai định dạng, chữ ký giả, token hết hạn (`exp` đã qua). | Gọi `POST /checkout` không kèm `Authorization: Bearer ...`. |
| **`403`** | **Forbidden** *(Đã xác thực nhưng thiếu quyền)* | Token hoàn toàn hợp lệ, server biết bạn là ai, nhưng **không đủ quyền** thực hiện hành động hoặc chạm vào tài nguyên của người khác. | User role `CUSTOMER` gọi `POST /admin/vouchers`, hoặc **User A** gọi `GET /orders/{orderId_của_User_B}` (**BOLA / IDOR**)! |

### C. Ma trận Test Bảo mật & Phân quyền JWT (JWT Negative Matrix)

| Kịch bản Test | Header / Token gửi đi | Expected Status | Error Schema Check |
| :--- | :--- | :---: | :---: |
| **1. Missing Token** | Không có header `Authorization` | `401` | Khớp `error-response.schema.json` |
| **2. Wrong Scheme** | `Authorization: Basic dXNlcjpwYXNz` hoặc thiếu chữ `Bearer ` | `401` | Không chấp nhận |
| **3. Malformed Token** | `Bearer abc.def` (thiếu phần signature) hoặc chuỗi rác | `401` | Không nổ `500 Internal Server Error`! |
| **4. Expired Token** | JWT có `exp` ở quá khứ | `401` | Message báo token hết hạn |
| **5. Tampered Payload / Signature** | Lấy token hợp lệ, sửa `role: "CUSTOMER"` thành `"ADMIN"` trong Base64 Payload mà giữ nguyên Signature cũ | `401` | Bắt buộc fail verify chữ ký |
| **6. `alg: none` Attack** | Header đổi `{"alg":"none"}` và bỏ phần Signature | `401` | Server phải từ chối thuật toán `none`! |
| **7. Horizontal Privilege Escalation (BOLA/IDOR)** | Token hợp lệ của **Buyer A**, gọi `GET /api/v1/orders/{orderId_của_Buyer_B}` | `403` *(hoặc `404` nếu chính sách ẩn tồn tại)* | **Lỗ hổng #1 OWASP API Top 10!** |
| **8. Vertical Privilege Escalation (BFLA)** | Token hợp lệ của `CUSTOMER`, gọi `POST /api/v1/admin/vouchers` | `403` | Chặn thao tác quản trị |

### Code mẫu JUnit 5 Parameterized cho JWT Negative Matrix
```java
class JwtSecurityNegativeApiTest extends BaseApiTest {

    static Stream<Arguments> invalidJwtCases() {
        String validToken = fetchJwtToken("buyer@techstore.vn", "Buyer@123");
        String[] parts = validToken.split("\\.");
        // Giả mạo đổi signature ký tự cuối
        String tamperedSig = parts[0] + "." + parts[1] + ".INVALID_SIGNATURE_123";
        // Giả mạo alg=none (JWT unsigned attack)
        String noneAlgHeader = Base64.getUrlEncoder().withoutPadding()
                .encodeToString("{\"alg\":\"none\",\"typ\":\"JWT\"}".getBytes());
        String algNoneToken = noneAlgHeader + "." + parts[1] + ".";

        return Stream.of(
            Arguments.of("Thiếu header Authorization", null, 401),
            Arguments.of("Sai prefix không có Bearer", validToken, 401),
            Arguments.of("Token sai cấu trúc 2 phần", "Bearer header.payload", 401),
            Arguments.of("Chữ ký bị giả mạo (Tampered Signature)", "Bearer " + tamperedSig, 401),
            Arguments.of("Tấn công alg=none", "Bearer " + algNoneToken, 401)
        );
    }

    @ParameterizedTest(name = "[{index}] {0} => HTTP {2}")
    @MethodSource("invalidJwtCases")
    void shouldRejectInvalidJwtTokens(String desc, String authHeader, int expectedStatus) {
        RequestSpecification req = given().spec(unauthenticatedSpec);
        if (authHeader != null) {
            req.header("Authorization", authHeader);
        }

        req.when()
            .get("/orders/my-orders")
        .then()
            .statusCode(expectedStatus)
            .body(matchesJsonSchemaInClasspath("schemas/error-response.schema.json"));
    }
}
```

---

## 3. Negative API Testing — Nghệ thuật Phá API có Hệ thống

Ngoài bảo mật JWT, một bộ **Negative API Test** chuyên nghiệp cần phủ đủ **5 nhóm lỗi thực tế** (không được trả về `500 Internal Server Error` hoặc Stack Trace!):

| Nhóm Negative Test | Các ca cần thử trên TechStore API | Expected Behavior |
| :--- | :--- | :--- |
| **1. Input Validation & Types** | - `quantity = 0`, `-1`, `2147483648` (overflow `int`)<br>- `productId = "abc"` (sai kiểu JSON)<br>- Thiếu trường `required` (`items` missing / `null` / `[]`)<br>- Chuỗi quá dài (`voucherCode` 5.000 ký tự) | `400 Bad Request` kèm thông báo trường lỗi rõ ràng; **không** trả `500`. |
| **2. HTTP Protocol & Headers** | - Gửi `Content-Type: text/plain` thay vì `application/json`<br>- JSON gãy cú pháp (`{"userId": 1,`)<br>- Gọi `DELETE /api/v1/checkout` (sai Method) | `415 Unsupported Media Type` / `400 Bad Request` / `405 Method Not Allowed`. |
| **3. Business & State Conflicts** | - Mua vượt tồn kho (`quantity > stock`)<br>- Hủy đơn đã ở trạng thái `SHIPPING`<br>- Dùng voucher chưa đến `startDate` hoặc đã hết lượt | `409 Conflict` hoặc `400 / 422 Unprocessable Entity` (khớp chuẩn thiết kế API). |
| **4. Resource Not Found & Boundary** | - `GET /api/v1/orders/999999999` (ID không tồn tại)<br>- `GET /api/v1/products?page=-1&size=10000` (BOLA / DoS phân trang!) | `404 Not Found`; ép `size` tối đa (VD `<= 100`) tránh sập RAM DB! |
| **5. Injection & Payloads độc hại** | - `voucherCode: "' OR '1'='1"` (SQL Injection)<br>- `note: "<script>alert(1)</script>"` (XSS) | `400` hoặc lưu dưới dạng text thuần vô hại (PreparedStatement), tuyệt đối không lỗi SQL `500`. |

> **Liên hệ *Agile Testing* (Chương 10, tr. 205):** Không chỉ gửi dữ liệu sai kiểu ở từng trường riêng lẻ, hãy thử **đảo thứ tự gọi API** (ví dụ: gọi `POST /checkout` *trước* khi thêm sản phẩm vào giỏ, hoặc gọi `POST /orders/{id}/cancel` **hai lần liên tiếp**) — những ca này UI bình thường chặn nút bấm nên không thấy, nhưng gọi thẳng API sẽ phanh phui rất nhiều bug logic!
