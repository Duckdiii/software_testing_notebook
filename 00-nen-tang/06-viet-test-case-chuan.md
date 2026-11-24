# Viết Test Case chuẩn (Test Case Design & Specification)

> **Nguồn tham khảo:** ISO/IEC/IEEE 29119-3, ISTQB CTFL Syllabus v4.0.1, `Week_01.pdf`, `Week1_Further_Reading.pdf`, `Dynamic_testing.pdf`, `Test_Cases_23110009.csv`.

Một Test Case chuyên nghiệp không phải là vài dòng ghi chú mơ hồ để một mình mình hiểu — nó phải đủ rõ ràng để **bất kỳ tester nào khác cũng thực thi lại được chính xác cùng một kết quả**.

---

## 1. Các khái niệm bắt buộc trước khi viết Test Case

| Khái niệm | Định nghĩa | Ví dụ (Đăng ký môn học — Lab 01) |
| :--- | :--- | :--- |
| **Test Basis** | Tài liệu làm căn cứ để phân tích & thiết kế test (SRS, User Story, Acceptance Criteria, luật nghiệp vụ). | Quy tắc R1–R4 của hệ thống đăng ký môn học. |
| **Test Condition** | Một khía cạnh/tình huống có thể kiểm chứng được trích từ Test Basis. | Kiểm tra biên dưới hợp lệ của số tín chỉ (`credits = 1`). |
| **Test Oracle** | Nguồn chuẩn đáng tin cậy dùng để xác định **Expected Result** (SRS, quy định pháp lý, chuyên gia nghiệp vụ). | R1 quy định: `1 <= credits <= 24` → với `credits = 1` phải được chấp nhận. |
| **Expected Result** | Kết quả **nên** xảy ra, được suy ra từ Test Oracle **trước khi** chạy phần mềm. | `ACCEPTED: Registration allowed.` |
| **Actual Result** | Hành vi thực tế quan sát được **sau khi** chạy phần mềm. | `REJECTED: Registration not allowed.` |
| **Test Verdict** | Phán quyết so sánh: `Actual == Expected` → **Pass**; `Actual != Expected` → **Fail**. | `Fail` (vì Actual khác Expected). |

> **Cảnh báo sai lầm phổ biến:**
> 1. **Circular Reasoning:** Mở app lên bấm thử xem nó ra gì rồi chép vào cột `Expected Result` → Sai hoàn toàn! `Expected Result` phải được xác định từ **Test Oracle trước khi chạy**.
> 2. **Nhầm lẫn ca Negative Test:** Khi nhập dữ liệu sai (ví dụ `credits = 0`) với `Expected = REJECTED`, và hệ thống thực tế trả về `REJECTED` → Kết luận là **PASS** (hệ thống chặn đúng yêu cầu), không phải Fail!

---

## 2. Cấu trúc chuẩn của một Test Case (theo ISO/IEC/IEEE 29119-3)

Một tài liệu Test Case chuẩn (trên Excel/CSV, Jira/Zephyr, TestRail) cần có các trường sau:

| Trường bắt buộc / Khuyến nghị | Mô tả chi tiết |
| :--- | :--- |
| **Test Case ID** | Mã định danh duy nhất, dễ phân nhóm theo module (ví dụ: `TC_AUTH_001`, `TC_CHK_002`). |
| **Title / Objective** | Mô tả ngắn gọn mục tiêu kiểm tra của ca này (*Verify that...* / *Kiểm tra...*). |
| **Requirement / Test Basis Ref** | Liên kết ngược về mã yêu cầu (`R1`, `US-12`) để tạo **Traceability Matrix**. |
| **Priority** | Độ ưu tiên thực thi (`High`, `Medium`, `Low`) dựa theo rủi ro nghiệp vụ. |
| **Preconditions** | Trạng thái hệ thống/dữ liệu phải có **trước khi** thực hiện Bước 1 (ví dụ: *Đã đăng nhập tài khoản khách hàng; giỏ hàng có 1 sản phẩm giá 500.000đ; form đã reset*). |
| **Test Data** | Giá trị dữ liệu đầu vào cụ thể, không ghi chung chung (ví dụ: ghi `credits=1, enrollment=25, prerequisite=Yes` thay vì *"nhập số hợp lệ"*). |
| **Steps to Execute** | Từng bước hành động đánh số thứ tự rõ ràng, mỗi bước chỉ làm 1 thao tác nguyên tử. |
| **Expected Result** | Kết quả mong đợi **cụ thể và quan sát được** (thông báo chính xác, HTTP status, trạng thái DB, chuyển trang). |
| **Actual Result** | Kết quả thực tế ghi nhận lúc chạy test *(chỉ điền khi thực thi)*. |
| **Status / Verdict** | `Pass` / `Fail` / `Blocked` / `Not Run`. |
| **Postconditions** *(tùy chọn)* | Trạng thái hệ thống sau khi test xong (ví dụ: *Đơn hàng chuyển trạng thái `PENDING` trong DB*). |
| **Evidence / Notes** | Tên file ảnh chụp màn hình, log API, hoặc Bug ID liên kết nếu `Fail`. |

---

## 3. Nguyên tắc vàng để viết Test Case "sạch" và hiệu quả

1. **Atomic & Focused (Mỗi ca 1 mục tiêu):** Tránh nhồi nhét kiểm tra cả đăng ký, áp voucher và hủy đơn vào cùng 1 test case quá dài; khi Fail sẽ khó khoanh vùng nguyên nhân.
2. **Independent & Repeatable (Độc lập & Lặp lại được):** Không để TC02 phụ thuộc vào dữ liệu rác do TC01 để lại. Luôn reset state về trạng thái chuẩn ở bước Precondition.
3. **Isolate Conditions (Cô lập biến khi test biên):** Khi kiểm tra điều kiện biên của `Credits` (`1`), hãy giữ cố định các biến khác (`Enrollment = 25`, `Prerequisite = Yes`) ở vùng hợp lệ để lỗi (nếu có) không bị điều kiện khác che khuất!
4. **Cân bằng Positive + Negative + Boundary:** Một chức năng luôn cần cả ca hợp lệ (*Happy Path*), ca giá trị biên (*Boundary*), và ca sai/thiếu/ngoại lệ (*Negative*).
5. **Tránh câu chữ mơ hồ:** Không viết *"Hệ thống chạy đúng"* ở Expected Result; hãy viết *"Hiển thị thông báo `ACCEPTED: Registration allowed.` và tổng tiền bằng `400.000đ`"*.

---

## 4. Ví dụ thực tế từ Lab 01 (`Test_Cases_23110009.csv`)

Quy tắc nghiệp vụ (Oracle):
- **R1:** `1 <= Credits <= 24`
- **R2:** `0 <= Enrollment <= 39`
- **R3:** `Prerequisite = Yes`
- **R4:** Chỉ `ACCEPTED` khi thỏa đồng thời R1, R2, R3.

| Test ID | Req | Precondition | Test Data | Steps | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| `TC01` | R1–R4 | Mở trang `CourseRegistration_Buggy.html`, form ở trạng thái trống | `Credits=12`, `Enrollment=25`, `Prerequisite=Yes` | 1. Nhập Credits = 12<br>2. Nhập Enrollment = 25<br>3. Tick Prerequisite<br>4. Bấm `Check Registration` | `ACCEPTED` *(ca hợp lệ chuẩn)* | `ACCEPTED` | **Pass** |
| `TC02` | R1 | Như trên | `Credits=1`, `Enrollment=25`, `Prerequisite=Yes` | Tương tự với `Credits=1` | `ACCEPTED` *(biên dưới hợp lệ của Credits)* | `REJECTED` | **Fail** *(Evidence: TC02.jpg)* |
| `TC03` | R1, R2 | Như trên | `Credits=24`, `Enrollment=39`, `Prerequisite=Yes` | Tương tự với `Credits=24`, `Enrollment=39` | `ACCEPTED` *(biên trên hợp lệ cả 2 trường)* | `ACCEPTED` | **Pass** |
| `TC04` | R1 | Như trên | `Credits=0`, `Enrollment=25`, `Prerequisite=Yes` | Tương tự với `Credits=0` | `REJECTED` *(dưới biên `Credits < 1`)* | `REJECTED` | **Pass** |
| `TC06` | R2 | Như trên | `Credits=12`, `Enrollment=40`, `Prerequisite=Yes` | Tương tự với `Enrollment=40` | `REJECTED` *(vượt biên `Enrollment > 39`)* | `ACCEPTED` | **Fail** *(Evidence: TC06.jpg)* |
| `TC07` | R3 | Như trên | `Credits=12`, `Enrollment=25`, `Prerequisite=No` | Không tick Prerequisite, bấm `Check Registration` | `REJECTED` *(thiếu môn tiên quyết)* | `REJECTED` | **Pass** |

---

## 5. Mẫu Test Case sẵn dùng cho TechStore / CRM (Deliverable GĐ 0)

Dưới đây là mẫu áp dụng cho chức năng **Áp dụng Voucher khi Checkout trên TechStore** để bạn tái sử dụng cho Deliverable:

- **Test Case ID:** `TC_TECHSTORE_VOUCHER_001`
- **Title:** Kiểm tra áp dụng mã giảm giá hợp lệ tại ngưỡng giá trị đơn hàng tối thiểu (`500.000đ`)
- **Requirement Ref:** `FR-CHK-05` (Mã `SALE20` giảm `20%`, tối đa `150.000đ`, áp dụng cho đơn hàng có tổng tiền hàng `>= 500.000đ`)
- **Priority:** `High`
- **Preconditions:**
  1. Người dùng đã đăng nhập với tài khoản hợp lệ (`user01@techstore.vn`).
  2. Giỏ hàng đang có đúng 1 sản phẩm giá `500.000đ` (tạm tính = `500.000đ`).
  3. Mã `SALE20` đang ở trạng thái hoạt động và còn lượt sử dụng.
- **Test Data:** `Voucher Code = "SALE20"`, `Subtotal = 500000`
- **Steps:**
  1. Điều hướng tới trang Checkout (`/checkout`).
  2. Nhập `SALE20` vào ô `Mã giảm giá`.
  3. Nhấn nút `Áp dụng`.
- **Expected Result:**
  1. Hiển thị thông báo thành công: `"Áp dụng mã giảm giá SALE20 thành công"`.
  2. Mục `Giảm giá` hiển thị `-100.000đ`.
  3. Mục `Tổng thanh toán` (trước phí ship) cập nhật thành `400.000đ`.
- **Actual Result:** *(Điền khi chạy)*
- **Status:** *(Pass / Fail / Blocked)*
- **Evidence:** *(Link ảnh/log nếu có)*
