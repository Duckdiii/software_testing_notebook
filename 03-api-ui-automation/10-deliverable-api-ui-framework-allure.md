# Deliverable Giai đoạn 3: Framework Automation API + UI + Allure Report (`techstore-automation-suite`)

> **Mục tiêu Deliverable (Roadmap GĐ 3):** Một **framework automation hoàn chỉnh (API + UI)** tích hợp báo cáo **Allure Report**, sẵn sàng đưa lên GitHub Portfolio, thể hiện tư duy kiến trúc SDET chuẩn mực (**giữ UI ở đỉnh Test Pyramid**, API làm chủ lực hồi quy).

---

## 1. Kiến trúc Repository chuẩn Portfolio (`techstore-automation-suite`)

```text
techstore-automation-suite/
├── pom.xml                                  # Maven Surefire chia 2 execution rõ ràng (JUnit 5 API + TestNG UI) + Allure
├── README.md                                # Hướng dẫn cài đặt, kiến trúc Pyramid, lệnh chạy & ảnh chụp Allure mẫu
├── .github/workflows/ci-automation.yml      # (Kết nối GĐ 5) Chạy API + Headless UI và xuất Allure Report Artifact
└── src/
    ├── main/java/com/techstore/framework/
    │   ├── config/ConfigManager.java        # Đọc config.properties + ghi đè qua System Properties (-Denv, -Dheadless)
    │   ├── api/
    │   │   ├── ApiSpecFactory.java          # Tạo RequestSpecification / ResponseSpecification + AllureRestAssured filter
    │   │   └── AuthClient.java              # Helper lấy JWT Token cho cả API Test & UI Hybrid Seeding
    │   └── ui/
    │       ├── driver/DriverManager.java    # ThreadLocal<WebDriver> (Chrome/Firefox headless + Selenium Manager)
    │       └── pages/
    │           ├── BasePage.java            # Đóng gói WebDriverWait (Implicit = 0s), click, type, getText
    │           ├── LoginPage.java           # Page Object đăng nhập (Fluent chaining -> ProductCatalogPage)
    │           └── CheckoutPage.java        # Page Object áp voucher & đặt hàng (KHÔNG chứa Assertion!)
    └── test/
        ├── java/com/techstore/tests/
        │   ├── api/
        │   │   ├── BaseApiTest.java         # Setup BaseURI, Specs, JWT Token cache
        │   │   ├── CheckoutApiTest.java     # JUnit 5 + REST Assured: EP/BVA, JSON Schema, Negative, Error Schema
        │   │   └── SecurityJwtApiTest.java  # JUnit 5 + REST Assured: 401/403, tampered JWT, BOLA/IDOR
        │   └── ui/
        │       ├── BaseUiTest.java          # TestNG @BeforeMethod/@AfterMethod quản lý DriverManager + API seed
        │       ├── AllureUiListener.java    # TestNG ITestListener tự động đính kèm Screenshot + Page Source khi Fail!
        │       └── CheckoutUiCriticalTest.java # TestNG: Chỉ 2–3 Critical User Journeys trên đỉnh Pyramid
        └── resources/
            ├── config/staging.properties
            ├── schemas/
            │   ├── order-response.schema.json
            │   └── error-response.schema.json
            ├── allure.properties            # allure.results.directory=target/allure-results
            ├── environment.properties       # Hiển thị Browser, Env, BaseURL trên Allure Dashboard
            └── testng-ui.xml                # Cấu hình TestNG parallel="methods" + AllureUiListener
```

> **Lưu ý kiến trúc quan trọng trong `pom.xml`:** Khi kết hợp **JUnit 5 (cho API Tests)** và **TestNG (cho UI Tests)** trong cùng một dự án Maven, nếu bạn khai báo `<suiteXmlFiles>` ở cấp `<configuration>` chung của `maven-surefire-plugin`, Surefire sẽ chỉ chạy TestNG và **bỏ qua toàn bộ JUnit 5 test**! Cách giải quyết chuẩn là tách **2 `<execution>` riêng** trong `maven-surefire-plugin` — cả hai cùng đổ kết quả JSON về `target/allure-results` để hợp nhất thành **1 Allure Dashboard duy nhất**!

---

## 2. File `pom.xml` Hoàn chỉnh (JUnit 5 API + TestNG UI + Allure Report)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.techstore</groupId>
    <artifactId>techstore-automation-suite</artifactId>
    <version>1.0.0</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>

        <junit.version>5.11.4</junit.version>
        <testng.version>7.10.2</testng.version>
        <rest-assured.version>5.5.0</rest-assured.version>
        <selenium.version>4.27.0</selenium.version>
        <assertj.version>3.26.3</assertj.version>
        <jackson.version>2.18.2</jackson.version>
        <allure.version>2.29.0</allure.version>
        <allure-maven.version>2.15.2</allure-maven.version>
        <aspectj.version>1.9.22.1</aspectj.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-bom</artifactId>
                <version>${allure.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- UI Automation -->
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

        <!-- API Automation -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>${rest-assured.version}</version>
        </dependency>
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>json-schema-validator</artifactId>
            <version>${rest-assured.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Utilities & Assertions -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>${assertj.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Allure Adapters (JUnit 5 + TestNG + REST Assured) -->
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-junit5</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-testng</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.qameta.allure</groupId>
            <artifactId>allure-rest-assured</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Copy environment.properties sang target/allure-results để hiện widget Environment trên Allure -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>3.3.1</version>
                <executions>
                    <execution>
                        <id>copy-allure-environment</id>
                        <phase>process-test-resources</phase>
                        <goals><goal>copy-resources</goal></goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/allure-results</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/test/resources</directory>
                                    <includes><include>environment.properties</include></includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- Tách 2 execution: 1) API Tests bằng JUnit 5, 2) UI Tests bằng TestNG XML -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.2</version>
                <configuration>
                    <argLine>
                        -Dfile.encoding=UTF-8
                        -javaagent:"${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar"
                    </argLine>
                    <systemPropertyVariables>
                        <allure.results.directory>${project.build.directory}/allure-results</allure.results.directory>
                    </systemPropertyVariables>
                </configuration>
                <executions>
                    <!-- Tắt default-test để không chạy trùng -->
                    <execution>
                        <id>default-test</id>
                        <phase>none</phase>
                    </execution>
                    <!-- 1. Chạy tầng API trước (nhanh, tạo feedback ngay) -->
                    <execution>
                        <id>api-junit5-tests</id>
                        <phase>test</phase>
                        <goals><goal>test</goal></goals>
                        <configuration>
                            <includes>
                                <include>**/api/**/*Test.java</include>
                            </includes>
                        </configuration>
                        <dependencies>
                            <dependency>
                                <groupId>org.apache.maven.plugins</groupId>
                                <artifactId>surefire-junit-platform</artifactId>
                                <version>3.5.2</version>
                            </dependency>
                        </dependencies>
                    </execution>
                    <!-- 2. Chạy tầng UI bằng TestNG suite XML -->
                    <execution>
                        <id>ui-testng-tests</id>
                        <phase>test</phase>
                        <goals><goal>test</goal></goals>
                        <configuration>
                            <suiteXmlFiles>
                                <suiteXmlFile>src/test/resources/testng-ui.xml</suiteXmlFile>
                            </suiteXmlFiles>
                        </configuration>
                        <dependencies>
                            <dependency>
                                <groupId>org.apache.maven.plugins</groupId>
                                <artifactId>surefire-testng</artifactId>
                                <version>3.5.2</version>
                            </dependency>
                        </dependencies>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.aspectj</groupId>
                        <artifactId>aspectjweaver</artifactId>
                        <version>${aspectj.version}</version>
                    </dependency>
                </dependencies>
            </plugin>

            <!-- Tạo & mở Allure Report bằng mvn allure:report / mvn allure:serve -->
            <plugin>
                <groupId>io.qameta.allure</groupId>
                <artifactId>allure-maven</artifactId>
                <version>${allure-maven.version}</version>
                <configuration>
                    <reportVersion>${allure.version}</reportVersion>
                    <resultsDirectory>${project.build.directory}/allure-results</resultsDirectory>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 3. Mã nguồn Cốt lõi của Framework (`src/main` & `src/test`)

### 3.1. Quản lý Cấu hình & API Client (`ConfigManager.java`, `ApiSpecFactory.java`, `AuthClient.java`)

```java
// src/main/java/com/techstore/framework/config/ConfigManager.java
package com.techstore.framework.config;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public final class ConfigManager {
    private static final Properties PROPS = new Properties();

    static {
        String env = System.getProperty("env", "staging");
        String path = "config/" + env + ".properties";
        try (InputStream is = ConfigManager.class.getClassLoader().getResourceAsStream(path)) {
            if (is != null) {
                PROPS.load(is);
            }
        } catch (IOException e) {
            throw new IllegalStateException("Cannot load config file: " + path, e);
        }
    }

    private ConfigManager() {}

    public static String get(String key) {
        // Ưu tiên System Property (-Dapi.baseUrl=...) ghi đè file properties!
        return System.getProperty(key, PROPS.getProperty(key));
    }

    public static boolean getBoolean(String key, boolean defaultVal) {
        String val = get(key);
        return val == null ? defaultVal : Boolean.parseBoolean(val);
    }
}
```

```java
// src/main/java/com/techstore/framework/api/ApiSpecFactory.java
package com.techstore.framework.api;

import com.techstore.framework.config.ConfigManager;
import io.qameta.allure.restassured.AllureRestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.http.ContentType;
import io.restassured.specification.RequestSpecification;

public final class ApiSpecFactory {
    private ApiSpecFactory() {}

    public static RequestSpecification baseSpec() {
        return new RequestSpecBuilder()
                .setBaseUri(ConfigManager.get("api.baseUrl"))
                .setBasePath("/api/v1")
                .setContentType(ContentType.JSON)
                .setAccept(ContentType.JSON)
                .addFilter(new AllureRestAssured()) // Ghi full Request/Response vào Allure!
                .build();
    }

    public static RequestSpecification authenticatedSpec(String jwtToken) {
        return new RequestSpecBuilder()
                .addRequestSpecification(baseSpec())
                .addHeader("Authorization", "Bearer " + jwtToken)
                .build();
    }
}
```

```java
// src/main/java/com/techstore/framework/api/AuthClient.java
package com.techstore.framework.api;

import io.qameta.allure.Step;
import io.restassured.RestAssured;
import java.util.Map;

public final class AuthClient {
    private AuthClient() {}

    @Step("Đăng nhập qua API với tài khoản {email}")
    public static String loginAndGetToken(String email, String password) {
        return RestAssured.given()
                .spec(ApiSpecFactory.baseSpec())
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

### 3.2. `DriverManager.java` & Page Objects (`BasePage.java`, `LoginPage.java`, `CheckoutPage.java`)

```java
// src/main/java/com/techstore/framework/ui/driver/DriverManager.java
package com.techstore.framework.ui.driver;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.firefox.FirefoxOptions;
import java.time.Duration;

public final class DriverManager {
    private static final ThreadLocal<WebDriver> DRIVER = new ThreadLocal<>();

    private DriverManager() {}

    public static void initDriver(String browser, boolean headless) {
        WebDriver webDriver = switch (browser.toLowerCase()) {
            case "firefox" -> {
                FirefoxOptions opts = new FirefoxOptions();
                if (headless) opts.addArguments("-headless");
                yield new FirefoxDriver(opts);
            }
            default -> {
                ChromeOptions opts = new ChromeOptions();
                if (headless) {
                    opts.addArguments("--headless=new", "--window-size=1920,1080",
                            "--no-sandbox", "--disable-dev-shm-usage");
                }
                yield new ChromeDriver(opts); // Selenium Manager tự quản lý binary!
            }
        };
        webDriver.manage().timeouts().implicitlyWait(Duration.ofSeconds(0));
        if (!headless) webDriver.manage().window().maximize();
        DRIVER.set(webDriver);
    }

    public static WebDriver getDriver() {
        return DRIVER.get();
    }

    public static void quitDriver() {
        WebDriver d = DRIVER.get();
        if (d != null) {
            try {
                d.quit();
            } finally {
                DRIVER.remove();
            }
        }
    }
}
```

```java
// src/main/java/com/techstore/framework/ui/pages/CheckoutPage.java
package com.techstore.framework.ui.pages;

import io.qameta.allure.Step;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;

public class CheckoutPage extends BasePage {

    private final By voucherInput = By.cssSelector("[data-testid='voucher-input']");
    private final By applyVoucherBtn = By.cssSelector("[data-testid='apply-voucher-btn']");
    private final By discountText = By.cssSelector("[data-testid='discount-amount']");
    private final By shippingFeeText = By.cssSelector("[data-testid='shipping-fee']");
    private final By finalTotalText = By.cssSelector("[data-testid='final-total']");
    private final By placeOrderBtn = By.cssSelector("[data-testid='place-order-btn']");
    private final By orderStatusBadge = By.cssSelector("[data-testid='order-status']");
    private final By voucherErrorAlert = By.cssSelector("[data-testid='voucher-error']");

    public CheckoutPage(WebDriver driver) {
        super(driver);
    }

    @Step("Áp dụng mã giảm giá '{code}'")
    public CheckoutPage applyVoucher(String code) {
        type(voucherInput, code);
        click(applyVoucherBtn);
        return this;
    }

    @Step("Nhấn nút Đặt hàng")
    public CheckoutPage submitOrder() {
        click(placeOrderBtn);
        return this;
    }

    public String getDiscountAmountText() { return getText(discountText); }
    public String getShippingFeeText() { return getText(shippingFeeText); }
    public String getFinalTotalText() { return getText(finalTotalText); }
    public String getOrderStatus() { return getText(orderStatusBadge); }
    public String getVoucherErrorMessage() { return getText(voucherErrorAlert); }
}
```

### 3.3. Test Suite API (JUnit 5) & UI (TestNG + Allure Listener)

```java
// src/test/java/com/techstore/tests/ui/AllureUiListener.java
package com.techstore.tests.ui;

import com.techstore.framework.ui.driver.DriverManager;
import io.qameta.allure.Attachment;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;
import org.testng.ITestListener;
import org.testng.ITestResult;

import java.nio.charset.StandardCharsets;

public class AllureUiListener implements ITestListener {

    @Override
    public void onTestFailure(ITestResult result) {
        WebDriver driver = DriverManager.getDriver();
        if (driver != null) {
            attachScreenshot(driver);
            attachCurrentUrl(driver.getCurrentUrl());
            attachPageSource(driver.getPageSource());
        }
    }

    @Attachment(value = "Screenshot on Failure", type = "image/png")
    public byte[] attachScreenshot(WebDriver driver) {
        return ((TakesScreenshot) driver).getScreenshotAs(OutputType.BYTES);
    }

    @Attachment(value = "Failed URL", type = "text/plain")
    public String attachCurrentUrl(String url) {
        return url;
    }

    @Attachment(value = "Page DOM Snapshot", type = "text/html")
    public byte[] attachPageSource(String html) {
        return html.getBytes(StandardCharsets.UTF_8);
    }
}
```

```java
// src/test/java/com/techstore/tests/ui/CheckoutUiCriticalTest.java
package com.techstore.tests.ui;

import com.techstore.framework.config.ConfigManager;
import com.techstore.framework.ui.driver.DriverManager;
import com.techstore.framework.ui.pages.CheckoutPage;
import com.techstore.framework.ui.pages.LoginPage;
import io.qameta.allure.*;
import org.testng.Assert;
import org.testng.annotations.Test;

@Epic("TechStore E2E Critical Flows")
@Feature("UI Checkout Flow (Top of Pyramid)")
public class CheckoutUiCriticalTest extends BaseUiTest {

    @Test(groups = {"smoke", "ui"})
    @Severity(SeverityLevel.BLOCKER)
    @Story("Khách hàng đăng nhập, áp mã SALE20 và đặt hàng thành công")
    @Description("Luồng UI sống còn duy nhất cần bảo vệ ở đỉnh Test Pyramid — mọi ca biên chi tiết đã chạy ở API & Unit Test.")
    public void verifyCriticalPurchaseFlowWithSale20() {
        DriverManager.getDriver().get(ConfigManager.get("ui.baseUrl") + "/login");

        CheckoutPage checkout = new LoginPage(DriverManager.getDriver())
                .loginSuccess("buyer@techstore.vn", ConfigManager.get("buyer.password"))
                .addSampleProductAndGoToCheckout("SKU-LAPTOP-500K")
                .applyVoucher("SALE20");

        Assert.assertEquals(checkout.getDiscountAmountText(), "-100.000đ");
        Assert.assertEquals(checkout.getShippingFeeText(), "30.000đ");
        Assert.assertEquals(checkout.getFinalTotalText(), "430.000đ");

        checkout.submitOrder();
        Assert.assertEquals(checkout.getOrderStatus(), "PENDING_PAYMENT");
    }

    @Test(groups = {"regression", "ui"})
    @Severity(SeverityLevel.NORMAL)
    @Story("Hiển thị thông báo lỗi trên UI khi áp dụng mã giảm giá không đủ điều kiện")
    public void verifyVoucherErrorBannerDisplayedOnUi() {
        DriverManager.getDriver().get(ConfigManager.get("ui.baseUrl") + "/login");

        CheckoutPage checkout = new LoginPage(DriverManager.getDriver())
                .loginSuccess("buyer@techstore.vn", ConfigManager.get("buyer.password"))
                .addSampleProductAndGoToCheckout("SKU-MOUSE-300K")
                .applyVoucher("SALE20");

        Assert.assertTrue(checkout.getVoucherErrorMessage().contains("500.000đ"),
                "Thông báo lỗi tối thiểu 500k phải hiển thị rõ cho người dùng!");
    }
}
```

### 3.4. File cấu hình TestNG & Allure Resources

```xml
<!-- src/test/resources/testng-ui.xml -->
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="TechStore UI Critical Suite" parallel="methods" thread-count="2">
    <listeners>
        <listener class-name="com.techstore.tests.ui.AllureUiListener"/>
    </listeners>
    <test name="Chrome Headless Critical Journeys">
        <parameter name="browser" value="chrome"/>
        <classes>
            <class name="com.techstore.tests.ui.CheckoutUiCriticalTest"/>
        </classes>
    </test>
</suite>
```

```properties
# src/test/resources/environment.properties
Project=TechStore E-Commerce
Test.Architecture=API (JUnit 5 + REST Assured) + UI (TestNG + Selenium 4 POM)
Browser=Chrome Headless
Java.Version=21
Environment=Staging
```

---

## 4. Vận hành & Xem Báo cáo Allure Report

### Các lệnh Maven thực thi hàng ngày
```bash
# 1. Chạy toàn bộ API + UI Tests (sinh kết quả vào target/allure-results/)
mvn clean test -Denv=staging -Dheadless=true

# 2. Chỉ chạy nhanh tầng API hoặc chỉ tầng UI bằng execution ID của Maven
mvn test-compile surefire:test@api-junit5-tests
mvn test-compile surefire:test@ui-testng-tests -Dheadless=true

# 3. Sinh & tự động mở Allure Dashboard trên trình duyệt
mvn allure:serve
# Hoặc xuất bản static HTML report (đưa lên GitHub Pages / CI Artifacts):
mvn allure:report
```

### Cách đọc 5 Tab giá trị nhất trên Allure Dashboard (để trình bày khi phỏng vấn)
1. **Overview:** Tổng quan tỷ lệ Pass/Fail/Broken, widget **Environment** (đọc từ `environment.properties`), phân bổ mức độ nghiêm trọng (`Severity`: Blocker/Critical/Normal).
2. **Behaviors (`@Epic` → `@Feature` → `@Story`):** Gộp chung cả **API Tests** và **UI Tests** dưới cùng một Feature nghiệp vụ `Checkout`, giúp PM/PO nhìn thấy ngay tính năng Checkout đang xanh ở API hay vỡ ở UI!
3. **Suites:** Xem thời gian chạy từng tầng — chứng minh **Test Pyramid khỏe mạnh** (ví dụ: 18 API tests chạy trong `2.1s`, 2 UI tests chạy trong `7.5s`).
4. **Timeline:** Biểu đồ Gantt trực quan hóa 2 thread UI chạy **song song thực sự (`Thread 1` & `Thread 2`)**.
5. **Test Body → Attachments:** Khi Test API fail → bấm xem ngay **Request & Response HTTP** do `AllureRestAssured` tự đính kèm; khi Test UI fail → bấm xem ngay **Screenshot PNG**, **Failed URL** và **HTML DOM Snapshot**!

---

## 5. Checklist Nghiệm thu Deliverable Giai đoạn 3 (Đưa lên GitHub Portfolio)

- [x] **Tuân thủ Test Pyramid:** Tỷ trọng test API (EP/BVA, JSON Schema, JWT Negative, BOLA) áp đảo; UI chỉ giữ Critical Flows (`smoke`).
- [x] **Tách lớp sạch sẽ:** Page Objects không chứa `Assert`, không chứa dữ liệu cứng; Test không chứa `By` locator hay `Thread.sleep`.
- [x] **Thread-safe Parallelism:** Dùng `ThreadLocal<WebDriver>` + `DRIVER.remove()`, không có biến `static WebDriver` dùng chung.
- [x] **An toàn bảo mật Git:** Không commit password/token thật trong repo (`ConfigManager` đọc từ System Property / CI Secrets).
- [x] **Allure Report hoàn chỉnh:** Có `AllureRestAssured` cho API, `AllureUiListener` chụp Screenshot khi UI fail, `@Epic/@Feature/@Step`, và `environment.properties`.
- [x] **README.md của Repo rõ ràng:** Có sơ đồ kiến trúc, yêu cầu cài đặt, lệnh chạy từng tầng, và ảnh chụp màn hình Allure Dashboard mẫu.
