# Concepts Bổ sung — Giai đoạn 0: Nền tảng Tư duy Kiểm thử

Thư mục này chuyển thể sang Markdown, phân loại, lọc trùng lặp và giải thích rõ các điểm mơ hồ từ bộ ghi chú bài học trên lớp thuộc Tuần 1–2.

## Danh sách các bài học chuyên sâu (`concepts/`)

| File | Chủ đề bài học | Giá trị bổ sung so với tài liệu gốc `01..07` |
| :--- | :--- | :--- |
| [01-static-validation.md](./01-static-validation.md) | **Static Validation (ở giai đoạn đầu)** | Giải thích hội chứng *IKIWISI*, phân biệt *Static Verification vs Static Validation*, 3 kỹ thuật wireframe/Figma/storyboard và case ĐKHP |
| [02-v-model-and-vv.md](./02-v-model-and-vv.md) | **Vị trí của V&V trong Mô hình chữ V** | Verification 2 chiều bên nhánh trái, 4 cặp đối xứng V&V bên nhánh phải, *Giao thoa V&V* ở System Testing & liên hệ Sprint Agile |
| [03-vv-standards-and-note-clarifications.md](./03-vv-standards-and-note-clarifications.md) | **Chuẩn V&V (ISO 12207, CMMI, IEEE 1012) & Đính chính Ghi chú** | Bổ sung 3 chuẩn V&V quốc tế, *Dynamic Test Work Products*, **giải thích 7 điểm mâu thuẫn / mơ hồ / lỗi gõ trong các PDF trên lớp** |
| [04-informal-review.md](./04-informal-review.md) | **Informal Review (Peer Check)** | 4 hình thức (*Over-the-shoulder*, PR Review, *Pair Programming*, Peer check test case), hội chứng *LGTM* và thiếu *Audit Trail* |
| [05-walkthrough.md](./05-walkthrough.md) | **Walkthrough** | Kỹ thuật *Mental Dry-Run* qua 5 loại kịch bản, 4 vai trò, case ĐKHP `W1→W8` bắt 4 lỗ hổng logic & đối chiếu Formal Inspection |
| [06-technical-review.md](./06-technical-review.md) | **Technical Review (ARB / Design Doc)** | 5 trụ cột thẩm định, 5 vai trò chuyên gia, đối lập với Fagan Inspection (cấm vs khuyến khích bàn giải pháp) & case 50k user |
| [07-system-testing.md](./07-system-testing.md) | **System Testing (E2E trên môi trường thật)** | Môi trường Staging không giả lập, hành trình E2E + 4 nhóm phi chức năng, làm rõ mâu thuẫn nhãn V&V |
| [08-lab01-guide-and-knowledge-check.md](./08-lab01-guide-and-knowledge-check.md) | **Thực hành Lab 01 & Ôn tập Knowledge Check** | Quy trình Task 1–6, cấu trúc file `.zip`, giải 4 câu Q1–Q4 Lab 01 & 5 câu Knowledge Check (lọc phần đã có) |
| [09-formal-inspection.md](./09-formal-inspection.md) | **Formal Inspection (Fagan / IEEE 1028)** | 5 vai trò nghiêm ngặt (*Reader* paraphrasing), chỉ số định lượng (*Preparation/Inspection Rate, Defect Density*), quy trình 6 bước & cấm bàn giải pháp |
| [10-static-testing-reading-techniques-and-process.md](./10-static-testing-reading-techniques-and-process.md) | **Mở rộng Static Testing (PBR, 6 vai trò & 5 vs 6 bước)** | Kỹ thuật đọc **Perspective-based Reading (PBR)**, 6 vai trò review tổng quát, *False Positives* & giải thích **5 bước (ISTQB v4) vs 6 bước (IEEE 1028)** |
| [11-week02-notes-and-labs-guide.md](./11-week02-notes-and-labs-guide.md) | **Ghi chú Tuần 2 & Hướng dẫn Lab 02a, 02b, 03, 04** | Giải thích các dòng tốc ký trên lớp (`ERD manager`, `formal review`, `QA/QC`), quy trình Ruff + lỗi logic `S1/S2` & đính chính đề Lab Tuần 2 |

> **Các bài học liên thông ở giai đoạn sau:**
> - **Unit & Integration Fundamentals (Chiến lược tích hợp)** → xem tại [02-unit-integration-java/concepts/](../../02-unit-integration-java/concepts/01-unit-and-integration-fundamentals.md).
> - **UAT** & **Alpha / Beta Testing** → xem tại [04-quy-trinh-thuc-te/concepts/](../../04-quy-trinh-thuc-te/concepts/README.md).
> - **OAT (Operational Acceptance Testing)** & **ISO/IEC 25010** → xem tại [05-non-functional-cicd/concepts/](../../05-non-functional-cicd/concepts/README.md).

---

## Nhật ký lọc trùng lặp (Các file PDF đã được bỏ qua phần lặp)

| File ghi chú gốc của trường | Trạng thái xử lý | Nội dung đã có sẵn trong Repo tại |
| :--- | :--- | :--- |
| `Static Validation (ở giai đoạn đầu).pdf` *(gửi lại ở lượt 2)* | **Trùng 100%** với file đã chuyển thể ở lượt 1 | [01-static-validation.md](./01-static-validation.md) |
| `Requirements quality và checklist review.pdf` | **Bỏ qua phần lặp (~95%)**; giữ đính chính nhỏ ở file `03` | [05-review-checklist-c1-c6.md](../05-review-checklist-c1-c6.md) |
| `Static Testing.pdf` & `Static testing overview...pdf` | **Bỏ qua phần lặp (~80%)**; tách PBR, 6 vai trò, User Manuals & 5 vs 6 bước sang file `10` | [04-static-vs-dynamic-testing.md](../04-static-vs-dynamic-testing.md) & [01-seven-testing-principles.md](../01-seven-testing-principles.md) |
| `Dynamic testing.pdf` | **Bỏ qua phần lặp (~90%)**; giữ *Dynamic Work Products* & làm rõ 3-value BVA | [04-static-vs-dynamic-testing.md](../04-static-vs-dynamic-testing.md), [06-viet-test-case-chuan.md](../06-viet-test-case-chuan.md) & [01-test-design/](../../01-test-design/README.md) |
| `Week 01.pdf` | **Bỏ qua phần lặp lý thuyết (~80%)**; tách phần Task 1–6 & Q1–Q4 sang file `08`, giải thích `30%/70%` & `QA/QC` sang file `03` | [01-seven-testing-principles.md](../01-seven-testing-principles.md), [06-viet-test-case-chuan.md](../06-viet-test-case-chuan.md), [07-viet-bug-report-chuan.md](../07-viet-bug-report-chuan.md) |
| `Week 02.pdf` | **Bỏ qua phần lặp V&V/C1–C9 (~35%)**; tách Lab 02a, 02b, 03, 04 & giải thích tốc ký sang file `11` | [03-verification-vs-validation.md](../03-verification-vs-validation.md) & [05-review-checklist-c1-c6.md](../05-review-checklist-c1-c6.md) |
| `Ôn tập.pdf` | **Bỏ qua phần lặp (~85%)**; tách Knowledge Check sang file `08`, tách **ISO/IEC 25010** sang Giai đoạn 5 | [00-nen-tang/01..07](../README.md) & [01-test-design/01..02](../../01-test-design/README.md) |
| `Verification và Validation.pdf` | **Bỏ qua phần lặp định nghĩa/ví dụ (~70%)**; giữ 3 chuẩn quốc tế & sửa lỗi copy-paste sang file `03` | [03-verification-vs-validation.md](../03-verification-vs-validation.md) |
