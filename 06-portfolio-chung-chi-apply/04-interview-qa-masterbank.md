# Ngân hàng Câu hỏi Phỏng vấn QA / Automation / SDET / Security QA (26 Câu Trọng tâm)

> **Mục tiêu:** Tổng hợp bộ câu hỏi hay bị hỏi nhất từ Junior đến Mid/Senior SDET kèm **cách trả lời trực diện, có ví dụ thực tế từ TechStore & CRM** (tránh trả lời lý thuyết suông).

---

## Nhóm A: Nền tảng & Test Design *(Hỏi trong 95% buổi phỏng vấn!)*

1. **Phân biệt Severity và Priority? Cho ví dụ 2 trường hợp "lệch pha".**
   - *Trả lời:* **Severity** là mức độ phá hủy kỹ thuật (do QA đề xuất); **Priority** là độ khẩn cấp cần sửa theo lịch phát hành kinh doanh (do PO/Lead chốt).
   - *Ví dụ lệch pha:* (1) **Low Severity + High Priority:** Sai chính tả tên thương hiệu/logo công ty ngay trang chủ trước giờ họp báo ra mắt. (2) **High Severity + Low Priority:** Crash khi xuất báo cáo cũ ở trang Admin mỗi năm dùng 1 lần và đã có nút xuất CSV thay thế.

2. **Phân biệt Smoke Testing, Sanity Testing và Regression Testing?**
   - **Smoke Testing:** Kiểm tra nhanh, diện rộng (*wide & shallow*) các chức năng sống còn sau mỗi bản build mới xem build có đủ ổn để test tiếp không (*"Build verification"*).
   - **Sanity Testing:** Kiểm tra hẹp và sâu (*narrow & deep*) sau khi nhận bản vá bug nhỏ hoặc thay đổi cục bộ, xem phần sửa và vùng lân cận trực tiếp có chạy đúng không.
   - **Regression Testing:** Chạy toàn bộ (hoặc tập hợp rộng) các ca cũ trên toàn hệ thống (thường bằng Automation) để đảm bảo thay đổi mới không phá hỏng tính năng cũ.

3. **Phân biệt Retesting (Confirmation Testing) vs. Regression Testing?**
   - *Retesting* = chạy lại **đúng test case từng bị Fail** để xác nhận bug đó đã hết. *Regression* = chạy lại **các chức năng cũ xung quanh** không liên quan trực tiếp đến bug vừa sửa.

4. **Bài tập tại chỗ kinh điển: *"Hãy thiết kế Test Case cho chức năng Đăng nhập / Đặt hàng / Áp dụng Mã giảm giá (hoặc Cây ATM)"***
   - *Khung trả lời 4 bước ăn điểm tuyệt đối:*
     1. **Hỏi làm rõ yêu cầu trước** (độ dài tối đa, ký tự đặc biệt, có khóa sau 3 lần sai không, có tính Flash Sale không?).
     2. **Chức năng (EP + BVA + Decision Table + State Transition):** Chia phân vùng hợp lệ/lỗi → đánh biên chính xác (`min-1, min, min+1`, `max-1, max, max+1`) → tổ hợp điều kiện.
     3. **Biên giới kỹ thuật & Bảo mật:** `null`, khoảng trắng đầu/cuối, viết hoa/thường email, SQL injection/XSS, double-click (Concurrency), API tampering (`quantity = -1`).
     4. **Phi chức năng & UX:** Thông báo lỗi rõ ràng, phím Tab/Enter, thời gian phản hồi.

5. **Verification khác Validation thế nào? Static Testing có tìm được mọi bug không?**
   - *Verification:* *"Are we building the product right?"* (khớp spec/code). *Validation:* *"Are we building the right product?"* (đúng nhu cầu người dùng).
   - Static Testing (Review, Linter/SonarQube) chỉ tìm trực tiếp Defect tĩnh mà không chạy app; nó không bắt được lỗi hiệu năng, concurrency hay sai nghiệp vụ khi chạy thật (như Lab 04 Ruff báo clean nhưng code vẫn sai `>=` vs `>`).

6. **Sự khác nhau giữa 100% Statement, Branch, Condition và Path Coverage?**
   - `100% Branch ⇒ 100% Statement` (nhưng ngược lại thì KHÔNG). `100% Condition` **không** tự suy ra `100% Branch` (ví dụ `A || B` với `(T,F)` và `(F,T)`). `Path Coverage` xét mọi đường đi khả dĩ ($2^k$ với $k$ `if` nối tiếp), thường thay bằng Basis Path ($V(G)$).

---

## Nhóm B: Java Unit / Integration Test & UI Automation Framework

7. **Phân biệt Stub vs. Mock (và Dummy, Fake, Spy) trong Mockito?**
   - **Stub:** Trả dữ liệu cứng giả định (`given(repo.findById(1L)).willReturn(...)`) để kiểm chứng **trạng thái/đầu ra**.
   - **Mock:** Dùng để kiểm chứng **hành vi tương tác** (`verify(orderRepo, never()).save(any())`).
   - **Spy:** Bọc quanh object thật, chỉ ghi đè một vài hàm bằng `doReturn(...).when(spy)...`. **Fake:** Cài đặt chạy thật rút gọn (VD: `InMemoryRepo`). **Dummy:** Tham số truyền cho đủ chữ ký hàm.

8. **Tại sao đạt 90% Code Coverage trên JaCoCo vẫn có thể lọt Bug lên Production?**
   - (1) Test thiếu assertion (*assertion-free* — chỉ chạy cho xanh dòng); (2) Thiếu ca biên BVA (`>` vs `>=`); (3) Dev quên viết hẳn 1 nhánh yêu cầu (*missing requirement* như quên `startDate` — coverage không thấy thứ chưa viết); (4) Lỗi tích hợp DB/Transaction/Concurrency mà mock che mất. → Giải pháp: Kết hợp **Mutation Testing (PITest)** để kiểm tra xem test có diệt được Mutant không!

9. **Test Pyramid là gì? Nêu 2 Anti-patterns thường gặp và cách tránh.**
   - **Test Pyramid** (Mike Cohn / *Agile Testing* Ch. 14): Nền đáy dày là **Unit & Component Tests**, tầng giữa là **Integration & API Tests**, đỉnh nhỏ nhất là **UI E2E Tests** + **Manual Exploratory**.
   - **Anti-pattern 1 — Ice Cream Cone (Cây kem ngược):** Quá nhiều UI E2E + Manual test, thiếu Unit test → chạy mất hàng giờ, rất flaky.
   - **Anti-pattern 2 — Cupcake:** Dev và QA không phối hợp nên lặp lại y hệt một bộ test validate ở cả 3 tầng Unit, API và UI.

10. **Tại sao không dùng H2 In-Memory DB mà phải dùng Testcontainers trong Java?**
    - H2 khác cú pháp SQL, JSONB, lock dòng (`SELECT FOR UPDATE`), constraint và Migration so với PostgreSQL/MySQL thật. Testcontainers chạy Postgres thật trong Docker qua `@ServiceConnection`, dùng Singleton Container pattern để chỉ boot 1 lần cho cả suite.

11. **Phân biệt `@Mock` (Mockito) và `@MockitoBean` / `@MockBean` (Spring Boot)? Tại sao nên tránh `@DirtiesContext`?**
    - `@Mock` là mock thuần Java không có Spring Context. `@MockitoBean` (thay thế `@MockBean` từ Spring Boot 3.4+/4.x) thay bean ngay bên trong Spring `ApplicationContext` khi chạy `@WebMvcTest` / `@SpringBootTest`.
    - Hạn chế `@DirtiesContext` vì nó hủy Spring Context Cache, buộc test sau phải khởi động lại Spring từ đầu làm chậm CI gấp nhiều lần.

12. **Xử lý Flaky Test như thế nào trong dự án thực tế?**
    - (1) Tuyệt đối không dùng `Thread.sleep` → thay bằng Explicit Wait (`WebDriverWait`) hoặc Playwright Auto-waiting / Web-first assertion; (2) Cô lập dữ liệu từng test (tạo qua API, không dùng chung 1 giỏ hàng khi chạy parallel); (3) Theo dõi Flake Rate qua JUnit XML/Allure, cô lập tạm vào **Quarantine có hạn mức & deadline sửa**, không dùng Retry mù quáng để che bug.

13. **Trong Selenium, tại sao không được trộn Implicit Wait với Explicit Wait? Xử lý `StaleElementReferenceException` ra sao?**
    - Trộn 2 loại Wait gây cộng dồn thời gian chờ bất thường khi check element biến mất/không tồn tại. Cách chuẩn: đặt Implicit Wait = `0s`, chỉ dùng `WebDriverWait`.
    - `StaleElementReferenceException` xảy ra khi DOM vừa bị React/AJAX re-render sau lúc tìm element. Xử lý bằng cách lưu locator `By` (tránh cache `WebElement`), gọi tìm lại trong `ExpectedConditions` hoặc dùng `FluentWait.ignoring(StaleElementReferenceException.class)`.

14. **Làm sao để chạy Selenium TestNG Parallel mà các luồng không đâm vào nhau?**
    - Bọc `WebDriver` trong `ThreadLocal<WebDriver>` (`DriverManager`), cấu hình `parallel="methods"` trong `testng.xml`, cô lập test data riêng cho mỗi test, và luôn gọi `driver.quit()` + `ThreadLocal.remove()` trong `@AfterMethod`.

15. **So sánh Selenium vs. Playwright? Khi nào chọn cái nào?**
    - Playwright kết nối trực tiếp CDP/WebSocket (nhanh hơn), có **Auto-waiting**, **Web-first assertions**, **`storageState`** tái sử dụng login, và **Trace Viewer** debug time-travel rất mạnh → hợp dự án mới cần ít flaky. Selenium hợp hệ thống cũ đã có sẵn bộ Java/TestNG lớn hoặc cần Appium / trình duyệt đặc thù.

16. **Có nên viết Assertion bên trong Page Object (POM) không?**
    - **Không!** (*Full Stack Testing* tr. 46). Page Object chỉ đóng gói locator và hành động trả về trạng thái/dữ liệu; cùng 1 trang `LoginPage` có thể được Assert thành công ở Positive test hoặc Assert báo lỗi ở Negative test.

---

## Nhóm C: API, Database & Microservices

17. **Phân biệt `401 Unauthorized` và `403 Forbidden`? BOLA/IDOR là gì?**
    - `401`: Chưa xác thực (thiếu/sai/hết hạn JWT). `403`: Đã xác thực hợp lệ nhưng không có quyền (role thấp gọi API Admin, hoặc User A truy cập tài nguyên User B — đây chính là lỗ hổng **BOLA/IDOR**, #1 OWASP API Top 10).

18. **Tại sao cần JSON Schema Validation ngoài việc kiểm tra HTTP Status Code?**
    - Đảm bảo hợp đồng dữ liệu không bị phá vỡ ngầm (đổi kiểu `int → string`, thiếu field `required`, hoặc lộ thừa field nhạy cảm như `passwordHash`), áp dụng cho cả Success lẫn Error Response.

19. **Contract Testing (Pact) khác Integration Testing thế nào?**
    - Pact kiểm tra hợp đồng Request/Response giữa Consumer cụ thể và Provider cụ thể **mà không cần 2 service phải chạy chung một môi trường cùng lúc** (Consumer test với Pact Mock xuất file `.json`; Provider verify lại file đó trong CI riêng).

20. **Làm thế nào test lỗi Race Condition (2 người cùng mua sản phẩm cuối cùng `stock = 1`) hoặc kiểm tra Transaction Rollback?**
    - Viết Integration Test với **Testcontainers** + `ExecutorService` + `CountDownLatch` ép 2 thread gọi API đồng thời, kiểm tra đúng 1 HTTP `201`, 1 HTTP `409`, `stock == 0` (không âm). Với DB thủ công, dùng `LEFT JOIN ... WHERE parent.id IS NULL` kiểm tra orphan records và `GROUP BY/HAVING` đối chiếu tổng tiền.

---

## Nhóm D: Agile Testing & Quy trình Thực tế

21. **Tester làm gì trong Scrum? Three Amigos giải quyết vấn đề gì?**
    - Tester tham gia từ Refinement, Three Amigos (PO + Dev + QA chốt AC, ví dụ biên & phân tầng Test Pyramid trước khi code), Pair Testing trong Sprint, Exploratory Testing (Q3), đến Retrospective.

22. **Phân biệt Acceptance Criteria (AC) và Definition of Done (DoD)?**
    - **AC:** Riêng cho **từng User Story** (hành vi nghiệp vụ để PO nghiệm thu). **DoD:** Chung cho **mọi Story/Increment** (code review, `>= 80%` coverage, pass CI, không còn bug High/Critical).

23. **Exploratory Testing khác Ad-hoc Testing ở điểm nào? SBTM vận hành ra sao?**
    - Ad-hoc là bấm bừa ngẫu hứng; Exploratory Testing có **Charter (Sứ mệnh)**, Heuristics (`SFDPOT`, `CRUD`), Time-box `60–90` phút, chấm tỷ lệ **TBS** (`Testing / Bug / Setup`), ghi Mind Map và họp **Debriefing PROOF**.

24. **Khi có một Bug lọt lên Production (Escaped Bug), bạn làm gì?**
    - (1) Phối hợp xác nhận tác động & tái hiện nhanh để hotfix; (2) Tổ chức **RCA (5 Whys)** tìm lỗ hổng quy trình (vì sao lọt qua Unit, API, Exploratory?); (3) Thêm test hồi quy tự động khóa vĩnh viễn bug đó; (4) Cập nhật vào chỉ số **Defect Escape Rate** trong Retrospective.

---

## Nhóm E: Performance & Security *(Điểm nhấn Khác biệt của Bạn!)*

25. **Phân biệt Load, Stress, Spike và Soak Testing? Vì sao đọc `p95`/`p99` quan trọng hơn Average?**
    - *Load:* Tải peak kỳ vọng. *Stress:* Đẩy quá ngưỡng tìm breaking point. *Spike:* Tăng vọt đột ngột (Flash Sale) xem khả năng hồi phục. *Soak:* Ngâm tải nhiều giờ săn Memory/Connection Leak.
    - *Average* bị làm đẹp bởi 95% request nhanh và che giấu 5% request treo; `p95`/`p99` lột trần đúng trải nghiệm tệ nhất của nhóm người dùng đuôi dài.

26. **Trong một dự án có OWASP ZAP chạy CI rồi, tại sao vẫn cần kiểm thử thủ công bằng Burp Suite?**
    - OWASP ZAP (DAST tự động) bắt rất tốt lỗi cấu hình (thiếu CSP/HSTS, cookie thiếu `HttpOnly`, stack trace, SQLi/XSS cơ bản), nhưng **hoàn toàn mù trước lỗi Logic nghiệp vụ** (IDOR/BOLA đọc đơn người khác, sửa `price/quantity`, bỏ qua bước thanh toán, Race Condition voucher) — những lỗi này phải săn bằng **Burp Suite Repeater / Turbo Intruder** và chuyển hóa thành Regression API/Integration test!
