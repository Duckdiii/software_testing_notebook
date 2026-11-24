# Hướng dẫn Ôn thi Chứng chỉ ISTQB CTFL v4.0.1 (Foundation Level)

> **Nguồn tham khảo:** ISTQB Certified Tester Foundation Level (CTFL) Syllabus v4.0.1 (`istqb.org`), toàn bộ tài liệu Giai đoạn 0 (`00-nen-tang/`) và Giai đoạn 1 (`01-test-design/`).

---

## 1. Cấu trúc Đề thi & Quy tắc Quan trọng (ISTQB CTFL v4.0 / v4.0.1)

| Thông số kỳ thi | Quy định chính thức | Lưu ý chiến lược cho bạn |
| :--- | :--- | :--- |
| **Số lượng câu hỏi** | **40 câu trắc nghiệm** (mỗi câu 1 điểm, chỉ 1 đáp án đúng — trừ một số câu ghi rõ *Select TWO*). | Không trừ điểm câu sai → tuyệt đối **không bỏ trống** câu nào! |
| **Điểm đạt (Pass)** | **`>= 65%` = Tối thiểu `26 / 40` câu đúng**. | Mục tiêu an toàn khi thi thử: đạt ổn định **`>= 32 / 40` (80%)**. |
| **Thời gian làm bài** | - **60 phút** chuẩn.<br>- **75 phút (+25% Extra Time)** cho thí sinh **không phải người bản ngữ tiếng Anh** (*Non-native English speakers*) khi chọn thi đề tiếng Anh! | **CỰC KỲ QUAN TRỌNG:** Bạn phải **yêu cầu/tick chọn Extra Time ngay lúc đăng ký thi** trên hệ thống Pearson VUE / nhà cung cấp thi địa phương — ngày đi thi không xin bổ sung được! |
| **Hình thức** | Thi đóng sách (*Closed-book*), tại trung tâm Pearson VUE hoặc Online Proctored. | Chuẩn bị căn cước/hộ chiếu hợp lệ và bàn làm việc trống khi thi online. |

### Phân bổ Trọng số 6 Chương (Chìa khóa Ôn tập Thông minh)

| Chương | Tên Chương (Syllabus v4.0.1) | Số câu ước lượng | Tài liệu tương ứng đã có sẵn trong Repo này |
| :---: | :--- | :---: | :--- |
| **1** | Fundamentals of Testing *(Nền tảng)* | **8 câu** | `00-nen-tang/01-seven-testing-principles.md`, `02-istqb-test-process.md`, `03-verification-vs-validation.md` |
| **2** | Testing Throughout the SDLC *(Trong các mô hình phát triển)* | **5 – 6 câu** | `00-nen-tang/03-verification-vs-validation.md`, `04-quy-trinh-thuc-te/01-agile-testing-scrum-three-amigos.md` |
| **3** | Static Testing *(Kiểm thử tĩnh & Review)* | **4 câu** | `00-nen-tang/04-static-vs-dynamic-testing.md`, `05-review-checklist-c1-c6.md` |
| **4** | **Test Analysis and Design** *(Kỹ thuật thiết kế test)* | **11 câu** *(Nhiều nhất!)* | Toàn bộ `01-test-design/01..09` (EP, BVA, Decision Table, State, Coverage, Error Guessing) |
| **5** | **Managing the Test Activities** *(Quản lý kiểm thử)* | **9 câu** *(Nhiều thứ 2!)* | `00-nen-tang/07-viet-bug-report-chuan.md`, `04-quy-trinh-thuc-te/02..04` (Test Plan, Risk, Metrics, Defect) |
| **6** | Test Tools *(Công cụ kiểm thử)* | **2 câu** | `03-api-ui-automation/`, `05-non-functional-cicd/` (Lợi ích & rủi ro tự động hóa) |

> **Chiến lược 50% điểm số:** Chỉ riêng **Chương 4 (11 câu)** + **Chương 5 (9 câu)** đã chiếm **20 / 40 câu (50% toàn đề)**! Nếu nắm chắc bài tập tính **EP, 2-value/3-value BVA, Decision Table, State Transition (0-switch / 1-switch), Statement/Branch Coverage** ở GĐ 1, bạn gần như nắm chắc phần thắng!

---

## 2. 10 Bẫy Ngôn từ Kinh điển trong Đề thi ISTQB CTFL v4.0.1

1. **"100% Condition Coverage suy ra 100% Branch Coverage?"** → **SAI!** (Nhớ phản ví dụ `if (A || B)` với `(T,F)` và `(F,T)` ở `01-test-design/07-white-box-coverage.md`). Ngược lại, **100% Branch Coverage LUÔN suy ra 100% Statement Coverage**.
2. **Error vs. Defect vs. Failure:** Con người làm nhầm (*Error/Mistake*) → tạo ra lỗi trong code/tài liệu (*Defect/Fault/Bug*) → khi chạy gây sự cố (*Failure*). Static Testing tìm trực tiếp **Defect** trước khi nó kịp gây Failure!
3. **Testing vs. Debugging:** Testing là hoạt động **tìm ra Failure / xác minh sửa lỗi** (do Tester/Dev); Debugging là **tìm nguyên nhân gốc trong code và sửa Defect** (do Dev).
4. **7 Testing Principles — bẫy câu "Absence-of-errors fallacy":** Hệ thống không có bug nhưng không đáp ứng nhu cầu người dùng thì vẫn thất bại (Nguyên lý 7).
5. **Shift-Left không có nghĩa là "chỉ test sớm bằng unit test":** Nó bao gồm review yêu cầu sớm, viết test song song với code, dùng TDD/BDD, và tự động hóa sớm trong CI.
6. **Review Types (v4.0.1 gồm 4 mức):** *Informal review*, *Walkthrough* (Author dẫn dắt), *Technical review* (Peers/chuyên gia xét kỹ thuật), *Inspection* (chính thức nhất, có Moderator, Entry/Exit criteria, đo metrics, Author **không** dẫn dắt).
7. **2-Value BVA vs. 3-Value BVA:** Với miền `[1, 10]`: 2-Value lấy `{0, 1, 10, 11}` (4 giá trị); 3-Value lấy `{0, 1, 2, 9, 10, 11}` (6 giá trị — bắt được cả lỗi đổi toán tử `>=` thành `=` hay `>`).
8. **Product Risk vs. Project Risk:**
   - *Product Risk:* Rủi ro gắn trực tiếp với chất lượng sản phẩm (tính sai tiền, app crash, lỗ hổng bảo mật).
   - *Project Risk:* Rủi ro về tiến độ, nhân sự, môi trường test trễ, thiếu kỹ năng, nhà cung cấp chậm giao hàng.
9. **Test Strategy vs. Test Approach:** Trong một tổ chức, *Test Strategy* mang tính tổng quát nhiều dự án; *Test Approach* là cách triển khai chiến lược đó vào 1 dự án cụ thể (Analytical, Model-based, Methodical, Process-compliant, Directed/Consultative, Regression-averse, Reactive).
10. **Regression Testing vs. Confirmation Testing (Retesting):** *Confirmation* chạy lại đúng test đã fail để xác nhận bug đó đã được sửa; *Regression* chạy rộng ra để đảm bảo bản sửa không làm hỏng các tính năng khác.

---

## 3. Bộ 10 Câu Luyện Thi Mẫu Chuẩn Format ISTQB v4.0.1 (Kèm Giải thích)

1. **[Ch.1 — K2] Một ứng dụng tính thuế chạy nhanh, không có bất kỳ crash hay sai công thức nào theo SRS, nhưng người dùng cuối vẫn từ chối sử dụng vì quy trình nhập liệu quá rườm rà không khớp nghiệp vụ kế toán thực tế. Điều này minh họa nguyên lý kiểm thử nào?**
   - A. Exhaustive testing is impossible
   - B. Defect clustering
   - C. Absence-of-defects fallacy
   - D. Tests wear out
   → **Đáp án: C.** Đạt 100% đúng spec nhưng không đáp ứng kỳ vọng/nhu cầu kinh doanh của người dùng là *Ngụy biện không còn lỗi*.

2. **[Ch.2 — K2] Hoạt động nào sau đây thuộc Test-First Approach?**
   - A. Viết code chức năng trước, sau đó viết Unit Test để đạt 80% coverage.
   - B. Viết Acceptance Test hoặc Unit Test dựa trên yêu cầu **trước** khi viết code cài đặt (TDD, ATDD, BDD).
   - C. Chạy Static Analysis bằng SonarQube sau khi code xong.
   - D. Để khách hàng thực hiện UAT trước khi bàn giao.
   → **Đáp án: B.**

3. **[Ch.3 — K2] Vai trò nào chịu trách nhiệm chính trong việc dẫn dắt buổi họp (*lead the review meeting*) đối với hình thức *Formal Inspection*?**
   - A. Author
   - B. Manager
   - C. Facilitator / Moderator
   - D. Scribe / Recorder
   → **Đáp án: C.** Trong Inspection chuẩn, **Moderator (Facilitator)** điều phối cuộc họp (Author không dẫn dắt như ở Walkthrough).

4. **[Ch.4 — K3 EP] Hệ thống tính phí vận chuyển dựa trên trọng lượng gói hàng (số nguyên `kg` hợp lệ từ `1` đến `50`): `1–5kg` phí `20k`; `6–20kg` phí `35k`; `21–50kg` phí `50k`. Nếu áp dụng Equivalence Partitioning cho cả phân vùng hợp lệ và không hợp lệ, bộ giá trị nào đạt 100% EP coverage tối thiểu?**
   - A. `0, 3, 12, 35, 51`
   - B. `1, 5, 6, 20, 21, 50`
   - C. `3, 12, 35`
   - D. `0, 1, 5, 6, 20, 21, 50, 51`
   → **Đáp án: A.** Có 5 phân vùng: `< 1` (`0`), `1–5` (`3`), `6–20` (`12`), `21–50` (`35`), `> 50` (`51`). Đáp án B & D là BVA; C thiếu 2 phân vùng invalid.

5. **[Ch.4 — K3 3-Value BVA] Cho quy tắc: *"Nếu điểm thi `>= 50` và `<= 100` thì Đậu; `0` đến `49` thì Trượt"*. Nếu kiểm thử cho miền **Đậu (`50–100`)** bằng **3-Value BVA**, cần những giá trị nào?**
   - A. `49, 50, 100, 101`
   - B. `49, 50, 51, 99, 100, 101`
   - C. `0, 49, 50, 100`
   - D. `50, 75, 100`
   → **Đáp án: B.** Mỗi biên của phân vùng (`50` và `100`) lấy `{biên - 1, biên, biên + 1}` → `{49, 50, 51}` và `{99, 100, 101}`.

6. **[Ch.4 — K3 State Transition] Một đơn hàng có trạng thái `Created → Paid → Shipped → Delivered`; từ `Created` hoặc `Paid` có thể chuyển sang `Cancelled`. Cần tối thiểu bao nhiêu Test Case để đạt **100% All Valid Transitions (0-switch) coverage**?**
   - *Các chuyển đổi hợp lệ:* `Created→Paid`, `Paid→Shipped`, `Shipped→Delivered`, `Created→Cancelled`, `Paid→Cancelled` (5 chuyển đổi).
   - Các chuỗi tối thiểu phủ hết 5 cạnh:
     1) `Created → Paid → Shipped → Delivered` (phủ 3 cạnh đầu)
     2) `Created → Cancelled` (phủ cạnh 4)
     3) `Created → Paid → Cancelled` (phủ cạnh 5)
   → **Đáp án: 3 Test Cases.**

7. **[Ch.4 — K3 White-box] Xét đoạn code:**
   ```java
   int discount = 0;
   if (isVip) { discount += 10; }
   if (total > 500) { discount += 5; }
   return discount;
   ```
   **Cần tối thiểu bao nhiêu test case để đạt 100% Statement Coverage và 100% Branch Coverage?**
   - A. 1 cho Statement, 2 cho Branch
   - B. 2 cho Statement, 4 cho Branch
   - C. 1 cho Statement, 4 cho Branch
   - D. 2 cho Statement, 2 cho Branch
   → **Đáp án: A!** *(Câu K3 hay đánh lừa nhất!)*:
   - **Statement:** Chỉ cần **1 ca** `(isVip = true, total = 600)` là chạy qua mọi dòng code (`discount=0`, `discount+=10`, `discount+=5`, `return`) → **100% Statement!**
   - **Branch:** Có 2 `if` độc lập nhau (4 nhánh `T/F`), nhưng **2 ca** `(true, 600)` (đi cả 2 nhánh True) và `(false, 100)` (đi cả 2 nhánh False) đã phủ đủ 4/4 nhánh → **100% Branch!** *(Nếu đề hỏi 100% **Path Coverage** thì mới cần **4** ca!).*

8. **[Ch.5 — K2 Risk] Việc máy chủ môi trường Staging thường xuyên bị sập vào buổi chiều khiến đội QA chậm tiến độ test 3 ngày thuộc loại rủi ro nào?**
   - A. Product Risk
   - B. Project Risk
   - C. Security Risk
   - D. Quality Debt
   → **Đáp án: B (Project Risk)** — ảnh hưởng tới lịch trình/nguồn lực thực hiện dự án chứ không phải lỗi của phần mềm đang bán.

9. **[Ch.5 — K2 Test Estimation] Kỹ thuật ước lượng kiểm thử nào sử dụng dữ liệu trung bình của ba giá trị *(Optimistic, Most Likely, Pessimistic)*?**
   - A. Planning Poker
   - B. Three-Point Estimation ($E = \frac{a + 4m + b}{6}$)
   - C. Extrapolation
   - D. Wideband Delphi
   → **Đáp án: B.**

10. **[Ch.6 — K1] Đâu là một RỦI RO (Risk) phổ biến khi triển khai Test Automation?**
    - A. Giảm thời gian chạy Regression.
    - B. Kỳ vọng phi thực tế rằng công cụ sẽ tự động tìm thấy mọi bug và đánh giá thấp chi phí bảo trì test script.
    - C. Tăng độ lặp lại nhất quán của test execution.
    - D. Giải phóng tester để tập trung vào Exploratory Testing.
    → **Đáp án: B.**
