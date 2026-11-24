# BDD với Cucumber: Hiểu Đúng Bản Chất & Tránh Anti-Patterns

> **Nguồn tham khảo:** *Full Stack Testing* (Chương 2 — *Behavior Driven Development (BDD)*, trang 49), *Agile Testing* (Chương 8 & 9 — *Business-Facing Tests that Support the Team*), Cucumber Official Docs (`cucumber.io`).

*(Ghi chú Roadmap GĐ 3b: Với Cucumber/BDD, ở giai đoạn này bạn cần **biết bản chất, đọc/viết được cú pháp Gherkin + Step Definition cơ bản và nhận diện đúng khi nào KHÔNG nên dùng**, chưa cần đào sâu xây framework lớn).*

---

## 1. BDD là gì? (Không phải là "Công cụ viết Test UI bằng tiếng Anh"!)

**Behavior-Driven Development (BDD)** — do Dan North khởi xướng — là một **quy trình cộng tác (collaboration practice)** giúp **Business/PO, Dev và QA (nhóm Three Amigos)** thống nhất yêu cầu nghiệp vụ trước khi code thông qua các **ví dụ cụ thể** viết bằng ngôn ngữ tự nhiên (**Gherkin**).

- Như *Full Stack Testing* (tr. 49) giải thích: Gherkin sử dụng cấu trúc **`Given` (Tiền điều kiện) – `When` (Hành động) – `Then` (Kết quả mong đợi)** để biến yêu cầu người dùng thành các **tài liệu sống thực thi được (Executable Living Documentation)**.
- **Cucumber** là công cụ phổ biến nhất đọc các file `.feature` viết bằng Gherkin và ánh xạ từng câu chữ sang mã Java (Step Definitions) để chạy tự động trên JUnit 5 hoặc TestNG.

---

## 2. Cấu trúc Chuẩn của một Feature File (Gherkin) & Runner Java

### A. File đặc tả (`src/test/resources/features/voucher_checkout.feature`)
```gherkin
Feature: Áp dụng mã giảm giá khi thanh toán TechStore
  Là một khách hàng TechStore
  Tôi muốn áp dụng mã giảm giá hợp lệ cho đơn hàng
  Để được trừ đúng số tiền khuyến mãi theo chính sách

  Background:
    Given khách hàng "buyer@techstore.vn" đã đăng nhập và có sản phẩm thường giá 500000 VND trong giỏ

  @smoke @voucher
  Scenario: Áp dụng thành công mã SALE20 cho đơn hàng đủ điều kiện
    When khách hàng áp dụng mã giảm giá "SALE20"
    Then số tiền được giảm phải là 100000 VND
    And tổng thanh toán cuối cùng phải là 430000 VND

  @regression
  Scenario Outline: Kiểm tra quy tắc phí vận chuyển theo hạng thành viên và giá trị đơn
    Given giỏ hàng có tổng tiền hàng là <subtotal> VND và trạng thái VIP là "<isVip>"
    When hệ thống tính phí vận chuyển
    Then phí vận chuyển phải là <shippingFee> VND

    Examples:
      | subtotal | isVip | shippingFee |
      | 499999   | false | 30000       |
      | 500000   | false | 0           |
      | 150000   | true  | 0           |
```

### B. Các từ khóa Gherkin cốt lõi
| Từ khóa | Vai trò |
| :--- | :--- |
| `Feature` | Tên và mô tả giá trị nghiệp vụ của tính năng. |
| `Background` | Các bước `Given` lặp lại giống nhau cho **mọi Scenario** trong cùng file `.feature`. |
| `Scenario` | Một kịch bản kiểm thử cụ thể (chuẩn `Given - When - Then`, nối tiếp bằng `And` / `But`). |
| `Scenario Outline` + `Examples` | Test Data-Driven chạy lặp lại 1 kịch bản cho bảng giá trị (`<placeholder>`). |
| `@smoke`, `@regression` | Gắn thẻ (Tags) để lọc chạy bộ Smoke Test nhanh trên CI. |

### C. Step Definitions + Hooks + JUnit 5 Runner (Cucumber JVM 7)
```java
// stepdefinitions/VoucherSteps.java
package com.techstore.bdd.steps;

import io.cucumber.java.en.*;
import static org.assertj.core.api.Assertions.assertThat;

public class VoucherSteps {
    // Chia sẻ state giữa các class Step Definitions khác nhau thông qua PicoContainer (Constructor Injection)
    private final TestContext context;

    public VoucherSteps(TestContext context) {
        this.context = context;
    }

    @Given("khách hàng {string} đã đăng nhập và có sản phẩm thường giá {int} VND trong giỏ")
    public void userLoggedInWithItem(String email, int price) {
        context.setAuthToken(authApi.login(email));
        context.setSubtotal(price);
    }

    @When("khách hàng áp dụng mã giảm giá {string}")
    public void applyVoucher(String code) {
        context.setLastResponse(checkoutApi.previewOrder(context.getAuthToken(), code, context.getSubtotal()));
    }

    @Then("số tiền được giảm phải là {int} VND")
    public void verifyDiscountAmount(int expectedDiscount) {
        assertThat(context.getLastResponse().discountAmount()).isEqualTo(expectedDiscount);
    }
}
```
- **Hooks (`@Before` / `@After` trong `io.cucumber.java`):** Khởi tạo / dọn dẹp dữ liệu hoặc chụp screenshot đính kèm vào Cucumber Scenario (`scenario.attach(bytes, "image/png", "failure")`) khi `scenario.isFailed()`.
- **Runner chuẩn JUnit 5 (`RunCucumberTest.java`):**
  ```java
  @Suite
  @IncludeEngines("cucumber")
  @SelectClasspathResource("features")
  @ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.techstore.bdd")
  public class RunCucumberTest {}
  ```

---

## 3. Declarative vs. Imperative Gherkin & 4 Anti-Patterns Cần Tránh

Sai lầm phổ biến nhất khiến nhiều team ghét Cucumber là biến Gherkin thành **bản dịch từng cú click chuột UI**:

| ❌ Imperative Gherkin (Anti-pattern — Nên TRÁNH!) | ✅ Declarative Gherkin (Chuẩn BDD — Nên DÙNG!) |
| :--- | :--- |
| `Given tôi mở trình duyệt Chrome và vào "/login"`<br>`When tôi nhập "buyer@techstore.vn" vào ô "#email"`<br>`And tôi click nút "#btn-login"`<br>`And tôi nhập "SALE20" vào input "#voucher"`<br>`Then text của span ".discount" là "100000"` | `Given khách hàng đã đăng nhập và có đơn 500000 VND`<br>`When khách hàng áp dụng mã giảm giá "SALE20"`<br>`Then số tiền được giảm phải là 100000 VND` |

### 4 Anti-Patterns chết người khi dùng Cucumber
1. **Dùng Cucumber khi chỉ có mình QA viết và đọc:** Nếu PO/BA/Dev **không bao giờ đọc** file `.feature`, việc viết thêm lớp Gherkin + Regex Step Definitions chỉ làm tăng gấp đôi công bảo trì code so với JUnit/TestNG thuần!
2. **Đẩy 30 dòng `Scenario Outline` chạy qua Selenium UI:** Nhân chậm thời gian chạy trình duyệt lên 30 lần; các bảng BVA nhiều dòng nên chạy ở tầng Unit/API.
3. **Dùng biến `public static` để truyền dữ liệu giữa các Step Definition:** Khi chạy parallel sẽ gây đụng dữ liệu giữa các luồng (`Race Condition`). Hãy dùng **`cucumber-picocontainer`** hoặc **Spring DI**.
4. **Nhét logic `if-else` và `WebDriver` trực tiếp vào Step Definition:** Step Definition chỉ nên là cầu nối mỏng gọi xuống **API Client** hoặc **Page Object**!
