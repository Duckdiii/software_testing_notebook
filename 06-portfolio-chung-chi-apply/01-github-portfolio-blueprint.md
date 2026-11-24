# Chiến lược Xây dựng GitHub Portfolio (Quan trọng hơn cả Chứng chỉ!)

> **Nguyên tắc từ `ROADMAP.md`:** *"Một repo test tốt có sức nặng hơn mười chứng chỉ."* Tech Lead và Hiring Manager thường chỉ dành **30 giây đầu tiên** để lướt qua trang GitHub Profile và `README.md` của bạn trước khi quyết định có mời phỏng vấn kỹ thuật hay không.

---

## 1. Cấu trúc 3 Repository Chuẩn Portfolio (Gom từ GĐ 0 → GĐ 5)

Thay vì để hàng chục repo bài tập rời rạc, hãy ghim (**Pin**) đúng **3 repository chất lượng cao nhất** lên trang chủ GitHub của bạn:

| Thứ tự Pin | Tên Repo đề xuất | Nội dung & Giá trị chứng minh với Nhà tuyển dụng | Nguồn từ lộ trình |
| :-: | :--- | :--- | :--- |
| **1** | **`techstore-test-suite`** *(Repo chủ lực)* | Kết hợp đầy đủ **Test Pyramid** cho TechStore: Unit & Testcontainers Integration (`>= 80%` Branch Coverage) + REST Assured API (JSON Schema, JWT Security) + Selenium/Playwright UI Smoke (`ThreadLocal` parallel) + **GitHub Actions CI + Allure Report**. | GĐ 2 + GĐ 3 + GĐ 5 (`ci-quality-gate.yml`) |
| **2** | **`qa-docs-crm-techstore`** *(Hoặc Notion Public kèm Repo)* | Trưng bày tư duy **Test Design & Quy trình**: Test Design Document + RTM 2 chiều (EP/BVA/Decision Table/State Transition), Test Strategy & Release Test Plan cho CRM, Exploratory SBTM Session Report, và mẫu Bug Report chuẩn. | GĐ 0 + GĐ 1 + GĐ 4 |
| **3** | **`techstore-security-perf-lab`** | Điểm khác biệt **Security + Performance QA**: Script Grafana k6 (Load/Stress/Spike/Soak) + Performance Tuning Report (trị N+1 & HikariCP) + OWASP ZAP Automation + Burp Suite Pentest Findings Report (`SEC-01..06`). | GĐ 5 (Deliverable 2 & 3) |

---

## 2. Tiêu chuẩn Vàng của một File `README.md` Repo Kiểm thử

Một `README.md` chuyên nghiệp phải trả lời trọn vẹn **5 câu hỏi** ngay khi mở trang GitHub:

1. **Badges & Elevator Pitch (3 dòng đầu):** Repo này test hệ thống nào? Dùng công cụ gì? Đang có badge CI xanh hay không?
2. **Kiến trúc & Phân tầng Test Pyramid (Sơ đồ Mermaid):** Giải thích vì sao bạn dồn 65% test ở Unit/Integration, 25% ở API và chỉ giữ ~10% Critical Flows ở UI!
3. **Điểm nhấn Kỹ thuật (Engineering Highlights):** Nêu rõ các bài toán khó đã giải quyết (VD: *Testcontainers Singleton + `@ServiceConnection`, bắt Race Condition `stock = 1` bằng `CountDownLatch`, Selenium `ThreadLocal` parallel, Allure Report tự đính kèm Screenshot + HTTP Request/Response*).
4. **Cách chạy chỉ bằng 1–2 câu lệnh (Quick Start):** `git clone` → `docker compose up -d --wait` → `mvn clean test` → `mvn allure:serve`.
5. **Bằng chứng Kết quả Trực quan (Visual Proof):** Ảnh chụp màn hình thực tế của **Allure Dashboard**, **JaCoCo HTML Report (`>= 80%`)**, **k6 Summary**, và **OWASP ZAP Report**.

### 5 "Cờ Đỏ" (Red Flags) cần dọn sạch trước khi gửi link GitHub cho HR/Tech Lead
- ❌ Commit file chứa mật khẩu, JWT secret, hoặc API key thật lên Git (phải dùng `.env.example` và GitHub Secrets).
- ❌ `README.md` trống trơn chỉ có tiêu đề mặc định hoặc không có hướng dẫn chạy.
- ❌ Ghi trên README là *"có dùng Docker / Allure / k6"* nhưng trong repo **không có** `docker-compose.yml` hay file config nào chạy được thật!
- ❌ Commit thư mục `target/`, `node_modules/`, `.idea/`, hoặc ảnh rác không tổ chức.
- ❌ Đẩy toàn bộ code copy từ tutorial mà không tùy biến theo tên dự án/nghiệp vụ của mình.

---

## 3. Chiến lược "3-Minute Walkthrough" khi Phỏng vấn trực tiếp (CAR Framework)

Khi Tech Lead hỏi: *"Em hãy giới thiệu 1 dự án kiểm thử trên GitHub mà em tâm đắc nhất"*, đừng mở từng file code đọc dòng lệnh — hãy trình bày đúng **3 phút theo mô hình C–A–R**:

1. **Context (30 giây — Bối cảnh & Bài toán):**
   > *"Em xây dựng `techstore-test-suite` cho hệ thống E-Commerce TechStore (Spring Boot + PostgreSQL) nhằm giải quyết bài toán kiểm thử tự động đa tầng thay cho việc test tay lặp lại hoặc viết quá nhiều UI test chậm và dễ gãy."*
2. **Action (90 giây — Kiến trúc & Kỹ thuật đã áp dụng):**
   > *"Em thiết kế theo đúng **Test Pyramid**: ở tầng đáy, em viết JUnit 5 + Mockito + Testcontainers PostgreSQL để kiểm thử các rule tính giảm giá Voucher, loại trừ Flash Sale và dùng `CountDownLatch` kiểm chứng race condition tồn kho `stock = 1`, đạt trên **80% branch coverage** có kiểm chứng bằng **PITest mutation testing**. Ở tầng giữa, em dùng REST Assured kiểm tra HTTP contract bằng JSON Schema và ma trận bảo mật JWT/IDOR. Ở tầng đỉnh, em chỉ giữ các luồng mua hàng sống còn bằng Selenium 4 POM với `ThreadLocal` chạy song song và tự động gắn lên **GitHub Actions** kèm k6 Smoke, OWASP ZAP Baseline và xuất **Allure Report**."*
3. **Result (60 giây — Kết quả & Giá trị mang lại):**
   > *"Nhờ phân tầng đúng, toàn bộ bộ Unit + API chạy dưới 1 phút cho phản hồi tức thì ở mỗi PR, phát hiện sớm các lỗi biên `startDate`, lỗ hổng IDOR xem đơn người khác và nút nghẽn N+1 query khiến `p95` giảm từ 780ms xuống 128ms."*
