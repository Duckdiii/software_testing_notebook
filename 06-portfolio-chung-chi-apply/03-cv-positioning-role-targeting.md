# Định vị CV & Chiến lược Ứng tuyển 4 Vị trí Mục tiêu

> **Chiến lược cốt lõi (`ROADMAP.md` GĐ 6):** Tuyệt đối không định vị mình là *"Fresher Tester"* — hãy định vị là **"Software Engineer với nền tảng QA, Automation & Security"**. Đây là lợi thế cạnh tranh rất hiếm giúp bạn vượt trội trên thị trường tuyển dụng.

---

## 1. Phân tích Lợi thế Cạnh tranh Độc nhất của Bạn (UVP)

| Điểm mạnh bạn đang có | Tại sao nhà tuyển dụng cực kỳ chuộng? | Tránh cách viết yếu → Nên viết thế nào? |
| :--- | :--- | :--- |
| **Lập trình Java / Spring Boot** | Biết đọc source code, viết JUnit 5, Mockito, Testcontainers, tìm bug race condition & transaction rollback mà tester thuần không thấy. | ❌ *"Biết Java cơ bản"* <br> ✅ *"Engineered JUnit 5, Mockito & Testcontainers integration suites achieving >= 80% branch coverage (verified with PITest mutation testing)."* |
| **Kinh nghiệm Scrum Master (CRM)** | Hiểu quy trình Agile thực tế, điều phối Three Amigos, thiết lập Definition of Done, xử lý Triage & RCA. | ❌ *"Làm Scrum Master trong project nhóm"* <br> ✅ *"Facilitated Three Amigos & Sprint Quality Gates; authored ISO 29119-aligned Test Strategy & SBTM reports reducing escaped defects."* |
| **Nền tảng Security (Burp Suite, WebGoat, Juice Shop, OWASP ZAP)** | Tự kiểm thử IDOR/BOLA, JWT, Injection và tích hợp DAST tự động vào CI/CD. | ❌ *"Có tìm hiểu Burp Suite"* <br> ✅ *"Conducted OWASP Top 10:2025 security testing (Burp Suite & OWASP ZAP DAST CI), uncovering IDOR/BOLA and race-condition vulnerabilities."* |

---

## 2. Tùy biến Hồ sơ theo 4 Vị trí Mục tiêu (Role Targeting Matrix)

Không dùng 1 CV duy nhất gửi cho mọi JD! Giữ nguyên lõi kỹ năng, chỉ **đảo thứ tự ưu tiên & tinh chỉnh Summary** theo đúng vai trò JD đăng tuyển:

| Vị trí ứng tuyển | Trọng tâm nhà tuyển dụng tìm kiếm | Phần cần đẩy lên đầu CV của bạn |
| :--- | :--- | :--- |
| **1. SDET** *(Software Development Engineer in Test — Lương & trần phát triển cao nhất)* | Code Java chắc, Unit/Integration Test, Testcontainers, REST Assured, Selenium/Playwright, CI/CD Quality Gates. | **TechStore Test Suite**: Test Pyramid (`>= 80%` Branch + PITest + Testcontainers Concurrency + REST Assured + GitHub Actions CI). |
| **2. Automation Tester / QA Automation** | REST Assured / Postman, Selenium POM (`ThreadLocal` Parallel), Playwright, Allure Report, Data-driven. | **API + UI Framework**: `techstore-automation-suite` (JUnit 5 API + TestNG Parallel Selenium + Playwright Hybrid + Allure). |
| **3. Security QA / AppSec QA** | OWASP Top 10, Burp Suite, WebGoat/Juice Shop, JWT/BOLA Negative API, OWASP ZAP DAST CI. | **TechStore Security & Perf Lab**: Burp Repeater/Turbo Intruder PoCs + ZAP GitHub Actions DAST + REST Assured JWT security matrix. |
| **4. QA/QC Engineer** *(Middle/Junior thiên quy trình + Full-stack QA)* | Test Design (EP, BVA, Decision Table, State), RTM, Test Plan/Strategy, SBTM, Jira/JQL, SQL DB Testing, ISTQB CTFL. | **CRM & TechStore QA Docs**: Test Design Doc + RTM + CRM Test Plan + SBTM Report + SQL Integrity Checks + Chứng chỉ ISTQB CTFL. |

---

## 3. Công thức Viết Bullet Points Đạt Chuẩn ATS & Gây Ấn Tượng

Áp dụng công thức **Google XYZ / CAR**: **`[Động từ hành động mạnh] + [Công việc & Công cụ cụ thể] + [Kết quả đo lường được]`**.

### Ví dụ chuyển đổi Bullet Points trên CV:
- **Test Design & Documentation:**
  > *"Designed 45+ risk-based test cases using EP, BVA, Decision Table, and State Transition with a 100% bidirectional Requirements Traceability Matrix (RTM), identifying 5 critical business-logic & concurrency defects before release."*
- **Java Unit & Integration Testing:**
  > *"Built automated Unit and Integration tests with JUnit 5, Mockito 5, AssertJ, and Testcontainers (PostgreSQL), elevating checkout module branch coverage to 93.9% with JaCoCo & PITest quality gates."*
- **API & UI Automation + CI/CD:**
  > *"Architected a hybrid API & UI automation framework (REST Assured, JSON Schema validation, Selenium 4 POM with ThreadLocal parallel execution, and Playwright) integrated into GitHub Actions CI with unified Allure Reports."*
- **Performance & Security QA:**
  > *"Executed k6 load/spike tests (300 VUs) pinpointing Hibernate N+1 queries and HikariCP bottlenecks (cutting p95 latency from 780ms to 128ms), alongside OWASP ZAP CI scans and Burp Suite IDOR assessments."*

---

## 4. Từ khóa ATS Bắt buộc phải Có (Checklist Rà soát CV)

- **Core QA & Standards:** `ISTQB CTFL`, `Test Design (EP, BVA, Decision Table, State Transition)`, `RTM`, `Test Strategy & Test Plan`, `Exploratory Testing (SBTM)`, `Root Cause Analysis (RCA)`.
- **Java & Automation Stack:** `Java`, `JUnit 5`, `TestNG`, `Mockito`, `AssertJ`, `Spring Boot Test`, `Testcontainers`, `JaCoCo`, `PITest`, `TDD`, `REST Assured`, `JSON Schema`, `Postman/Newman`, `Selenium WebDriver`, `Page Object Model (POM)`, `Playwright`, `Allure Report`.
- **DB, Non-Functional & DevOps:** `PostgreSQL / SQL`, `Flyway`, `Grafana k6`, `JMeter`, `OWASP Top 10`, `Burp Suite`, `OWASP ZAP`, `WCAG / axe-core / Lighthouse`, `GitHub Actions`, `Docker Compose`, `Jira / JQL`, `TestRail / Zephyr`, `Agile / Scrum`.
