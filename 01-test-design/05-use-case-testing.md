# Use Case Testing — Kiểm thử theo Ca sử dụng

> **Nguồn tham khảo:** ISTQB CTFL Syllabus (Use Case / Scenario-based Testing), `SRS-SmartQueue.pdf`, *Agile Testing* (Crispin & Gregory).

Trong khi EP/BVA tập trung vào từng trường dữ liệu và Decision Table tập trung vào tổ hợp logic, **Use Case Testing** nhìn hệ thống từ **góc độ quy trình nghiệp vụ đầu-cuối (End-to-End Business Process)** thông qua sự tương tác giữa **Tác nhân (Actor)** và **Hệ thống (System)**.

---

## 1. Cấu trúc chuẩn của một Use Case

Một tài liệu Use Case đầy đủ gồm 6 thành phần:
1. **Actors (Tác nhân):** Người dùng hoặc hệ thống bên ngoài tương tác (VD: *Khách hàng*, *Quản trị viên*, *Cổng thanh toán VNPAY*).
2. **Preconditions (Điều kiện tiên quyết):** Trạng thái cần thỏa mãn trước khi bắt đầu Use Case (VD: *Đã đăng nhập, giỏ hàng có ít nhất 1 sản phẩm*).
3. **Main Success Scenario (Luồng chính / Happy Path):** Chuỗi các bước tương tác lý tưởng từ đầu đến khi hoàn thành mục tiêu nghiệp vụ.
4. **Alternative Flows (Luồng rẽ nhánh thay thế):** Các lựa chọn hợp lệ khác để đạt mục tiêu (VD: *Chọn thanh toán COD thay vì VNPAY; áp dụng mã giảm giá*).
5. **Exception / Error Flows (Luồng ngoại lệ / Lỗi):** Khi điều kiện không thỏa hoặc hệ thống gặp sự cố (VD: *Sản phẩm hết hàng, thẻ bị từ chối, mất kết nối cổng thanh toán*).
6. **Postconditions (Hậu điều kiện):** Trạng thái cuối cùng của dữ liệu/hệ thống sau khi kết thúc (VD: *Đơn hàng được tạo, tồn kho bị trừ, giỏ hàng được làm rỗng* — hoặc *Không có đơn hàng nào được tạo nếu thanh toán thất bại*).

---

## 2. Quy trình chuyển hóa Use Case thành Test Cases

1. **Bước 1:** Xác định **1 Test Case cho Main Success Scenario (Happy Path)** đi từ bước đầu đến bước cuối.
2. **Bước 2:** Với **mỗi nhánh Alternative Flow**, tạo ít nhất 1 Test Case đi theo luồng chính tới điểm rẽ nhánh → đi hết nhánh thay thế → quay lại luồng chính (hoặc kết thúc).
3. **Bước 3:** Với **mỗi nhánh Exception Flow**, tạo ít nhất 1 Test Case kích hoạt điều kiện lỗi tại đúng bước tương ứng và kiểm tra cả **thông báo lỗi** lẫn **Postcondition** (VD: tồn kho không bị trừ nhầm!).
4. **Bước 4:** Kết hợp với **EP/BVA** để điền dữ liệu test cụ thể cho từng bước nhập liệu trong kịch bản.

### Tiêu chuẩn Coverage
- Bao phủ **100% các luồng** trong Use Case: `1 Main Flow + tất cả Alternative Flows + tất cả Exception Flows`.

---

## 3. Ví dụ hoàn chỉnh: Use Case "Đặt hàng & Thanh toán (Checkout)" trên TechStore

- **Use Case ID:** `UC-CHK-01`
- **Primary Actor:** Khách hàng đã đăng nhập (`Customer`)
- **Supporting Actor:** Cổng thanh toán (`Payment Gateway`)
- **Preconditions:** Giỏ hàng có ít nhất 1 sản phẩm còn hàng; khách hàng đang ở trang `/cart`.

### Luồng kịch bản
- **Main Success Scenario (M):**
  1. Khách hàng bấm nút **"Tiến hành thanh toán"** → hệ thống hiển thị trang Checkout với địa chỉ mặc định và danh sách sản phẩm.
  2. Khách hàng kiểm tra địa chỉ giao hàng và chọn phương thức vận chuyển tiêu chuẩn.
  3. Khách hàng chọn thanh toán **COD**.
  4. Khách hàng bấm **"Đặt hàng"**.
  5. Hệ thống kiểm tra tồn kho, tạo đơn hàng trạng thái `CONFIRMED`, trừ tồn kho, xóa giỏ hàng và hiển thị trang xác nhận mã đơn hàng.
- **Alternative Flows (A):**
  - `A1 (Áp dụng voucher hợp lệ)`: Tại bước 2, khách hàng nhập mã `SALE20` → hệ thống trừ 20% vào tổng thanh toán → tiếp tục bước 3.
  - `A2 (Thanh toán online VNPAY)`: Tại bước 3, chọn `VNPAY` → hệ thống tạo đơn `PENDING_PAYMENT`, chuyển hướng sang cổng VNPAY → thanh toán thành công → callback cập nhật đơn thành `PAID`, trừ tồn kho, xóa giỏ hàng.
  - `A3 (Thêm địa chỉ giao hàng mới)`: Tại bước 2, chọn "Địa chỉ mới", nhập hợp lệ và lưu làm địa chỉ nhận hàng.
- **Exception Flows (E):**
  - `E1 (Sản phẩm hết hàng tại thời điểm bấm Đặt hàng)`: Tại bước 5, tồn kho thực tế `<` số lượng đặt (do người khác vừa mua hết) → hệ thống từ chối tạo đơn, giữ nguyên giỏ hàng và báo rõ sản phẩm nào hết hàng.
  - `E2 (Mã voucher không hợp lệ / hết hạn)`: Tại `A1`, nhập mã hết hạn/sai → hiển thị thông báo lỗi cụ thể, tổng tiền không đổi.
  - `E3 (Thanh toán VNPAY thất bại / Người dùng bấm Hủy tại cổng)`: Tại `A2`, callback báo thất bại → đơn chuyển `PAYMENT_FAILED`, tồn kho **không bị trừ**, người dùng có thể thử thanh toán lại.
  - `E4 (Chưa có địa chỉ giao hàng)`: Tại bước 4, nếu bỏ trống/chưa chọn địa chỉ → chặn submit và báo lỗi validate.

### Bộ Test Cases sinh ra từ `UC-CHK-01`

| Test ID | Kịch bản phủ | Tóm tắt bước thực hiện | Expected Postcondition & Output |
| :--- | :--- | :--- | :--- |
| `TC_UC_01` | `M` (Main Flow) | Checkout với địa chỉ mặc định + COD + không voucher → bấm Đặt hàng | Đơn `CONFIRMED`, tồn kho giảm đúng số lượng, giỏ hàng trống |
| `TC_UC_02` | `M + A1` | Tại Checkout nhập voucher `SALE20` hợp lệ → COD → Đặt hàng | Tổng tiền giảm đúng 20%, lượt dùng voucher giảm 1, đơn `CONFIRMED` |
| `TC_UC_03` | `M + A2` | Chọn VNPAY → Thanh toán thành công trên sandbox | Đơn chuyển `PAID`, tồn kho giảm, giỏ hàng trống |
| `TC_UC_04` | `M + A3` | Tạo địa chỉ mới ở bước 2 → Đặt hàng | Đơn lưu đúng địa chỉ mới |
| `TC_UC_05` | `M + E1` | User khác mua hết tồn kho trước khi bấm Đặt hàng | Báo lỗi hết hàng, **không tạo đơn**, **tồn kho không âm** |
| `TC_UC_06` | `M + A1 + E2` | Nhập voucher hết hạn `EXPIRED10` | Báo lỗi voucher hết hạn, tổng tiền giữ nguyên |
| `TC_UC_07` | `M + A2 + E3` | Chọn VNPAY → Bấm Hủy giao dịch trên cổng VNPAY | Đơn `PAYMENT_FAILED` / hủy, **tồn kho không bị trừ**, không mất giỏ hàng |
| `TC_UC_08` | `M + E4` | Bỏ trống địa chỉ giao hàng → Bấm Đặt hàng | Chặn đặt hàng, highlight trường bắt buộc |

---

## 4. So sánh Use Case Testing với các kỹ thuật khác

| Điểm mạnh | Điểm yếu / Cần bổ sung |
| :--- | :--- |
| - Kiểm chứng được **tích hợp liên thông nhiều màn hình/service** (Integration & System Testing).<br>- Dễ hiểu cho cả PO, Dev, Tester (gắn với ngôn ngữ nghiệp vụ và BDD Given-When-Then).<br>- Bắt tốt lỗi **luồng xử lý & Postcondition** (ví dụ: thanh toán fail nhưng quên hoàn kho). | - **Không đi sâu vào giá trị biên và tổ hợp điều kiện nhỏ** bên trong từng form.<br>- Phụ thuộc vào chất lượng tài liệu Use Case (nếu Use Case thiếu Exception Flows thì test cũng sót).<br>- **Giải pháp:** Dùng Use Case làm khung E2E, kết hợp **EP/BVA** và **Decision Table** cho các bước nhập liệu & tính toán chi tiết! |
