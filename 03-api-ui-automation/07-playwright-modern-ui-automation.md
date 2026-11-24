# Playwright (Java & TypeScript): Automation Hiện đại, Chống Flaky & Hybrid API+UI

> **Nguồn tham khảo:** Playwright Official Documentation (`playwright.dev`, `playwright.dev/java`), Microsoft Playwright Best Practices (2025–2026).

Sau khi nắm vững Selenium (vốn là nền tảng phổ biến ở đa số công ty Việt Nam), **Playwright** (của Microsoft) là công nghệ đang tăng trưởng tuyển dụng mạnh nhất nhờ khắc phục triệt để 3 nỗi đau kinh điển của Selenium: **cài đặt driver rườm rà, test flaky do chờ đợi thủ công, và debug lỗi CI mù mờ**.

---

## 1. Tại sao Playwright ít Flaky và Nhanh hơn Selenium?

| Tiêu chí | Selenium 4 (WebDriver) | Playwright (TypeScript / Java) |
| :--- | :--- | :--- |
| **Kiến trúc giao tiếp** | Gửi từng HTTP command rời rạc qua WebDriver (`Code → HTTP → ChromeDriver → Browser`). | Kết nối **2 chiều bền vững (WebSocket / CDP+)** trực tiếp tới browser context — nhận sự kiện DOM/Network tức thời. |
| **Quản lý trình duyệt** | Điều khiển trình duyệt cài sẵn qua Selenium Manager. | Tự tải bản Chromium, Firefox, **WebKit (nhân Safari)** riêng biệt được vá chuẩn hóa. |
| **Cô lập mỗi Test** | Khởi tạo `new ChromeDriver()` tốn nhiều RAM & ~1–2s mỗi lần mở. | Dùng **`BrowserContext`** (như tab ẩn danh độc lập hoàn toàn về Cookie/LocalStorage) — tạo mới chỉ mất **vài mili-giây**! |
| **Cơ chế Chờ (Waiting)** | Phải tự viết `WebDriverWait(ExpectedConditions...)` cho mọi tương tác. | **Auto-waiting tích hợp sẵn** vào mọi hành động (`.click()`, `.fill()`) & **Web-First Assertions** tự retry! |
| **Tái sử dụng Đăng nhập** | Tự extract cookie, mở trang domain rồi nạp lại cookie. | **`storageState()`** lưu & nạp toàn bộ Cookie + `localStorage` bằng **1 dòng code**! |
| **Debug khi Fail trên CI** | Chụp ảnh màn hình tĩnh + console log. | **Trace Viewer (`.zip`)**: tua lại từng mili-giây kèm DOM snapshot tương tác được, Network HAR, Console & Screenshot! |
| **Ngôn ngữ chính** | **Java** là hệ sinh thái mạnh nhất. | **TypeScript** (`@playwright/test`) là hệ sinh thái gốc đầy đủ nhất; ngoài ra có **Java**, Python, .NET. |

---

## 2. Ba Vũ khí Cốt lõi của Playwright

### Vũ khí 1: Auto-Waiting & Actionability Checks + Web-First Assertions
Trước khi thực hiện `locator.click()`, Playwright tự động chạy **Actionability Checks** kiểm tra phần tử:
1. Đã gắn vào DOM (*Attached*)
2. Đang hiển thị (*Visible*)
3. Đã ngừng chuyển động animation (*Stable*)
4. Không bị modal/overlay khác che khuất (*Receives Events*)
5. Không bị `disabled` (*Enabled*)

Đồng thời, thay vì lấy text ra rồi assert tĩnh (dễ sai khi React chưa kịp cập nhật text), Playwright dùng **Web-First Assertions** tự retry liên tục đến khi thỏa hoặc hết timeout (`5s`):
- **Java:** `assertThat(page.getByTestId("order-status")).hasText("PENDING_PAYMENT");`
- **TypeScript:** `await expect(page.getByTestId('order-status')).toHaveText('PENDING_PAYMENT');`

### Vũ khí 2: Modern User-Facing Locators (Gần với cách người dùng nhìn thấy)
Playwright khuyến khích tìm element theo vai trò ngữ nghĩa (Accessibility role) và `data-testid` thay vì CSS/XPath phức tạp:
- `page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Đặt hàng"))`
- `page.getByLabel("Mã giảm giá")`
- `page.getByPlaceholder("Nhập email...")`
- `page.getByTestId("final-total")` *(tương ứng thuộc tính `data-testid="final-total"`)*

### Vũ khí 3: `storageState` + `APIRequestContext` (Hybrid API + UI — Không bao giờ Login lặp qua UI!)
Thay vì để 20 UI test mất 20 lần gõ email/password trên form Login:
1. Dùng `APIRequestContext` của Playwright gọi thẳng `POST /api/v1/auth/login` (hoặc login UI **1 lần duy nhất** ở setup).
2. Lưu trạng thái xác thực ra `auth-buyer.json` qua `context.storageState(...)`.
3. Mọi test Checkout tiếp theo tạo `BrowserContext` nạp thẳng file `auth-buyer.json` → **mở thẳng `/checkout` ở trạng thái đã đăng nhập chỉ trong 50ms!**

---

## 3. Ví dụ Code Hoàn chỉnh: Playwright Java vs. Playwright TypeScript

### A. Playwright Java (Kết hợp JUnit 5) — Hybrid API Setup + Checkout UI + Trace Viewer
```java
package com.techstore.playwright;

import com.microsoft.playwright.*;
import com.microsoft.playwright.options.AriaRole;
import org.junit.jupiter.api.*;

import java.nio.file.Paths;
import java.util.Map;

import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;

class TechStoreCheckoutPlaywrightTest {

    static Playwright playwright;
    static Browser browser;
    BrowserContext context;
    Page page;

    @BeforeAll
    static void launchBrowser() {
        playwright = Playwright.create();
        browser = playwright.chromium().launch(
                new BrowserType.LaunchOptions().setHeadless(true));

        // BƯỚC 1: Seed dữ liệu & đăng nhập 1 lần qua APIRequestContext (nhanh gấp 20 lần UI login!)
        APIRequestContext api = playwright.request().newContext(
                new APIRequest.NewContextOptions().setBaseURL("http://localhost:8080"));
        APIResponse loginRes = api.post("/api/v1/auth/login",
                com.microsoft.playwright.options.RequestOptions.create()
                        .setData(Map.of("email", "buyer@techstore.vn", "password", "Buyer@123")));
        Assertions.assertEquals(200, loginRes.status());
        api.storageState(new APIRequestContext.StorageStateOptions()
                .setPath(Paths.get("target/auth-state.json")));
        api.dispose();
    }

    @BeforeEach
    void createIsolatedContext() {
        // BƯỚC 2: Mỗi test dùng 1 BrowserContext sạch nạp sẵn session từ auth-state.json
        context = browser.newContext(new Browser.NewContextOptions()
                .setStorageStatePath(Paths.get("target/auth-state.json")));

        // Bật Trace Viewer (chụp DOM snapshot + network + screenshots mỗi bước)
        context.tracing().start(new Tracing.StartOptions()
                .setScreenshots(true)
                .setSnapshots(true)
                .setSources(true));

        page = context.newPage();
    }

    @Test
    void shouldApplySale20AndPlaceOrderSmoothly() {
        page.navigate("http://localhost:3000/checkout");

        // Không cần viết Thread.sleep hay ExplicitWait — Playwright tự chờ sẵn sàng!
        page.getByLabel("Mã giảm giá").fill("SALE20");
        page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Áp dụng")).click();

        // Web-first assertion tự retry nếu API tính giá mất 300ms mới trả về DOM!
        assertThat(page.getByTestId("discount-amount")).hasText("-100.000đ");
        assertThat(page.getByTestId("final-total")).hasText("430.000đ");

        page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Đặt hàng")).click();
        assertThat(page.getByTestId("order-status")).hasText("PENDING_PAYMENT");
    }

    @AfterEach
    void closeContext(TestInfo testInfo) {
        // Lưu file trace.zip để mở xem lại timeline từng bước khi debug!
        context.tracing().stop(new Tracing.StopOptions()
                .setPath(Paths.get("target/traces/" + testInfo.getDisplayName() + ".zip")));
        context.close();
    }

    @AfterAll
    static void tearDownAll() {
        browser.close();
        playwright.close();
    }
}
```

### B. Playwright TypeScript (`@playwright/test`) — Tham khảo cú pháp gốc cực gọn
*(Nhiều JD QA Automation hiện nay tuyển song song Java-Selenium hoặc TypeScript-Playwright)*
```typescript
import { test, expect } from '@playwright/test';

test('Checkout áp dụng SALE20 thành công', async ({ page }) => {
  await page.goto('/checkout');
  await page.getByLabel('Mã giảm giá').fill('SALE20');
  await page.getByRole('button', { name: 'Áp dụng' }).click();

  await expect(page.getByTestId('discount-amount')).toHaveText('-100.000đ');
  await expect(page.getByTestId('final-total')).toHaveText('430.000đ');

  await page.getByRole('button', { name: 'Đặt hàng' }).click();
  await expect(page.getByTestId('order-status')).toHaveText('PENDING_PAYMENT');
});
```

---

## 4. Nên học Selenium trước hay Playwright trước? (Góc nhìn Nghề nghiệp)

- **Lộ trình tối ưu theo Roadmap GĐ 3b:**
  1. Làm chủ **Selenium + Java + TestNG + POM + `ThreadLocal`** trước — vì phần lớn hệ thống Enterprise/Bank/Outsource tại Việt Nam vẫn đang vận hành các bộ Selenium Java lớn, và mọi nguyên lý thiết kế Framework (Locator, Explicit Wait, POM, Parallel Thread Safety) đều xuất phát từ đây.
  2. Học tiếp **Playwright** ngay sau đó — chỉ mất vài ngày để làm chủ vì tư duy POM và Locator giống nhau, nhưng bạn sẽ có thêm lợi thế cạnh tranh rất lớn ở các công ty Product hiện đại tuyển Playwright!
