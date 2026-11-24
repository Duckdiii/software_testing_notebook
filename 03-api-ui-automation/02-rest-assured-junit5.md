# REST Assured (Java) kết hợp JUnit 5

> **Nguồn tham khảo:** REST Assured Official Documentation (5.x), *Full Stack Testing* (Chương 2 — *Service Tests: Java-REST Assured Framework*, trang 59–63), Allure REST Assured Documentation.

**REST Assured** là thư viện Java DSL hàng đầu để tự động hóa kiểm thử RESTful API theo cú pháp hành vi dễ đọc **`given() → when() → then()`**. Dù trong *Full Stack Testing* ví dụ minh họa dùng REST Assured + TestNG, ở các dự án Spring Boot / Java hiện đại, **REST Assured + JUnit 5** (hoặc dùng TestNG nếu gom chung UI TestNG) là bộ đôi tiêu chuẩn cực kỳ mạnh mẽ.

---

## 1. Cấu hình Maven & Kiến trúc Cơ bản (`pom.xml`)

```xml
<properties>
    <rest-assured.version>5.5.0</rest-assured.version>
    <junit.version>5.11.4</junit.version>
    <allure.version>2.29.0</allure.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>${rest-assured.version}</version>
        <scope>test</scope>
    </dependency>
    <!-- Module kiểm chứng JSON Schema -->
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>json-schema-validator</artifactId>
        <version>${rest-assured.version}</version>
        <scope>test</scope>
    </dependency>
    <!-- Jackson để tự động Serialize/Deserialize POJO & Java Record -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.18.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>${junit.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>io.qameta.allure</groupId>
        <artifactId>allure-rest-assured</artifactId>
        <version>${allure.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 2. Tư duy `given() / when() / then()` & Tái sử dụng Specification

Nếu viết lặp đi lặp lại `.baseUri(...)`, `.contentType(ContentType.JSON)`, `.header("Authorization", ...)` trong 50 test method, code sẽ phình to và khó bảo trì. Giải pháp chuẩn là **`RequestSpecification`** và **`ResponseSpecification`**:

```java
package com.techstore.api.base;

import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.RestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.builder.ResponseSpecBuilder;
import io.restassured.filter.log.LogDetail;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;
import io.restassured.specification.ResponseSpecification;
import org.junit.jupiter.api.BeforeAll;

import static org.hamcrest.Matchers.lessThan;

public abstract class BaseApiTest {

    protected static RequestSpecification unauthenticatedSpec;
    protected static RequestSpecification buyerAuthSpec;
    protected static ResponseSpecification fastJsonResponseSpec;

    @BeforeAll
    static void initApiSpecs() {
        String baseUrl = System.getProperty("api.baseUrl", "http://localhost:8080");

        // Ghi request/response vào Allure Report & chỉ log ra console khi validation FAIL!
        RestAssured.enableLoggingOfRequestAndResponseIfValidationFails();

        unauthenticatedSpec = new RequestSpecBuilder()
                .setBaseUri(baseUrl)
                .setBasePath("/api/v1")
                .setContentType(ContentType.JSON)
                .setAccept(ContentType.JSON)
                .addFilter(new AllureRestAssured())
                .build();

        // Lấy JWT 1 lần dùng chung cho cả suite (tiết kiệm hàng chục request login thừa!)
        String jwtToken = fetchJwtToken("buyer@techstore.vn", "Buyer@123");

        buyerAuthSpec = new RequestSpecBuilder()
                .addRequestSpecification(unauthenticatedSpec)
                .addHeader("Authorization", "Bearer " + jwtToken)
                .build();

        fastJsonResponseSpec = new ResponseSpecBuilder()
                .expectContentType(ContentType.JSON)
                .expectResponseTime(lessThan(2000L)) // SLA phản hồi < 2 giây
                .build();
    }

    protected static String fetchJwtToken(String email, String password) {
        return RestAssured.given()
                .spec(unauthenticatedSpec)
                .body(Map.of("email", email, "password", password))
            .when()
                .post("/auth/login")
            .then()
                .statusCode(200)
                .extract()
                .path("accessToken");
    }
}
```

> **Mẹo chống rác Log trên CI:** Thay vì `.log().all()` làm tràn log GitHub Actions, hãy dùng `RestAssured.enableLoggingOfRequestAndResponseIfValidationFails()` (hoặc `.log().ifValidationFails(LogDetail.ALL)`). Test pass thì log sạch sẽ, test fail thì in trọn vẹn Request + Response!

---

## 3. Serialize / Deserialize với Java Record & Jackson (Không ghép chuỗi JSON!)

Như *Full Stack Testing* (Ví dụ 3-15 & 3-16, trang 62–63) đã hướng dẫn: **tránh cộng chuỗi JSON thủ công** (`"{\"code\":\"" + code + "\"}"`) vì rất dễ lỗi dấu ngoặc kép và ký tự đặc biệt. Hãy dùng **Java `record` / POJO** để Jackson tự chuyển đổi 2 chiều:

```java
// Payload DTO dùng Java Record (Java 17+)
public record CheckoutPayload(
    Long userId,
    boolean vip,
    boolean hasFreeshipCode,
    String voucherCode,
    String paymentMethod,
    List<OrderItemPayload> items
) {
    public record OrderItemPayload(Long productId, int quantity) {}
}

public record OrderSummaryResponse(
    Long orderId,
    String status,
    int subtotal,
    int discountAmount,
    int shippingFee,
    int finalTotal
) {}
```

### Sử dụng trong JUnit 5 `@Test` & `@ParameterizedTest`
```java
class CheckoutApiTest extends BaseApiTest {

    @Test
    @DisplayName("POST /checkout tạo đơn hàng và deserialize thành OrderSummaryResponse")
    void shouldCreateOrderSuccessfully() {
        CheckoutPayload payload = new CheckoutPayload(
                101L, false, false, "SALE20", "COD",
                List.of(new CheckoutPayload.OrderItemPayload(1L, 1))
        );

        OrderSummaryResponse actual = given()
                .spec(buyerAuthSpec)
                .body(payload) // Jackson tự serialize Object -> JSON!
            .when()
                .post("/checkout")
            .then()
                .spec(fastJsonResponseSpec)
                .statusCode(201)
                .body("status", equalTo("PENDING_PAYMENT"))
                .extract()
                .as(OrderSummaryResponse.class); // Jackson tự deserialize JSON -> Record!

        // Kết hợp AssertJ kiểm tra đa trường chặt chẽ
        assertThat(actual.orderId()).isPositive();
        assertThat(actual.discountAmount()).isEqualTo(100_000);
        assertThat(actual.finalTotal()).isEqualTo(430_000);
    }
}
```

---

## 4. Truy vấn sâu JSON Body bằng `JsonPath` & GPath (Groovy Expressions)

REST Assured dùng **GPath** (cú pháp Groovy) cho phép lọc và kiểm tra mảng JSON phức tạp chỉ trong một dòng mà không cần viết vòng lặp Java:

```java
given()
    .spec(buyerAuthSpec)
    .queryParam("category", "LAPTOP")
    .queryParam("page", 0)
    .queryParam("size", 10)
.when()
    .get("/products")
.then()
    .statusCode(200)
    // Kiểm tra mảng content không rỗng và mọi phần tử đều có price > 0
    .body("content", not(empty()))
    .body("content.every { it.price > 0 }", is(true))
    // Tìm phần tử theo điều kiện ngay trong JSON
    .body("content.find { it.sku == 'SKU-LAPTOP-01' }.stock", greaterThanOrEqualTo(1))
    // Thu thập danh sách tất cả SKU trả về
    .body("content.sku", hasItem("SKU-LAPTOP-01"));
```

### Trích xuất Response để dùng cho bước tiếp theo (`extract()`)
```java
Response res = given()
        .spec(buyerAuthSpec)
        .pathParam("id", createdOrderId)
    .when()
        .get("/orders/{id}")
    .then()
        .statusCode(200)
        .extract().response();

String status = res.jsonPath().getString("status");
List<Integer> itemQuantities = res.jsonPath().getList("items.quantity", Integer.class);
```

---

## 5. Các cấu hình nâng cao hay dùng trong thực tế

1. **Xử lý Upload Multipart File (`multipart`):**
   ```java
   given()
       .spec(adminAuthSpec)
       .contentType(ContentType.MULTIPART)
       .multiPart("file", new File("src/test/resources/testdata/product-thumb.png"), "image/png")
       .multiPart("productId", "101")
   .when()
       .post("/products/upload-image")
   .then()
       .statusCode(200);
   ```
2. **Xử lý SSL Self-Signed ở môi trường Staging nội bộ:** `.relaxedHTTPSValidation()`.
3. **Kiểm soát Redirect khi test link tải file / SSO:** `.redirects().follow(false)`.
4. **Timeout Configuration tránh treo CI:**
   ```java
   RestAssured.config = RestAssuredConfig.config().httpClient(
       HttpClientConfig.httpClientConfig()
           .setParam("http.connection.timeout", 5000)
           .setParam("http.socket.timeout", 10000)
   );
   ```
5. **Kết hợp JUnit 5 `@ParameterizedTest` (`@MethodSource` / `@CsvSource`):** Đổ thẳng bộ EP/BVA và Negative Security Matrix vào REST Assured mà không phải nhân bản method test (xem bài `03-api-json-schema-jwt-negative-testing.md`).
