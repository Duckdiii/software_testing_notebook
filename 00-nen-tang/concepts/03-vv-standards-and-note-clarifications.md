# Chuẩn Quốc tế về V&V & Giải thích Các điểm Mơ hồ trong Tài liệu Trường

> **Mục đích file này:**
> 1. Bổ sung **các phần kiến thức mới chưa có** trong repo từ các file tổng hợp (`Verification và Validation.pdf`, `Dynamic testing.pdf`).
> 2. **Giải thích rõ ràng mọi điểm mâu thuẫn, mơ hồ, viết tắt hoặc lỗi định dạng** xuất hiện trong bộ tài liệu note tại trường để bạn không bị rối khi ôn tập hay làm bài kiểm tra.
> 3. **Không lặp lại** các phần đã trình bày đầy đủ ở [01-seven-testing-principles.md](../01-seven-testing-principles.md) .. [07-viet-bug-report-chuan.md](../07-viet-bug-report-chuan.md).

---

## Phần A: Kiến thức bổ sung chưa có trong tài liệu cũ

### 1. Verification và Validation theo 3 chuẩn quốc tế (`Verification và Validation.pdf`)

Ngoài định nghĩa của Barry Boehm, IEEE 29119 và Ian Sommerville (đã có ở [03-verification-vs-validation.md](../03-verification-vs-validation.md)), tài liệu môn học còn đối chiếu V&V theo 3 bộ chuẩn quản trị quy trình & đảm bảo chất lượng sau:

| Chuẩn quốc tế | Hoạt động Verification (Xác minh) | Hoạt động Validation (Xác thực) / Mục tiêu V&V |
| :--- | :--- | :--- |
| **ISO/IEC 12207:2008** *(Vòng đời phần mềm)* | - **Xác minh yêu cầu:** Review tài liệu yêu cầu *(sửa typo trong PDF `"eview"` → review)*.<br>- **Xác minh thiết kế:** Đánh giá toàn bộ tài liệu thiết kế HLD & LLD.<br>- **Kiểm tra code:** Thực hiện Review code.<br>- **Xác minh tài liệu:** Kiểm tra hướng dẫn sử dụng và tài liệu kèm theo. | - Chuẩn bị test requirements, test cases và thông số để phân tích kết quả test.<br>- Đánh giá các test case/test spec phản ánh đúng yêu cầu và phù hợp sử dụng.<br>- Test biên, stress test, chức năng và xử lý thông báo lỗi/thoát ứng dụng an toàn.<br>- Xác nhận phần mềm đáp ứng nhu cầu nghiệp vụ và *fit for use*. |
| **CMMI** *(Capability Maturity Model Integration)* | - Thực hiện **Peer Reviews**.<br>- Xác minh các sản phẩm công việc (*work products*) được chọn.<br>- Chuẩn hóa quy trình bằng quy tắc lập kế hoạch và thực hiện đánh giá. | - Xác nhận sản phẩm và thành phần sản phẩm **phù hợp với môi trường vận hành thực tế**.<br>- Theo dõi và kiểm soát khi thực hiện validation.<br>- Rút kinh nghiệm, thu thập thông tin cải tiến và thiết lập quy trình chuẩn. |
| **IEEE 1012** *(Standard for System, Software, and Hardware V&V)* | Tập trung bộ mục tiêu chung cho cả V&V:<br>1. Phát hiện và sửa lỗi từ sớm.<br>2. Tăng cường sự tham gia của cấp quản lý vào kiểm soát quy trình và rủi ro sản phẩm.<br>3. Hỗ trợ vòng đời phần mềm giúp tuân thủ tiến độ (*schedule*) và ngân sách (*budget*). | *(Áp dụng xuyên suốt cho cả tiến độ, chi phí và kiểm soát rủi ro kỹ thuật lẫn nghiệp vụ).* |

### 2. Các sản phẩm của Kiểm thử động (`Dynamic Test Work Products`) & Retesting vs Regression (`Dynamic testing.pdf`)

Để đối xứng với sản phẩm của Kiểm thử tĩnh, khi thực thi **Dynamic Testing** đội dự án tạo ra và sử dụng 5 nhóm tạo tác (*work products*) chính:
1. **Executable Builds:** Bản build thực thi được (`.apk`, `.ipa`, ứng dụng web triển khai trên Staging/Test).
2. **Test Cases:** Tập kịch bản kiểm thử chi tiết (tiền điều kiện, bước chạy, kỳ vọng).
3. **Test Data:** Bộ dữ liệu mẫu phục vụ chạy test (tài khoản chuẩn, số âm, biên, thẻ test...).
4. **Test Log / Execution Report:** Nhật ký các lần chạy (Pass/Fail, thời gian thực thi, log build).
5. **Defect Report (Bug Report):** Báo cáo sai lệch kèm bước tái hiện và bằng chứng.

> **Nhắc lại phân biệt `Confirmation Testing (Retesting)` vs `Regression Testing`:**
> - **Confirmation Testing / Retesting:** Chạy lại **đúng test case từng Fail** sau khi Dev báo đã sửa bug → chứng minh *bug cũ đã hết*.
> - **Regression Testing:** Chạy lại **các tính năng cũ liên quan/xung quanh** → chứng minh *bản sửa lỗi (hoặc tính năng mới) không làm gãy thứ đang chạy tốt*.

---

## Phần B: Giải thích rành mạch các điểm Mâu thuẫn & Mơ hồ trong Note tại trường

Khi gộp các file note từ nhiều buổi học, có **7 điểm mâu thuẫn hoặc viết tắt dễ gây hiểu sai** cần được đính chính chuẩn theo ISTQB CTFL v4.0.1 và ISO 9000:

### 1. Mâu thuẫn lớn nhất: `Verification → QA` và `Validation → QC` (`Week 01.pdf` p.3 & `Ôn tập.pdf` p.3)
- **Hiện tượng trong file trường:** `Week 01.pdf` (trang 3) và `Ôn tập.pdf` (trang 3) ghi chú ngắn gọn *"Verification (Xác minh) → QA"* và *"Validation (Xác nhận) → QC"*.
- **Vì sao gây hiểu sai?** Ngay ở mục kế tiếp (mục 5 của `Ôn tập.pdf` và tài liệu Tuần 2), trường lại định nghĩa đúng:
  - **QA (Quality Assurance)** hướng **Quy trình** (*Process-oriented*, phòng ngừa lỗi).
  - **QC (Quality Control)** hướng **Sản phẩm** (*Product-oriented*, kiểm tra/test trực tiếp sản phẩm).
- **Giải thích chuẩn xác:** **Cả Verification và Validation khi bạn trực tiếp kiểm tra sản phẩm cụ thể** (review SRS, inspect code, chạy Unit Test, chạy System Test, chạy UAT) **đều là hoạt động thuộc QC**! Còn **QA** là việc *xây dựng quy chuẩn, quy trình, checklist, CI/CD, đào tạo* để đội ngũ thực hiện tốt Verification & Validation từ đầu. Đừng bao giờ học vẹt *"Verification = QA, Validation = QC"*.

### 2. Lỗi copy-paste lẫn lộn dưới `Unit Testing & Integration Testing` và `Static Validation` (`Verification và Validation.pdf` p.3–4)
- **Hiện tượng trong file trường:**
  - Ở trang 3, ngay dưới dòng *"Unit Testing & Integration Testing: Chạy mã để verify các hàm, module..."*, danh sách đánh số `1.–5.` lại ghi *"4. Điều này được thực hiện mà không cần chạy phần mềm"* và *"5. Bao gồm tất cả các kỹ thuật test tĩnh..."*.
  - Ở trang 4, ngay dưới dòng *"Static Validation (ở giai đoạn đầu): Đưa bản thiết kế mẫu..."*, danh sách `1.–5.` lại ghi *"4. Được thực hiện cùng với việc chạy phần mềm"* và *"5. Bao gồm tất cả các kỹ thuật test động..."*.
  - Ở Bảng 4 trang 4, cột kỹ thuật chủ đạo ghép `Verification` chủ yếu với *Static* và `Validation` chủ yếu với *Dynamic*.
- **Giải thích chuẩn xác:** Hai khối `1.–5.` bị paste nhầm vị trí (chúng thực chất mô tả khái niệm **Verification** nói chung và **Validation** nói chung ở định nghĩa truyền thống, không phải mô tả cho Unit Test hay Static Validation). Thực tế V&V và Static/Dynamic tạo thành **4 góc phần tư độc lập**:

| | **Verification (Đúng đặc tả / Thiết kế)** | **Validation (Đúng nhu cầu người dùng / Thực tế)** |
| :--- | :--- | :--- |
| **Static Testing (Không chạy code)** | Reviews SRS/Design, Linters, Static Analysis | **Static Validation:** Khách bấm thử Wireframe, Figma Prototype, Storyboard |
| **Dynamic Testing (Có chạy code)** | **Unit Testing, Integration Testing** *(verify theo LLD/Interface)* | **UAT, Alpha/Beta, OAT, System Testing** *(E2E người dùng thật)* |

### 3. Mâu thuẫn nhãn V&V của `System Testing` giữa 4 file PDF
- `System Testing.pdf` (p.3) và `UAT.pdf` (p.3) ghi góc nhìn của System Testing là *Verification*.
- `Verification và Validation.pdf` (p.3) đặt System Testing dưới mục *Validation*.
- `Vị trí của V&V trong Mô hình chữ V.pdf` (p.3–4) gọi System Testing là **Giao thoa V&V**.
- **Kết luận chuẩn:** Hãy dùng cách gọi **Giao thoa V&V** — System Testing kiểm chứng cả hai: đối chiếu thiết kế kiến trúc/SLA kỹ thuật (Verification) và chạy hành trình E2E thực tế của người dùng cuối trước bàn giao (Validation).

### 4. Mơ hồ cụm viết tắt `"30% dev + 70% test"` và `"Discover defects + evaluate quality + verify requirement"` (`Week 01.pdf` p.1)
- **Giải thích `"Discover defects + evaluate quality + verify requirement"`:** Đây là tóm tắt 3 mục tiêu đầu của Kiểm thử theo ISTQB CTFL v4.0.1 Section 1.1.1 (còn đầy đủ còn có: ngăn ngừa lỗi, cung cấp thông tin cho stakeholder ra quyết định release, kiểm tra tuân thủ hợp đồng/pháp lý và validate fit-for-use).
- **Giải thích `"30% dev + 70% test"`:** Đây chỉ là một câu nhấn mạnh mang tính hình tượng/thực tế chi phí trên slide bài giảng (rằng trong một hệ thống thương mại nghiêm túc, thời gian/chi phí cho đảm bảo chất lượng, viết test, chạy hồi quy, debug và bảo trì thường áp đảo thời gian viết code tính năng thô ban đầu) — **không phải quy định cứng** rằng mọi lập trình viên chỉ ngồi code 30% giờ làm việc!

### 5. Phát biểu `"Failure chỉ xảy ra trong quá trình kiểm thử động"` (`Ôn tập.pdf` p.3)
- **Vì sao chưa trọn vẹn?** Nếu chỉ nói *"Failure chỉ xảy ra trong quá trình Dynamic Testing"*, câu này mâu thuẫn với chính 3 sự cố lịch sử ở trang 1–2 (`Ariane 5`, `Therac-25`, `Knight Capital`) — vốn xảy ra khi phần mềm đã vận hành ngoài đời thật!
- **Diễn đạt chính xác:** **Failure chỉ xảy ra khi mã nguồn chứa Defect thực sự được thực thi (*at runtime*)** — điều đó xảy ra khi ta chủ động chạy **Dynamic Testing**, HOẶC khi **người dùng thật vận hành hệ thống trên Production** nếu bug bị lọt qua khâu test!

### 6. `"Technical Review có chiều sâu cao nhất"` vs `"Formal Inspection có mức độ hình thức cao nhất"` (`Technical Review.pdf` p.1 vs `Walkthrough.pdf` p.5)
- Không mâu thuẫn nếu phân biệt đúng trục so sánh:
  - **Technical Depth (Chiều sâu kiến trúc/công nghệ):** **Technical Review** cao nhất (hội đồng ARB mổ xẻ DB, concurrency, security, chi phí cloud, chọn công nghệ).
  - **Process Formality & Statistical Rigor (Mức độ hình thức quy trình):** **Formal Inspection (Fagan)** cao nhất (5 vai trò độc lập Moderator/Author/Reader/Recorder/Inspector, tốc độ đọc chuẩn dòng/giờ, bắt buộc fix 100% defect mới đạt Exit Criteria).

### 7. Khác biệt đánh số Checklist Review & Lỗi biên soạn/OCR nhỏ trong PDF trường
- **Checklist `C01–C10` (`Requirements quality.pdf`) vs `C1–C6 / C7–C9` ([05-review-checklist-c1-c6.md](../05-review-checklist-c1-c6.md)):** Hai tài liệu gốc của trường (`Lab02a` và `Week02_Further_Reading`) đánh số thứ tự tiêu chí hơi khác nhau (VD: `Lab02a` xếp `C02 = Consistency`, `C03 = Clarity`; còn `05-review-checklist` giữ `C2 = Clarity`, `C4 = Consistency` theo tài liệu đọc thêm). Khi làm Lab 02 trên lớp nhớ bám đúng bảng C01–C10 của đề Lab!
- **Câu ví dụ `C05` trong `Requirements quality.pdf`:** Viết *"Sinh viên đăng ký tối đa đến 24 tín chỉ"* để minh họa câu mơ hồ (chưa rõ có nhận số `24` và chặn số `0` hay không), câu sửa chuẩn phải ghi rõ cận dưới và cận trên: *"Từ 1 đến 24 tín chỉ (bao gồm cả 1 và 24)"*.
- **Các lỗi mất chữ / xuất PDF trong `Week 01.pdf` và `Ôn tập.pdf`:**
  - `Week 01.pdf` trang 5 (R2): viết thiếu số `0` (`"từ đến 39"` → đúng là **`0 đến 39`**).
  - `Week 01.pdf` trang 11 (Thang điểm): bị mất chữ số hàng đơn vị khi export PDF (`".0 điểm"`, `".5 điểm"`). Trên trang 10 có ghi rõ Task 6 (`Q1–Q4`) chiếm **`2.0 điểm`** trong tổng `10.0 điểm`, và đầy đủ ảnh minh chứng là `1.0 điểm` (tổng 3 mục còn lại = `7.0 điểm` theo format `.0`, `.5`, `.5` — bạn đối chiếu lại file đề `Lab01_Assignment.pdf` trước khi tính trọng số điểm chính thức).
  - Các lỗi hiển thị LaTeX/HTML thừa trong PDF (`→\rightarrow→`, `1≤credits≤241 \le \text{credits} \le 241≤credits≤24`, `như nhau.=`, `<br>`, `<mark>Fail</mark>`, `"nderlying defect"`) đã được chuẩn hóa sạch hoàn toàn khi chuyển sang các file Markdown trong repo.
