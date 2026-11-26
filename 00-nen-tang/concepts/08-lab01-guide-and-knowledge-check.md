# Ghi chú Thực hành Lab 01 & Bộ Câu hỏi Ôn tập Tuần 1

> **Nguồn bài học:** Phần thực hành & tự kiểm tra chưa trùng lặp từ `Week 01.pdf` (Task 1–6) và `Ôn tập.pdf` (Phần 1 & Phần 9).  
> *Lưu ý:* Phần lý thuyết nền tảng, bảng `TC01..TC08` và `BUG-01..BUG-02` chi tiết đã có đầy đủ tại [01-seven-testing-principles.md](../01-seven-testing-principles.md), [06-viet-test-case-chuan.md](../06-viet-test-case-chuan.md) và [07-viet-bug-report-chuan.md](../07-viet-bug-report-chuan.md) nên **không lặp lại tại đây**.

---

## 1. Mục tiêu học tập & Lộ trình môn học (Week 1 – Week 15)

- **Learning Objectives (LO1–LO5):**
  - **LO1:** Định nghĩa kiểm thử phần mềm và giải thích các mục tiêu cốt lõi.
  - **LO2:** Phân biệt chính xác chuỗi `Error → Defect/Bug → Failure` (+ `Root Cause`).
  - **LO3:** Vận dụng **7 nguyên lý kiểm thử cơ bản** của ISTQB.
  - **LO4:** Nhận diện 4 Test Levels, các Test Types và 7 bước Test Process.
  - **LO5:** Thiết kế Test Cases bằng **EP** & **BVA**, thực thi và viết Bug Report chuẩn.
- **Lộ trình tổng quan:** `Tuần 1–4` (Nền tảng, Test Design, Coverage) → `Tuần 5–8` (Unit & Integration: JUnit, pytest, API, Midterm) → `Tuần 9–13` (Agile, Exploratory, DB, Performance, Security) → `Tuần 14–15` (CI/CD, UI/Continuous Automation, Final Project).

---

## 2. Quy trình thực hành Lab 01 (`Week 01.pdf` — Task 1 → Task 6)

**Quy tắc nghiệp vụ SUT (`CourseRegistration_Buggy.html`):**
- `R1`: `1 <= Credits <= 24` *(gồm cả 1 và 24)*.
- `R2`: `0 <= Enrollment <= 39` *(gồm cả 0 và 39 — sửa lỗi thiếu số `0` trong `Week 01.pdf` trang 5 `"từ đến 39"`)*.
- `R3`: `Prerequisite = Yes` *(đã tick)*.
- `R4`: Chỉ `ACCEPTED` khi thỏa đồng thời cả `R1`, `R2`, `R3`.

**Các bước thực thi & Đóng gói nộp bài (90 phút):**
1. **Task 1 — Chuẩn bị workspace (5'):** Tạo thư mục `Lab01_<StudentID>_<FullName>` + thư mục con `Evidence/`, copy `CourseRegistration_Buggy.html`, `Test_Case_Template.csv`, `Bug_Report_Template.csv`. **Tuyệt đối không chỉnh sửa code HTML gốc.**
2. **Task 2 — Khám phá SUT (10'):** Chạy ca hợp lệ (`Credits=12, Enrollment=25` hoặc `5, Prerequisite=Yes`), chụp toàn bộ màn hình gồm input + kết quả `ACCEPTED` lưu thành `Evidence/E1.png`.
3. **Task 3 — Xác định Expected Result TRƯỚC KHI chạy (10'):** Suy luận từ Test Oracle `R1–R4` để điền `Expected Result` cho `TC01..TC08`, tránh *"chép ngược"* kết quả web làm sai tư duy kiểm thử.
4. **Task 4 — Thực thi & Chụp bằng chứng (25'):** Chạy 8 ca → lưu CSV chuẩn **UTF-8 (`Test_Cases_<StudentID>.csv`)**. Riêng 2 ca Fail chụp ảnh lưu `Evidence/E_TC02.png` (`Credits=1`) và `Evidence/E_TC06.png` (`Enrollment=40`).
5. **Task 5 — Báo cáo lỗi (20'):** Ghi `BUG-01` và `BUG-02` vào `Bug_Reports_<StudentID>.csv` (UTF-8). *(Nhớ nguyên tắc Black-box: ghi nhận triệu chứng khách quan & bằng chứng tái hiện, tránh đoán bừa dòng code khi chưa inspect source).*
6. **Task 6 — Trả lời 4 câu lý thuyết & Nén ZIP (10'):**

```text
Lab01_23520001_NguyenVanA/
├── Evidence/
│   ├── E1.png
│   ├── E_TC02.png
│   └── E_TC06.png
├── Test_Cases_23520001.csv
├── Bug_Reports_23520001.csv
└── Lab01_Report_23520001.pdf
```

> **Lưu ý thang điểm (Rubric PDF trang 10–11):** Bản PDF `Week 01.pdf` trang 11 bị mất chữ số hàng đơn vị ở 4 dòng (`".0 điểm"`, `".5 điểm"`). Trên trang 10 ghi rõ câu hỏi `Q1–Q4` chiếm **`2.0 điểm`** trong tổng `10.0 điểm`, và ảnh chụp màn hình chiếm **`1.0 điểm`**. Hãy kiểm tra lại file `Lab01_Assignment.pdf` gốc để xác nhận phân bổ chính xác `7.0 điểm` cho 3 mục CSV còn lại.

---

## 3. Lời giải 4 câu hỏi Lab 01 (`Q1–Q4`)

1. **Q1 — Phân biệt `Expected Result` và `Actual Result`:**
   - `Expected Result` được suy ra **trước khi chạy** từ Test Oracle (`R1–R4`).
   - `Actual Result` là kết quả quan sát thực tế khi chạy SUT. So sánh hai giá trị này là căn cứ kết luận `Pass` / `Fail`.
2. **Q2 — Giải thích 1 `Failure` quan sát được và `Underlying Defect` gây ra nó:**
   - **Failure:** Nhập `Credits = 1, Enrollment = 25, Prerequisite = Yes` → app hiển thị `REJECTED: Registration not allowed` (`TC02` Fail).
   - **Underlying Defect:** Khi kiểm tra script của ứng dụng, dev viết `c > 1` thay vì `c >= 1`, bỏ sót giá trị biên `1`.
3. **Q3 — Tại sao các giá trị biên `{0, 1, 24, 25, 39, 40}` đặc biệt hữu ích?**
   - Lập trình viên hay nhầm `<` vs `<=` hoặc `>` vs `>=` (Off-by-one). Test tại ranh giới vùng hợp lệ / không hợp lệ mang lại xác suất bắt lỗi cao nhất với ít test case nhất.
4. **Q4 — Tại sao 100% Test Case `Pass` vẫn không chứng minh phần mềm sạch 100% lỗi?**
   - Theo **Nguyên lý 1 ISTQB** (*Testing shows the presence, not the absence, of defects*): 8 ca Pass chỉ chứng minh đúng trong đúng 8 tổ hợp đầu vào đó; hệ thống vẫn có thể lỗi với số âm, chuỗi chữ, số thập phân hoặc lỗi đồng thời (*concurrency*).

---

## 4. Bộ 5 câu Knowledge Check Ôn tập Tuần 1 (`Ôn tập.pdf`)

1. **Tại sao kiểm thử chỉ chứng minh sự hiện diện của lỗi mà không thể chứng minh không còn lỗi?**
   - *Đáp án:* Không gian trạng thái và tổ hợp dữ liệu đầu vào là vô hạn; tập test hữu hạn chỉ kiểm chứng được các điều kiện đã chạy.
2. **Ví dụ phân biệt `Error → Defect → Failure`?**
   - *Đáp án:* Dev mệt mỏi hiểu sai biên dưới không lấy số 1 (`Error`) → viết lệnh `credits > 1` trong code (`Defect`) → sinh viên nhập `1` tín chỉ bị từ chối khi chạy app (`Failure`).
3. **Tại sao test case nhập dữ liệu sai bị `REJECTED` vẫn kết luận là `PASS`?**
   - *Đáp án:* Test Oracle quy định dữ liệu sai phải bị từ chối (`Expected = REJECTED`). Hệ thống chặn đúng (`Actual = REJECTED`). Vì `Actual == Expected` nên verdict là `PASS`.
4. **Tại sao `Severity` và `Priority` khác nhau?**
   - *Đáp án:* **Severity** đo tác động kỹ thuật/chức năng lên hệ thống (Tester đề xuất); **Priority** đo mức độ khẩn cấp về kinh doanh/lịch phát hành (PO/PM quyết định). VD: sai chính tả logo trang chủ trước ngày ra mắt có *Low Severity* nhưng *High Priority*.
5. **Thế nào là `"Absence-of-defects fallacy"`?**
   - *Đáp án:* Phần mềm chạy trơn tru, không lỗi kỹ thuật theo đặc tả nhưng đặc tả/UI không giải quyết đúng nhu cầu thực tế của người dùng → sản phẩm vẫn thất bại.
