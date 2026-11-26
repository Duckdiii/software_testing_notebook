# Ghi chú Tuần 2 & Hướng dẫn Lab 02a, 02b, Lab 03, Lab 04 (`Week 02.pdf`)

> **Nguồn bài học:** `Week 02.pdf` *(Tổng hợp lý thuyết Further Reading + quy trình Lab 02a, Lab 02b, Lab 03 Ruff, Lab 04 Report Submission & các lưu ý đính chính ghi chú trên lớp)*.

---

## Phần A: Giải thích & Làm rõ Ghi chú Tốc ký trên lớp (`Week 02.pdf` tr. 1–2)

Trong `Week 02.pdf` có một số dòng ghi chép nhanh ngay trên lớp khá vắn tắt hoặc dễ hiểu nhầm — bảng dưới đây giải thích rành mạch từng dòng:

| Ghi chú gốc trong `Week 02.pdf` | Ý nghĩa đầy đủ & Đính chính chuẩn |
| :--- | :--- |
| `"Hack -> ko có phần mềm nào ko có lỗi"` | Xuất phát từ thực tế bảo mật & Nguyên lý 1 ISTQB: mọi hệ thống phần mềm phức tạp đều tiềm ẩn khuyết tật; việc review/test/hardening nhằm **phát hiện và giảm rủi ro** chứ không chứng minh *"0 lỗi"*. |
| `"verification -> spec -> code -> check"`<br>`"validation -> user -> app -> FIT"` | Mẹo nhớ cực hay: **Verification** đối chiếu *Code/Tài liệu* với *Spec*; **Validation** đặt *App đang chạy (hoặc Prototype)* trước mặt *User thật* để chứng minh **Fit for purpose**. |
| `"Quality management -> QA (số lượng tài liệu đã đủ chưa, từng bc thực hiện đúng chưa...) -> QC (chất lượng document, content bên trong) -> (verification + validation) thuộc QC"` | **Đoạn này đính chính dứt điểm cho sai sót ở `Week 01.pdf` tr.3:** **Cả Verification và Validation khi trực tiếp đánh giá chất lượng nội dung sản phẩm đều thuộc QC!** *(Lưu ý thêm: QA không chỉ máy móc kiểm đếm "đủ giấy tờ chưa" mà trọng tâm là thiết lập và cải tiến quy trình, chuẩn coding, CI/CD, đào tạo để phòng ngừa lỗi từ gốc).* |
| `"formal review: cuộc họp chính thức, họp lại để thống nhất tất cả, kich off, có khách hàng, team dev, test, skateholders"` | *(Sửa typo `"kich off"` → kick-off, `"skateholders"` → stakeholders).* **Lưu ý phân biệt:** Họp *kick-off* dự án có khách hàng/stakeholders là buổi thống nhất phạm vi chung, **KHÔNG phải Formal Inspection (Fagan)**. Trong kiểm thử tĩnh chuẩn IEEE 1028, *Formal Inspection* do Moderator độc lập + các Inspectors kỹ thuật soi từng dòng theo checklist & metrics *(xem [09-formal-inspection.md](./09-formal-inspection.md))*. |
| `"walkthrough: đi nhanh qua vấn đề"` | Dễ gây hiểu lầm thành *"đọc lướt cho nhanh"*. Thực chất Walkthrough là buổi **Tác giả dẫn dắt cả nhóm đi tuần tự từng bước qua tài liệu/luồng bằng kịch bản (*Mental Dry-Run*)** để thống nhất hiểu biết và bắt lỗ hổng logic. |
| `"technical review: ... *technical manager, ERD manager (quản lí khách hàng)"` | Có khả năng ghi nhầm từ nghe giảng: **`ERD`** trong phần mềm là *Entity Relationship Diagram*; còn chức danh quản lý khách hàng/dịch vụ thường là **Engagement Manager / Delivery Manager / Account Manager** (hoặc **ERB / ARB** — *Engineering/Architecture Review Board*). Trọng tâm Technical Review là **nhóm kỹ sư/kiến trúc sư ngang cấp trở lên thẩm định kiến trúc, thuật toán và tính khả thi công nghệ**. |
| `SpotBugs`, `Checkstyle`, `PMD`, `SonarQube` | Với **Java** (Giai đoạn 2–5), bộ công cụ phân tích tĩnh tương đương **Ruff** (Python) gồm **Checkstyle** (chuẩn code style), **PMD** (code smells, dead code), **SpotBugs** (quét bytecode `.class` tìm NPE/resource leak) và **SonarQube** (SAST + metrics toàn diện). |

---

## Phần B: Quy trình Thực hành Các bài Lab Tuần 2

### 1. Lab 02a: Checklist-Based Requirements Review *(Cá nhân, 90', 10 điểm)*
Đóng vai Junior Tester rà soát phát biểu từ **5 bên**: *Academic Affairs, Student Rep, IT Department, Faculty Advisor, Project Coordinator* **(chỉ review tĩnh — không viết code, không chạy tool)**.

| Task | Thời lượng | Nội dung chính | Yêu cầu tối thiểu |
| :---: | :---: | :--- | :--- |
| **1** | `15'` | **Stakeholder analysis** | Lập bảng vai trò và kỳ vọng của 5 bên |
| **2** | `20'` | **Tách requirement atomic** | Tách `>= 12` yêu cầu gắn mã `REQ-xx`, phân loại **FR / NFR**, ghi rõ nguồn |
| **3** | `35'` | **Áp checklist `C01–C10`** | Tìm `>= 6 findings` trải trên `>= 4 category` (`Clarity, Completeness, Consistency, Testability, Boundary, Exception Handling, Security & Access, Feasibility, Traceability, Prioritization`) |
| **4** | `15'` | **Findings log & Cải tiến** | Phân loại *Potential Defect* vs *Clarification Question*; viết lại `3` requirement cho *testable* kèm nhãn `assumption` nếu có giả định |
| **5** | `5'` | **Reflection** | Trả lời `4` câu hỏi phản tư cuối bài |

- **Nộp bài:** `Lab02_StudentID_FullName.zip` chứa `Lab02_Review_Report_StudentID.pdf`.
- **Thang điểm (`10.0`):** Stakeholder `1.0` · FR/NFR `2.0` · Checklist `3.0` · Findings `2.0` · 3 cải tiến `1.5` · Reflection `0.5`.

---

### 2. Lab 02b: Author-Led Walkthrough *(Nhóm 3–4 người, 90', 10 điểm)*
- **Artifact:** `Draft Design v0.1` gồm luồng `W1–W8` *(đăng nhập → xem danh sách → chọn môn → tính tín chỉ từ chối nếu `> 24` → check trùng lịch → lưu và giảm seat → báo thành công → sửa lại sau)*.
- **Vai trò:** `Author`, `Facilitator`, `Reviewer`, `Recorder` *(nhóm 3 người thì Facilitator kiêm Recorder)*.
- **Trace 6 kịch bản cốt lõi (`S1–S6`):**
  1. `S1`: Standard (ca chuẩn hợp lệ)
  2. `S2`: Credit boundary (đã có đúng `24` tín chỉ rồi thêm `1` môn)
  3. `S3`: Prerequisite (chưa học môn tiên quyết)
  4. `S4`: Capacity (`2` người tranh `1` seat cuối đồng thời)
  5. `S5`: Exception (cố vấn học tập duyệt ngoại lệ)
  6. `S6`: Sửa đăng ký sau deadline
- **Yêu cầu mỗi scenario:** Đường đi qua `W1–W8`, kết quả kỳ vọng (nếu đã định nghĩa), quyết định còn thiếu, câu hỏi gửi stakeholder. Tìm **`>= 5 findings`** trên **`>= 3 scenarios`** (phân loại *Potential Defect / Ambiguity / Missing Case / Stakeholder Clarification*, ưu tiên `2` finding quan trọng nhất) và đề xuất **`>= 3` cải tiến workflow** (chỗ chưa chốt ghi `"Pending stakeholder confirmation"`).
- **Nộp bài:** `Lab02_Walkthrough_StudentID_Group.zip` chứa PDF `3–5` trang (*minutes, 6 scenario trace, findings, cải tiến, 2 reflection, đóng góp cá nhân*).
- **Thang điểm (`10.0`):** Chuẩn bị `1.0` · Scenario `3.0` · Findings `3.0` · Cải tiến `2.0` · Reflection `1.0`.

---

### 3. Lab 03 & Lab 04: Ruff Static Analysis (`35–40'`) & Report Submission (`15–20'`, `10 điểm`)

**Các bước thực hành PowerShell trên Windows:**
```powershell
py --version
mkdir Lab03_Ruff; cd Lab03_Ruff
py -m pip install ruff
py -m ruff --version                                      # Chụp L3-E1
# Copy registration_starter.py và giữ nguyên bản gốc
py -m ruff check --select F401,F841 registration_starter.py  # Chụp L3-E2
Copy-Item registration_starter.py registration_fixed.py
# Chỉ sửa đúng 2 lỗi lint (F401 import os ở dòng 1, F841 debug_message ở dòng 5) -> Chụp L3-E3
py -m ruff check --select F401,F841 registration_fixed.py    # Chụp L3-E4
```

**Bước 8 (Review business logic tĩnh — KHÔNG chạy file & KHÔNG sửa logic):**  
Ruff báo sạch không đồng nghĩa code đúng nghiệp vụ! Quan sát `registration_starter.py` phát hiện **2 lỗi logic** mà Ruff hoàn toàn bỏ qua:

| Scenario | Input mẫu | Kỳ vọng theo Policy | Hành vi thực tế đọc từ code |
| :--- | :--- | :--- | :--- |
| **`S1` Credit boundary** | `[12, 12]`, `seats=1` | **Chấp nhận** (`24 <= max`) | Code viết `total >= 24` → từ chối sai *(đúng phải là `total > 24`)* |
| **`S2` Full course** | `[3]`, `seats=0` | **Từ chối** | Code viết `seats >= 0` → vẫn confirm sai *(đúng phải là `seats > 0`)* |
| **`S3` Ordinary** | `[3, 4, 3]`, `seats=2` | **Chấp nhận** | Đúng |

> **Ranh giới Static vs. Dynamic trong Lab 04:** Khi bạn *đọc mã nguồn* để dự đoán `S1`/`S2` sai → đó vẫn là **Static Review**. Nếu bạn bấm `py registration_starter.py` cho chương trình *chạy thật* để kiểm chứng output → phải ghi rõ bước đó là **Dynamic Testing**, không phải Ruff!

**Cấu trúc đóng gói `Lab04_StudentID_FullName.zip` & Rubric (`10.0` điểm):**
```text
Lab04_StudentID_FullName.zip
├── source/
│   ├── registration_starter.py
│   └── registration_fixed.py
├── evidence/
│   ├── E1_original.png
│   ├── E2_ruff_before.png
│   ├── E3_corrected.png
│   └── E4_ruff_after.png
└── report/
    └── Lab04_Report_StudentID.pdf
```
- **Mục `E5` bắt buộc trong PDF:** Giải thích bằng chữ ít nhất `1` business rule Ruff không kiểm được và vì sao *"clean"* ≠ *"đúng nghiệp vụ"*.
- **Thang điểm (`10.0`):** Evidence `2.5` · Findings `2.0` · Correction & Recheck `2.0` · Reflection `1.5` · Tổ chức PDF & ZIP `2.0` *(chụp ảnh màn hình thật, không chỉnh sửa; chấm tài liệu và bằng chứng chứ không chấm số warning)*.

---

## Phần C: Ba Lưu ý Đính chính Đề Lab Tuần 2 (`Week 02.pdf` tr. 6)

1. **Tên thư mục lab bị lệch số:** Thư mục `Lab01` chứa đề *"Lab 02 Checklist"*, thư mục `Lab02` chứa đề *"Lab 02 Walkthrough"* (cả hai đều ghi `Lab 02`). Khi nén file nộp, luôn kiểm đúng tên `.zip` ghi trong từng file đề (`Lab02_...` vs `Lab02_Walkthrough_...`).
2. **Số lượng tiêu chí checklist khác nhau:** `Week2_Further_Reading.pdf` liệt kê **`9` mục** (`C1–C9`), trong khi `Lab 02a` yêu cầu **`10` mục (`C01–C10`**, bổ sung `Prioritization`). Khi làm `Lab 02a`, tuân theo bộ **`C01–C10`**.
3. **Mâu thuẫn trong đề Walkthrough (`Lab 02b`):** Đề bài có câu *"Each group adds more than 8 scenarios"* nhưng ở yêu cầu nộp bài lại ghi *"six scenario traces"* (`S1–S6`). Khi làm nộp chính thức nên xác nhận lại giảng viên giữ `6` kịch bản chuẩn hay viết `> 8` kịch bản.
