# Selenium 4 + Java + TestNG + Maven: Kiến trúc, Locator, Wait & Screenshot

> **Nguồn tham khảo:** *Full Stack Testing* (Chương 2 — *Java-Selenium Framework*, trang 33–48), Selenium 4 Official Documentation (`selenium.dev`), TestNG Documentation.

**Selenium WebDriver** là công cụ chuẩn mực lâu đời nhất để tự động hóa thao tác trình duyệt thực tế. Như *Full Stack Testing* (tr. 37) nhấn mạnh: Selenium WebDriver **chỉ làm một việc duy nhất là điều khiển trình duyệt** — để trở thành một framework kiểm thử hoàn chỉnh, nó phải ghép với **Maven** (quản lý thư viện & build lifecycle), **TestNG / JUnit** (quản lý test, assertion, chạy song song), và công cụ báo cáo (**Allure Report**).

---

## 1. Kiến trúc Selenium 4 & Selenium Manager (Cập nhật hiện đại)

### A. Giao tiếp qua chuẩn W3C WebDriver
Khác với Selenium 3 dùng JSON Wire Protocol, **Selenium 4** chuẩn hóa hoàn toàn theo giao thức **W3C WebDriver**, giao tiếp trực tiếp qua HTTP REST tới trình điều khiển trình duyệt (`chromedriver`, `geckodriver`, `msedgedriver`).

### B. Tạm biệt `System.setProperty(...)` và `WebDriverManager`!
Trong sách *Full Stack Testing* (xuất bản 2022, tr. 44), bạn thấy dòng:
`System.setProperty("webdriver.chrome.driver", "src/main/resources/chromedriver");`
- **Cập nhật quan trọng (từ Selenium 4.6+ đến 4.27+/4.x hiện nay):** Selenium tích hợp sẵn **Selenium Manager** ngay bên trong `selenium-java`.
- Khi bạn gọi `new ChromeDriver()`, Selenium Manager tự động phát hiện phiên bản Chrome trên máy, tải đúng `chromedriver` tương thích về cache (`~/.cache/selenium`) và chạy ngay — **không cần tải file `.exe` thủ công hay cài thư viện `bonigarcia/webdrivermanager` nữa**!

### C. Cấu hình `pom.xml` chuẩn (Selenium 4 + TestNG + Allure)
```xml
<properties>
    <selenium.version>4.27.0</selenium.version>
    <testng.version>7.10.2</testng.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>${selenium.version}</version>
    </dependency>
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>${testng.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

## 2. Chiến lược chọn Locator (Locator Strategy — Thứ tự Ưu tiên Vàng)

Chọn locator kém là nguyên nhân #1 gây ra **Flaky Tests** (test gãy liên tục mỗi khi Frontend chỉnh CSS/layout). Thứ tự ưu tiên từ tốt nhất đến hạn chế nhất:

| Thứ tự ưu tiên | Loại Locator | Ví dụ thực tế | Đánh giá & Lưu ý |
| :---: | :--- | :--- | :--- |
| **1 (Tốt nhất)** | **`data-testid` / `id` tĩnh** | `By.cssSelector("[data-testid='checkout-submit']")` hoặc `By.id("voucher-input")` | **Bền vững nhất!** Đừng dùng `id` sinh tự động kiểu `btn-172839` của React/MUI. Hãy đề nghị Dev gắn `data-testid` cố định! |
| **2** | **`name` / `CSS Selector` ngắn gọn** | `By.name("email")`, `By.cssSelector("button[type='submit']")` | Nhanh hơn XPath, cú pháp gọn, không phụ thuộc cấu trúc phân cấp DOM sâu. |
| **3** | **Relative Locators (Selenium 4)** | `RelativeLocator.with(By.tagName("button")).below(By.id("total-price"))` | Tiện khi element không có ID riêng nhưng đứng cạnh nhãn cố định. |
| **4 (Cẩn trọng)** | **XPath tương đối (`//...`)** | `//tr[contains(., 'SALE20')]//button[text()='Áp dụng']` | Mạnh khi cần tìm **ngược lên phần tử cha (`..` / `ancestor::`)** hoặc tìm theo **text hiển thị**, điều mà CSS Selector thuần không làm được. |
| **❌ Tuyệt đối tránh** | **Absolute XPath (`/html/body/div[3]/div[1]/...`)** | Copy thẳng từ Chrome DevTools "Copy full XPath" | Chỉ cần thêm 1 thẻ `<div>` quảng cáo là vỡ 100% test! |

> **Mẹo kiểm chứng Locator ngay trên Chrome DevTools:** Mở tab Console (`F12`), dùng `$$("css-selector")` cho CSS hoặc `$x("//xpath")` cho XPath để chắc chắn kết quả trả về **duy nhất 1 phần tử (`length === 1`)** trước khi dán vào code Java!

---

## 3. Chiến lược Chờ (Wait Strategies) — Chìa khóa trị Flaky Test

**TUYỆT ĐỐI CẤM dùng `Thread.sleep(3000)`** trong framework chuyên nghiệp (làm test vừa chậm vô ích khi mạng nhanh, vừa vẫn fail khi mạng hơi lag ở giây thứ 3.1).

| Loại Wait | Cơ chế hoạt động | Có nên dùng không? | Code chuẩn Selenium 4 (`Duration`) |
| :--- | :--- | :---: | :--- |
| **1. Implicit Wait** | Đặt 1 lần cho cả đời `WebDriver`: mọi `findElement` tự chờ tối đa $X$ giây nếu chưa thấy node trong DOM. | **Nên đặt `Duration.ZERO` (0s)!** | `driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(0));` |
| **2. Explicit Wait (`WebDriverWait`)** | Chỉ chờ tại đúng điểm cần thiết đến khi thỏa điều kiện `ExpectedConditions` (VD: visible, clickable, textPresent). Mặc định poll mỗi `500ms`. | **KHUYÊN DÙNG 100%!** | `new WebDriverWait(driver, Duration.ofSeconds(10)).until(ExpectedConditions.elementToBeClickable(btn));` |
| **3. Fluent Wait (`FluentWait`)** | Phiên bản tùy biến sâu của Explicit Wait: tự chỉnh chu kỳ poll (`pollingEvery`), bỏ qua ngoại lệ cụ thể (`ignoring(StaleElementReferenceException.class)`). | Dùng cho widget AJAX/animation phức tạp | Xem ví dụ bên dưới |

### ⚠️ Cảnh báo phỏng vấn kinh điển: "Tại sao KHÔNG ĐƯỢC trộn Implicit Wait + Explicit Wait?"
Theo tài liệu chính thức của Selenium (`selenium.dev`): Trộn `implicitlyWait(10s)` với `WebDriverWait(15s)` gây ra **hành vi thời gian chờ không lường trước**! Ví dụ: khi bạn dùng `ExpectedConditions.invisibilityOfElementLocated(...)` (chờ spinner biến mất) hoặc kiểm tra element *không* tồn tại, `findElement` bên trong `WebDriverWait` sẽ bị `Implicit Wait` níu giữ thêm 10 giây cho mỗi lần poll, khiến test chờ thành **20 giây hoặc treo lâu hơn nhiều**!
→ **Quy tắc chuẩn:** Tắt `Implicit Wait` (`0s`), chỉ dùng **`Explicit Wait` (`WebDriverWait`)** tập trung trong `BasePage`!

```java
// FluentWait chuẩn Selenium 4 cho spinner tải trang hoặc element hay re-render
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
        .withTimeout(Duration.ofSeconds(15))
        .pollingEvery(Duration.ofMillis(300))
        .ignoring(NoSuchElementException.class)
        .ignoring(StaleElementReferenceException.class);

WebElement orderCodeEl = fluentWait.until(d -> d.findElement(By.cssSelector("[data-testid='order-code']")));
```

---

## 4. Các Thao tác Trình duyệt Quan trọng Cần Nắm

1. **Xử lý Alert / Confirm:** `Alert alert = wait.until(ExpectedConditions.alertIsPresent()); alert.accept();`
2. **Xử lý `<iframe>` (rất hay gặp ở cổng thanh toán VNPAY / Stripe / ReCAPTCHA):**
   ```java
   wait.until(ExpectedConditions.frameToBeAvailableAndSwitchToIt(By.id("payment-iframe")));
   // Thao tác bên trong iframe...
   driver.switchTo().defaultContent(); // Bắt buộc thoát về trang chính sau khi xong!
   ```
3. **Xử lý Tab / Window mới:** Lưu `String mainTab = driver.getWindowHandle()`, sau khi click mở tab mới thì duyệt `driver.getWindowHandles()` và `driver.switchTo().window(newTab)`.
4. **Chuỗi thao tác chuột/bàn phím nâng cao (`Actions`):**
   `new Actions(driver).moveToElement(menuItem).pause(Duration.ofMillis(200)).click(subMenu).perform();`
5. **Headless Mode cho CI/CD (ChromeOptions):**
   ```java
   ChromeOptions options = new ChromeOptions();
   if (Boolean.parseBoolean(System.getProperty("headless", "false"))) {
       options.addArguments("--headless=new", "--window-size=1920,1080", "--no-sandbox", "--disable-dev-shm-usage");
   }
   ```
   *(Lưu ý `--headless=new` dùng engine Chrome thật từ Chrome 109+, tránh lỗi hiển thị của `--headless` cũ; `--no-sandbox` & `--disable-dev-shm-usage` cực kỳ quan trọng khi chạy trong Docker/GitHub Actions!).*

---

## 5. Chụp Screenshot khi Fail (2 Cách: `@AfterMethod` vs. `ITestListener`)

### Cách 1: Trong `@AfterMethod` của `BaseTest` *(theo *Full Stack Testing* Ví dụ 3-11, tr. 48)*
Đơn giản, nhưng trộn logic báo cáo vào class test và phải viết cẩn thận để **luôn chạy `driver.quit()` trước hoặc trong `finally`** nếu lưu ảnh lỗi!

### Cách 2: Tách vào TestNG `ITestListener` + Đính kèm thẳng vào Allure Report *(Chuẩn Framework)*
Tách sạch khỏi code test và tự động chụp cho **mọi** test fail trong toàn bộ suite:

```java
package com.techstore.ui.listeners;

import com.techstore.ui.driver.DriverManager;
import io.qameta.allure.Attachment;
import org.apache.commons.io.FileUtils;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;

import java.io.File;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class ScreenshotFailureListener implements ITestListener {

    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = DriverManager.getDriver();
        if (driver == null) {
            return;
        }
        // 1. Đính kèm thẳng vào Allure Report (dạng byte[])
        attachScreenshotToAllure(driver);

        // 2. Lưu thêm file PNG ra target/screenshots/ để xem nhanh khi chạy local
        saveScreenshotToDisk(driver, result.getMethod().getMethodName());
    }

    @Attachment(value = "Failure Screenshot", type = "image/png")
    public byte[] attachScreenshotToAllure(WebDriver driver) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    private void saveScreenshotToDisk(WebDriver driver, String testName) {
        String ts = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss"));
        File src = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
        File dest = new File("target/screenshots/" + testName + "_" + ts + ".png");
        try {
            FileUtils.copyFile(src, dest);
        } catch (IOException ignored) {
        }
    }
}
```
> **Lưu ý thứ tự chạy với `@AfterMethod`:** Trong TestNG, `ITestListener.onTestFailure()` chạy **TRƯỚC** `@AfterMethod` teardown (`driver.quit()`). Nhờ vậy `DriverManager.getDriver()` vẫn còn sống nguyên vẹn để chụp màn hình trước khi trình duyệt đóng!
