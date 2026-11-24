# Thực hành Giai đoạn 6: Audit Portfolio, Luyện thi ISTQB & Mock Interview 45 Phút

> **Mục tiêu (3 tuần nước rút):** Biến toàn bộ kiến thức từ GĐ 0 → GĐ 5 thành hồ sơ ứng tuyển sắc bén, vượt qua bài thi ISTQB CTFL v4.0.1 và tự tin xử lý các bài test kỹ thuật trong phỏng vấn.

---

## Tuần 1: Thực hành Audit 3 Repo Portfolio & Diễn tập 3-Minute Walkthrough

### Bài tập 1: Tự chấm điểm Portfolio GitHub (Thang 15 điểm — Phải đạt `>= 13/15` mới đem đi Apply!)
Kiểm tra lần lượt cả 3 repo (`techstore-test-suite`, `qa-docs-crm-techstore`, `techstore-security-perf-lab`):
- [ ] Trang Profile GitHub đã **Pin** đúng 3 repo nổi bật nhất chưa?
- [ ] Mỗi repo đều có **README.md** trình bày: *Elevator Pitch → Kiến trúc (Mermaid) → Tech Stack → Cách chạy local → Kết quả mẫu*?
- [ ] Có **ảnh chụp màn hình thật** (Allure Dashboard, JaCoCo >= 80%, k6 summary, ZAP report) đặt trong thư mục `docs/images/` và nhúng thẳng vào README?
- [ ] Đã xóa sạch `target/`, `.idea/`, mật khẩu cứng, và kiểm tra `.gitignore` chuẩn?
- [ ] Đã tự bấm `git clone` ra một thư mục mới tinh và chạy thử lệnh trong README xem người lạ có chạy thành công không?

### Bài tập 2: Tự thu âm bài thuyết trình *3-Minute Walkthrough* (CAR)
Bật đồng hồ bấm giờ **đúng 3 phút**, giải thích dự án `techstore-test-suite` thành tiếng không nhìn giấy:
- `0:00 – 0:30` (Context): Bài toán kiểm thử E-Commerce TechStore.
- `0:30 – 2:00` (Action): Cách chia Test Pyramid (Unit/Testcontainers + REST Assured + Selenium `ThreadLocal` + CI/ZAP/k6).
- `2:00 – 3:00` (Result): Số liệu cụ thể (`93.9%` branch coverage, bắt lỗi `startDate`, Race Condition, IDOR, cải thiện `p95` `780ms → 128ms`).

---

## Tuần 2: Luyện thi ISTQB CTFL v4.0.1 & Phân tích Lỗi

### Mục tiêu điểm số theo từng Chương (Để đạt an toàn `>= 32 / 40 = 80%`)
| Chương | Tổng số câu | Mục tiêu tối thiểu | Chiến lược làm bài |
| :-: | :---: | :---: | :--- |
| **Ch. 1** | 8 | `>= 6 / 8` | Chắc 7 nguyên lý, Error/Defect/Failure, Test Process 7 bước. |
| **Ch. 2** | 5–6 | `>= 4 / 6` | Chắc Shift-left, V-Model, Test Levels vs Test Types, Retrospectives. |
| **Ch. 3** | 4 | `>= 3 / 4` | Thuộc lòng vai trò 4 loại Review (đặc biệt Moderator trong Inspection). |
| **Ch. 4** | **11** | **`>= 9 / 11`** | Làm thật kỹ bài tập tính EP, 2-value/3-value BVA, Decision Table, State 0/1-switch, Statement vs Branch! |
| **Ch. 5** | **9** | **`>= 7 / 9`** | Phân biệt Product vs Project Risk, Entry vs Exit Criteria, Three-point Estimation, Bug Lifecycle. |
| **Ch. 6** | 2 | `>= 2 / 2` | Lợi ích & Rủi ro khi áp dụng Test Automation. |

### Quy trình giải 3 đề thi thử chính thức (Sample Exam A, B, C của ISTQB v4.0.1)
1. Bấm giờ đúng **75 phút** (nếu đăng ký Extra Time tiếng Anh) — làm liền mạch một lèo không tra cứu.
2. Chấm điểm: với mỗi câu sai hoặc phân vân, đối chiếu lại đúng file ghi chú ở `00-nen-tang/` và `01-test-design/` để sửa lỗ hổng tư duy chứ không học vẹt đáp án.

---

## Tuần 3: Mock Interview Thực chiến 45 Phút (3 Đề Live Test Hay Gặp Nhất)

Hãy tự làm trọn bộ 3 bài sau **trong giới hạn 15 phút/bài** như đang ngồi trước Tech Lead:

### Đề Live Test 1 (Tư duy QA / Product): *"Thiết kế Test Case cho API/Chức năng Chuyển tiền Ngân hàng nội bộ"*
- **Dữ liệu đề bài:** Hạn mức mỗi lần chuyển `10.000đ – 50.000.000đ`, tối đa `100.000.000đ/ngày`, phí `0đ` nếu cùng ngân hàng hoặc tài khoản VIP, ngược lại `5.500đ`, bắt buộc xác thực OTP (hết hạn 60s, sai 3 lần khóa OTP 15 phút).
- **Yêu cầu tự giải (15 phút):**
  1. Liệt kê câu hỏi làm rõ PO (*phí có cộng vào trần 50M/ngày không? Chuyển vào đúng 23:59:59 reset hạn mức ngày thế nào?*).
  2. Thiết kế bảng Test Case phủ **BVA** (`9.999`, `10.000`, `50.000.000`, `50.000.001`), **Decision Table** (VIP × SameBank), **State Transition** (OTP sai 3 lần), **Concurrency** (2 tab chuyển đồng thời khi số dư còn đúng `10.000.000đ`), và **Security IDOR** (đổi `fromAccountId` sang tài khoản người khác!).

### Đề Live Test 2 (Live Code Review & Unit Test Java — SDET): *"Tìm 3 bug ẩn trong hàm sau và viết JUnit 5 test bắt đủ 3 bug"*
```java
public int calculateLoyaltyPoints(int orderTotal, boolean isMember, boolean isReturned) {
    if (isReturned) return 0;
    int points = orderTotal / 10000;
    if (isMember && orderTotal > 500_000) {
        points = points * 2;
    }
    return points;
}
```
- **3 vấn đề cần chỉ ra ngay:**
  1. **Input validation:** Chưa chặn `orderTotal < 0` (trả về điểm âm!).
  2. **Boundary bug (`>` vs `>=`):** Nếu policy là *"Từ 500.000đ trở lên được nhân đôi điểm"*, viết `>` sẽ thiếu đúng mốc `500_000`!
  3. **Integer Overflow:** Nếu đơn lớn `points * 2` có nguy cơ tràn `int` nếu không kiểm tra cận trên (hoặc dùng `long`).

### Đề Live Test 3 (Xử lý Tình huống Pipeline / DevSecOps):
- **Tình huống:** *"Sáng thứ Hai lên công ty thấy PR Pipeline đang ĐỎ ở UI Test `CheckoutUiTest` (lúc chạy tay ở máy em lại XANH), đồng thời Slack báo cuối tuần có user phản hồi xem được hóa đơn của người khác qua link `/orders/105`. Em xử lý thứ tự các việc này thế nào?"*
- **Hướng trả lời chuẩn Senior/SDET:**
  1. **Ưu tiên số 1 (Security Critical Incident):** Xác minh ngay lỗ hổng IDOR trên `/orders/{id}` → báo Tech Lead/PO hotfix gấp kiểm tra Ownership `order.userId == currentUser.id` và viết ngay REST Assured test khóa lỗi này.
  2. **Ưu tiên số 2 (Chẩn đoán Flaky UI Test):** Mở ngay **Allure Report / Screenshot / Playwright Trace** từ job CI lỗi xem fail vì lý do gì (timeout AJAX, animation, hay đụng test data với test chạy song song?). Sửa wait/isolation hoặc tạm Quarantine có ticket nếu cần mở khóa gấp cho team merge hotfix Security!
