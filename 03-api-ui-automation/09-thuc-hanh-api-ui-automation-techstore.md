# Thực hành Giai đoạn 3: Xây dựng Bộ Automation API + UI cho TechStore

> **Mục tiêu:** Áp dụng toàn bộ công cụ Giai đoạn 3 (**Postman + Newman**, **REST Assured + JUnit 5**, **Selenium 4 + POM + TestNG Parallel**, và **Playwright Hybrid**) vào các luồng nghiệp vụ của **TechStore** — giữ **tầng API dày** và **tầng UI tối giản ở đỉnh Test Pyramid**.

---

## Phân bổ tầng kiểm thử cho bài Thực hành

| Phần | Công cụ | Phạm vi thực hành trên TechStore | Số lượng ca gợi ý |
| :---: | :--- | :--- | :---: |
| **A** | **Postman + Newman** | Khám phá & chạy hồi quy nhanh Auth + Products + Checkout (Pre-request auto-login + Ajv JSON Schema) | 6–8 requests |
| **B** | **REST Assured + JUnit 5** | Full API Regression: Happy Path, BVA (`499.999` vs `500.000`, `COD > 5M`), JSON Schema, JWT Negative + BOLA (`401/403`) | ~15 test cases |
| **C** | **Selenium 4 + POM + TestNG** | Chỉ giữ **Critical User Journeys** ở đỉnh Pyramid: Đăng nhập → Checkout áp `SALE20` → Lỗi giỏ hàng trống/hết hàng + Chạy Parallel + Chụp Screenshot khi Fail | 3–4 test cases |
| **D** | **Playwright (Java)** | Viết lại luồng Checkout dùng `APIRequestContext` + `storageState` (không login qua UI) + xuất `Trace Viewer` | 2 test cases |

---

## Phần A: Thực hành Postman + Newman CLI

### Bước 1: Tạo Collection & Environment
1. Tạo Environment `TechStore-Local` gồm:
   - `base_url` = `http://localhost:8080`
   - `buyer_email` = `buyer@techstore.vn`
   - `buyer_password` = *(chỉ điền ở Current Value, để trống Initial Value)*
   - `jwt_token` = *(tự động điền bởi script)*
2. Tạo Collection `TechStore-API-Suite` với 3 thư mục: `01-Auth`, `02-Catalog`, `03-Checkout`.

### Bước 2: Gắn Pre-request Script Auto-Login ở cấp Collection
Sử dụng script `pm.sendRequest(...)` từ bài `01-postman-collection-environment-scripts.md` để mỗi khi `jwt_token` rỗng hoặc hết hạn, Postman tự gọi `POST {{base_url}}/api/v1/auth/login` cập nhật `jwt_token`.

### Bước 3: Viết Test Script + Ajv JSON Schema cho `POST /api/v1/checkout`
- Kiểm tra `statusCode == 201`, `responseTime < 500ms`.
- Kiểm tra `discountAmount === 100000`, `shippingFee === 30000`, `finalTotal === 430000` khi mua sản phẩm `500.000đ` với mã `SALE20`.
- Lưu `pm.environment.set("last_order_id", body.orderId)` để request tiếp theo (`GET /api/v1/orders/{{last_order_id}}`) xác minh dữ liệu vừa lưu.
- Chạy `Ajv` validate cấu trúc `OrderResponse`.

### Bước 4: Export & Chạy qua Newman CLI
```bash
newman run TechStore-API-Suite.postman_collection.json \
  -e TechStore-Local.postman_environment.json \
  --env-var "buyer_password=Buyer@123" \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export target/newman-report.html
```

---

## Phần B: Thực hành REST Assured + JUnit 5 (BVA + JSON Schema + JWT Security)

Dựng class `CheckoutAndSecurityApiTest extends BaseApiTest` bao phủ trọn bộ ca API từ GĐ 1:

```java
package com.techstore.api;

import com.techstore.api.base.BaseApiTest;
import com.techstore.api.dto.CheckoutPayload;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import java.util.List;

import static io.restassured.RestAssured.given;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
import static org.hamcrest.Matchers.*;

@DisplayName("API Tests — TechStore Checkout, BVA & Security (JUnit 5 + REST Assured)")
class CheckoutAndSecurityApiTest extends BaseApiTest {

    @ParameterizedTest(name = "BVA Checkout: productId={0}, voucher={1}, pay={2} => HTTP {3}")
    @CsvSource({
        "1, SALE20, COD,   201", // Sản phẩm 500k + SALE20 -> giảm 100k + ship 30k = 430k
        "2, ,       COD,   409", // Sản phẩm vượt tồn kho hoặc hết hàng -> 409 Conflict
        "3, ,       COD,   400", // Sản phẩm giá 5.000.001đ chọn COD -> vượt hạn mức 5M
        "3, ,       VNPAY, 201"  // Sản phẩm giá 5.000.001đ chọn VNPAY -> hợp lệ!
    })
    void shouldEnforceBusinessBoundariesOverHttp(
            Long productId, String voucherCode, String paymentMethod, int expectedStatus) {
        var payload = new CheckoutPayload(101L, false, false, voucherCode, paymentMethod,
                List.of(new CheckoutPayload.OrderItemPayload(productId, 1)));

        String schema = (expectedStatus == 201)
                ? "schemas/order-response.schema.json"
                : "schemas/error-response.schema.json";

        given()
            .spec(buyerAuthSpec)
            .body(payload)
        .when()
            .post("/checkout")
        .then()
            .statusCode(expectedStatus)
            .body(matchesJsonSchemaInClasspath(schema));
    }

    @Test
    @DisplayName("Security BOLA/IDOR: Buyer A không được phép xem chi tiết đơn hàng của Buyer B -> 403")
    void shouldPreventHorizontalPrivilegeEscalationOnOrderDetail() {
        // Buyer B (tài khoản khác) tạo 1 đơn hàng
        String buyerBToken = fetchJwtToken("buyerB@techstore.vn", "Buyer@123");
        Long buyerBOrderId = given()
                .spec(unauthenticatedSpec)
                .header("Authorization", "Bearer " + buyerBToken)
                .body(new CheckoutPayload(102L, false, false, null, "VNPAY",
                        List.of(new CheckoutPayload.OrderItemPayload(1L, 1))))
            .when()
                .post("/checkout")
            .then()
                .statusCode(201)
                .extract().jsonPath().getLong("orderId");

        // Buyer A cố tình gọi GET /orders/{buyerBOrderId} -> BẮT BUỘC trả 403 Forbidden (hoặc 404)!
        given()
            .spec(buyerAuthSpec)
            .pathParam("id", buyerBOrderId)
        .when()
            .get("/orders/{id}")
        .then()
            .statusCode(anyOf(is(403), is(404)))
            .body(matchesJsonSchemaInClasspath("schemas/error-response.schema.json"));
    }
}
```

---

## Phần C: Thực hành Selenium 4 + POM + TestNG Parallel + Screenshot on Fail

Theo nguyên tắc **giữ UI Test ở đỉnh Test Pyramid**, không lặp lại 20 ca validate dữ liệu của Phần B trên trình duyệt! Chỉ xây dựng **3 Page Objects** (`LoginPage`, `ProductCatalogPage`, `CartCheckoutPage`) cho 2–3 luồng người dùng cuối (*Critical User Journeys*):

1. **Test 1 (`verifyCriticalCheckoutWithSale20`):**
   - Mở `/login` → đăng nhập qua `LoginPage.loginSuccess(...)` → vào `ProductCatalogPage` chọn sản phẩm `500.000đ` thêm vào giỏ → sang `CartCheckoutPage` nhập `SALE20` → bấm Áp dụng → Verify hiển thị giảm `-100.000đ` và tổng `430.000đ` → bấm Đặt hàng → Verify chuyển trang xác nhận `PENDING_PAYMENT`.
2. **Test 2 (`verifyNegativeVoucherErrorOnUi`):**
   - Giỏ hàng `300.000đ` → nhập `SALE20` → Verify banner lỗi tiếng Việt hiển thị rõ ràng (`"Chưa đạt đơn tối thiểu 500.000đ"`), nút Đặt hàng vẫn hoạt động bình thường khi gỡ mã.
3. **Thực nghiệm Chạy Parallel & Bắt Screenshot khi Fail:**
   - Cấu hình `testng.xml` với `parallel="methods" thread-count="2"`, dùng `DriverManager` (`ThreadLocal<WebDriver>`).
   - Cố tình sửa expected text trong Test 2 sai 1 ký tự để kiểm chứng `ScreenshotFailureListener.onTestFailure()` tự động chụp ảnh `.png` lưu vào `target/screenshots/` và đính kèm vào Allure Report trước khi `DriverManager.quitDriver()` đóng Chrome!

---

## Phần D: Thực hành Nâng cấp với Playwright (Java) Hybrid API + UI

Tạo thêm một class test Playwright để thấy trực tiếp sự khác biệt về **tốc độ** và **độ ổn định**:
1. Trong `@BeforeAll`: Dùng `playwright.request().newContext()` gọi `POST /api/v1/auth/login` và `POST /api/v1/cart/items` chuẩn bị sẵn giỏ hàng, lưu `storageState` ra `target/buyer-state.json`.
2. Trong `@Test`: Mở thẳng `http://localhost:3000/checkout` (bỏ qua hoàn toàn UI Login & Add-to-Cart!), áp `SALE20`, assert bằng **Web-first Assertion** (`assertThat(page.getByTestId("final-total")).hasText("430.000đ")`).
3. Trong `@AfterEach`: Xuất `target/traces/checkout-trace.zip`. Chạy lệnh xem lại từng khung hình và network call:
   ```bash
   mvn exec:java -e -D exec.mainClass=com.microsoft.playwright.CLI -D exec.args="show-trace target/traces/checkout-trace.zip"
   ```

---

## So sánh Tổng kết sau Thực hành (Dùng để trả lời Phỏng vấn)

| Tiêu chí đo thực tế | REST Assured (API Layer) | Selenium 4 + TestNG (UI Layer) | Playwright Hybrid (API Setup + UI) |
| :--- | :---: | :---: | :---: |
| **Thời gian chạy 1 Test Case** | `~50–150 ms` | `~4–8 giây` | `~1.5–3 giây` |
| **Nguyên nhân hay gãy nhất** | Đổi hợp đồng JSON / DB seed | Thay đổi CSS/XPath, animation, timing | Hiếm gặp nhờ Auto-waiting |
| **Phù hợp cho loại Test nào?** | EP/BVA, Decision Table, Negative, Auth/BOLA | Critical E2E User Flows, Cross-browser | Critical Flows, SPA nhiều async UI |
