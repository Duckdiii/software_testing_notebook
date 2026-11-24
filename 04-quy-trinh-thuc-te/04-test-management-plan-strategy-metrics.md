# Test Management: Test Strategy vs. Test Plan, Risk-Based Testing, Metrics & TestRail/Zephyr

> **Nguồn tham khảo:** ISTQB CTFL Syllabus v4.0.1 (Section 5.1–5.4), ISO/IEC/IEEE 29119-3, *Agile Testing* (Chương 5 — *Test Strategy vs. Test Planning & Lean Measurements*, tr. 74–88 & Chương 15), TestRail & Zephyr Scale Docs.

---

## 1. Phân biệt Test Strategy vs. Test Plan (Câu hỏi phỏng vấn cực kỳ hay gặp)

| Tiêu chí | Test Strategy (Chiến lược kiểm thử) | Test Plan (Kế hoạch kiểm thử) |
| :--- | :--- | :--- |
| **Bản chất** | Tài liệu **định hướng dài hạn, ít thay đổi (*Static*)**: quy định *cách thức & triết lý* kiểm thử ở cấp Công ty / Toàn bộ Sản phẩm. | Tài liệu **tác nghiệp cụ thể (*Dynamic*)**: quy định *ai làm gì, khi nào, phạm vi nào, môi trường nào* cho một **Dự án / Release / Sprint cụ thể**. |
| **Câu hỏi trả lời** | **HOW & WHY** *(Kiểm thử theo mô hình Test Pyramid thế nào? Chuẩn automation, tiêu chuẩn coverage, quy trình defect chung ra sao?)* | **WHAT, WHO, WHEN, WHERE** *(Release v1.2 này test tính năng gì, không test gì? Ai phụ trách? Lịch chạy UAT & ngày Go-Live?)* |
| **Ai soạn & duy trì?** | QA Lead / Test Manager / Engineering Manager (cùng Senior SDET). | Test Lead / QA Engineer trong team (cập nhật khi scope Release thay đổi). |
| **Trong Agile (*Agile Testing* tr. 86–87)** | Viết **1 lần dùng chung** cho nhiều sprint/release, giúp onboard nhân sự mới và đáp ứng kiểm toán (CMMI/ISO). | Giữ **tinh gọn (Lightweight Test Plan)** — tập trung vào **Rủi ro, Dependencies, Môi trường, Dữ liệu và Exit Criteria** thay vì viết 50 trang Word không ai đọc! |

---

## 2. Risk-Based Testing (RBT — Kiểm thử Dựa trên Rủi ro)

Vì **không thể test vét cạn 100%** (Nguyên lý 2 ISTQB), Risk-Based Testing phân bổ thời gian, độ sâu kỹ thuật và mức độ tự động hóa theo mức độ rủi ro của từng tính năng.

### Công thức đánh giá Rủi ro Sản phẩm (Product Risk)
$$\text{Risk Level} = \text{Likelihood (Xác suất xảy ra lỗi)} \times \text{Impact (Mức độ thiệt hại kinh doanh/kỹ thuật)}$$

- **Likelihood (Xác suất — 1 đến 3 hoặc Low/Med/High):** Code mới hoàn toàn, thuật toán phức tạp ($V(G)$ cao), dev mới viết, tích hợp bên thứ 3, hay có lịch sử nhiều bug cũ.
- **Impact (Tác động — 1 đến 3 hoặc Low/Med/High):** Mất tiền trực tiếp, lộ dữ liệu bảo mật, vi phạm pháp lý, sập toàn hệ thống, không có phương án thay thế (*workaround*).

### Ma trận Hành động theo Mức Rủi ro

| Mức Rủi ro | Điểm `L × I` | Chiến lược kiểm thử tương ứng | Ví dụ trên **CRM** |
| :--- | :---: | :--- | :--- |
| **Critical / High** | `6 – 9` | Test sâu nhất: Decision Table + BVA + State Transition + Unit/Integration Test bắt buộc `>= 85%` + SBTM Exploratory + Regression ưu tiên 1. | **Lead Conversion**, **Phân quyền dữ liệu khách hàng (RBAC/IDOR)**, **Tính doanh thu Deal & Hoa hồng**. |
| **Medium** | `3 – 4` | Phủ EP/BVA cơ bản, API Happy + Negative Path, smoke UI. | Tìm kiếm/lọc nâng cao danh sách Lead, Xuất báo cáo CSV. |
| **Low** | `1 – 2` | Test nhanh một lần hoặc kết hợp khi exploratory; chưa cần đầu tư UI automation phức tạp. | Đổi màu giao diện thông báo, sắp xếp cột bảng phụ. |

---

## 3. Bộ Metrics Chất lượng Chuẩn & Cảnh báo Metric Độc hại

Theo Chương 5 *Agile Testing* (tr. 74–78): *"Tuyệt đối không dùng số lượng bug tìm được để đánh giá hiệu suất cá nhân Tester hay trừng phạt Dev!"* — điều đó chỉ làm team giấu bug hoặc chẻ nhỏ 1 bug thành 5 ticket vô nghĩa. Metrics chỉ có một mục đích: **giúp cả team nhìn thấy điểm yếu quy trình để cải tiến**.

### 4 Chỉ số QA có Giá trị Thực tiễn Cao

| Chỉ số (Metric) | Công thức tính | Ý nghĩa & Ngưỡng tham chiếu |
| :--- | :--- | :--- |
| **1. Defect Escape Rate (DER — Tỷ lệ lọt lỗi)** | $$\text{DER (\%)} = \frac{\text{Bug tìm thấy ở Production}}{\text{Tổng Bug (Pre-release + Production)}} \times 100\%$$ | Đo khả năng "bắt lưới" của toàn bộ quy trình QA/Dev trước khi giao khách. Mục tiêu lành mạnh thường **`< 5%`** (với Critical/High Escape = `0%`). |
| **2. Defect Removal Efficiency (DRE — Hiệu suất loại lỗi)** | $$\text{DRE (\%)} = \frac{\text{Bug bắt được trước khi Release}}{\text{Tổng Bug (Pre-release + Production)}} \times 100\%$$ | Nghịch đảo của DER (`DRE = 100% - DER`). DRE `>= 95%` cho thấy lưới Static + Unit + QA đang làm việc tốt. |
| **3. Defect Density (Mật độ lỗi)** | $$\text{Defect Density} = \frac{\text{Số Bug của Module}}{\text{KLOC hoặc Story Points / Số Test Case}}$$ | Tìm **điểm nóng rủi ro (Bug Clusters — Nguyên lý 4 ISTQB!)**: nếu module `Deal Pipeline` có mật độ bug gấp 4 lần module khác → cần dừng lại refactor & tăng Unit Test! |
| **4. Test Execution & Automation Progress** | `% Passed`, `% Failed`, `% Blocked`, `% Automated Regression` | Theo dõi tiến độ trước ngày Release (`Exit Criteria`) và ngăn chặn nợ kỹ thuật Automation (*Full Stack Testing* tr. 82). |

---

## 4. Công cụ Quản lý Test: TestRail vs. Zephyr Scale

Trong doanh nghiệp, ngoài Jira quản lý Task & Bug, team thường dùng **TestRail** hoặc **Zephyr (Scale/Squad)** để lưu kho Test Case hồi quy và xuất báo cáo nghiệm thu theo từng Release.

| Tiêu chí so sánh | **Zephyr Scale** *(Jira App)* | **TestRail** *(Standalone + Jira Integration)* |
| :--- | :--- | :--- |
| **Nơi lưu trữ dữ liệu** | **Nằm thẳng bên trong Jira** (không cần chuyển tab sang hệ thống ngoài). | Ứng dụng web **độc lập bên ngoài**, kết nối Jira qua App/API webhook. |
| **Truy vết Requirement ↔ Test ↔ Bug** | Cực mượt vì dùng chung Jira Issue Key (Story ↔ Test Case ↔ Test Cycle ↔ Bug). | Tốt (tạo Remote Issue Link từ Test Run sang Jira Ticket; xem được trạng thái Jira Bug ngay trong TestRail). |
| **Quản lý Version Test Case & Tái sử dụng** | Hỗ trợ Version test case, Call-to-Test (tái sử dụng bước Login), BDD Gherkin. | Mạnh về **Shared Test Steps**, **Milestones** đa dự án, báo cáo **Comparison across runs** cực đẹp. |
| **Tích hợp Automation (CI/CD)** | Import kết quả JUnit XML / Cucumber JSON qua **Zephyr REST API** từ GitHub Actions/Jenkins. | Import qua **TestRail CLI (`trcli`)** hoặc REST API đẩy kết quả JUnit XML lên Test Run tự động. |
| **Khi nào chọn công cụ nào?** | Team nhỏ/vừa sống 100% trong hệ sinh thái Atlassian Jira, muốn chi phí gọn và dashboard ngay trên Jira. | Tổ chức lớn có nhiều dự án đa dạng, QA Lead cần báo cáo kiểm toán đa chiều (Milestones, Forecasts). |
