# Deliverable Giai đoạn 1: Test Design Document & Requirements Traceability Matrix (RTM)

> **Chuẩn áp dụng:** ISO/IEC/IEEE 29119-3 (Test Design Specification), ISTQB CTFL Syllabus v4.0.1.  
> **Đối tượng áp dụng mẫu:** Module **Checkout & Voucher** — Dự án **TechStore** *(bạn có thể giữ nguyên cấu trúc chuẩn này khi nộp/đưa vào Portfolio GitHub)*.

---

## 1. Thông tin Tài liệu (Document Control)

| Mục | Nội dung |
| :--- | :--- |
| **Mã tài liệu** | `TDD-TECHSTORE-CHK-01` |
| **Phiên bản** | `v1.0` |
| **Dự án / Module** | TechStore E-Commerce — Checkout & Voucher (`FR-CHK`, `FR-VCH`) |
| **Test Basis (Tài liệu gốc)** | SRS TechStore v1.2 (Mục 3.4 Checkout, Mục 3.5 Voucher), Business Rules `BR-CHK-01..06` |
| **Kỹ thuật thiết kế áp dụng** | EP, 2-Value/3-Value BVA, Decision Table, State Transition, Error Guessing |
| **Mục tiêu chất lượng** | 100% yêu cầu `FR`/`BR` trong phạm vi đều có ít nhất 1 Test Case truy vết được |

---

## 2. Danh sách Yêu cầu đầu vào (Test Basis Summary)

| Req ID | Mô tả yêu cầu (Test Basis) | Mức độ rủi ro |
| :--- | :--- | :---: |
| **`FR-CHK-01`** | Hệ thống kiểm tra tồn kho (`quantity <= stock`) cho mọi sản phẩm khi bấm Đặt hàng; nếu thiếu hàng phải từ chối và giữ nguyên giỏ hàng. | **High** |
| **`FR-CHK-02`** | Phí vận chuyển mặc định `30.000đ`; miễn phí ship nếu khách là `VIP` HOẶC tổng tiền hàng sau giảm `>= 500.000đ` HOẶC có mã `FREESHIP`. | **Medium** |
| **`FR-CHK-03`** | Thanh toán `COD` chỉ cho phép với tổng thanh toán `<= 5.000.000đ`; `VNPAY` không giới hạn. | **High** |
| **`FR-CHK-04`** | Vòng đời đơn hàng: `PENDING_PAYMENT → PAID → SHIPPING → COMPLETED`; chỉ cho phép hủy đơn khi ở trạng thái `PENDING_PAYMENT` hoặc `CONFIRMED` (trước khi `SHIPPING`). | **High** |
| **`FR-VCH-01`** | Mã `SALE20` giảm `20%` (tối đa `150.000đ`) cho đơn hàng hợp lệ có tổng tiền hàng `>= 500.000đ` trong thời gian hiệu lực `[startDate, endDate]`. | **High** |
| **`FR-VCH-02`** | Không áp dụng voucher giảm giá lên sản phẩm đang gắn cờ `Flash Sale`, và không tính giá trị sản phẩm Flash Sale vào ngưỡng tối thiểu `500.000đ`. | **High** |
| **`FR-VCH-03`** | Mỗi voucher có giới hạn tổng lượt sử dụng (`usageLimit > 0`); không cho phép vượt quá giới hạn khi nhiều request đồng thời. | **Critical** |

---

## 3. Phân tích Điều kiện Kiểm thử (Test Conditions)

| Condition ID | Req Ref | Điều kiện cần kiểm tra | Kỹ thuật chọn |
| :--- | :--- | :--- | :--- |
| **`COND-01`** | `FR-CHK-01` | Số lượng đặt `1`, `stock`, `stock + 1`, `0` | BVA / EP |
| **`COND-02`** | `FR-CHK-02` | Tổ hợp 3 điều kiện Freeship (`VIP`, `>= 500k`, `FREESHIP`) & biên `499.999` / `500.000` | Decision Table + BVA |
| **`COND-03`** | `FR-CHK-03` | Thanh toán COD tại biên `5.000.000` vs `5.000.001`; VNPAY vượt `5.000.000` | BVA + EP |
| **`COND-04`** | `FR-CHK-04` | Chuyển đổi trạng thái hợp lệ & thử hủy đơn khi đang `SHIPPING` / `COMPLETED` | State Transition |
| **`COND-05`** | `FR-VCH-01` | Giá trị đơn tại biên `499.999`, `500.000`, `750.000`, `1.000.000` (check cap `150k`); ngày hiệu lực | BVA + EP |
| **`COND-06`** | `FR-VCH-02` | Giỏ hàng chỉ có Flash Sale vs hỗn hợp (Thường + Flash Sale) | Decision Table |
| **`COND-07`** | `FR-VCH-03` | Voucher còn `1` lượt, 2 user bấm dùng đồng thời; đổi giỏ hàng sau khi áp voucher | BVA + Error Guessing |

---

## 4. Đặc tả Ca Kiểm thử Chi tiết (Test Cases)

| Test Case ID | Req Ref | Cond Ref | Technique | Preconditions | Input / Test Data | Steps Summary | Expected Result | Priority |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **`TC_CHK_001`** | `FR-CHK-01` | `COND-01` | BVA | User login; SP A có `stock = 5` | `quantity = 5`, `COD`, địa chỉ hợp lệ | 1. Thêm 5 SP A vào giỏ<br>2. Vào Checkout → bấm Đặt hàng | Tạo đơn thành công; `stock` của SP A giảm còn `0`; giỏ hàng trống | **High** |
| **`TC_CHK_002`** | `FR-CHK-01` | `COND-01` | BVA | User login; SP A có `stock = 5` | `quantity = 6` (hoặc SP vừa bị mua hết còn 4) | 1. Đặt `quantity = 6` (hoặc chỉnh DB `stock = 4` trước khi bấm Đặt) | Báo lỗi `"Sản phẩm A không đủ tồn kho"`, không tạo đơn, giữ giỏ hàng | **High** |
| **`TC_CHK_003`** | `FR-CHK-02` | `COND-02` | BVA + DT | User thường (non-VIP), không mã `FREESHIP` | `Subtotal = 499.999đ` | 1. Checkout đơn `499.999đ` không mã freeship | Phí ship = `30.000đ`; Tổng = `529.999đ` | **Medium** |
| **`TC_CHK_004`** | `FR-CHK-02` | `COND-02` | BVA + DT | User thường (non-VIP), không mã `FREESHIP` | `Subtotal = 500.000đ` | 1. Checkout đơn `500.000đ` | Phí ship = `0đ` (Freeship); Tổng = `500.000đ` | **Medium** |
| **`TC_CHK_005`** | `FR-CHK-02` | `COND-02` | DT | User `VIP = true` (hoặc nhập mã `FREESHIP`) | `Subtotal = 150.000đ`, `VIP = true` | 1. Checkout đơn `150.000đ` với tài khoản VIP | Phí ship = `0đ` (Freeship); Tổng = `150.000đ` | **Medium** |
| **`TC_CHK_006`** | `FR-CHK-03` | `COND-03` | BVA | User login; giỏ hợp lệ | `Total = 5.000.000đ`, `Payment = COD` | 1. Chọn COD cho đơn `5.000.000đ` → Đặt hàng | Cho phép chọn COD và đặt hàng thành công | **High** |
| **`TC_CHK_007`** | `FR-CHK-03` | `COND-03` | BVA | User login; giỏ hợp lệ | `Total = 5.000.001đ`, `Payment = COD` | 1. Chọn COD cho đơn `5.000.001đ` | Chặn/Vô hiệu hóa COD (báo vượt hạn mức COD `5.000.000đ`), cho phép chọn `VNPAY` | **High** |
| **`TC_CHK_008`** | `FR-CHK-04` | `COND-04` | State | Có đơn hàng ở trạng thái `SHIPPING` | `OrderStatus = SHIPPING`, Action = `Cancel` | 1. Mở chi tiết đơn `SHIPPING`<br>2. Bấm Hủy hoặc gọi `POST /orders/{id}/cancel` | UI không hiển thị nút Hủy; API trả `409 Conflict`, trạng thái giữ nguyên `SHIPPING` | **High** |
| **`TC_VCH_001`** | `FR-VCH-01` | `COND-05` | BVA | Mã `SALE20` đang trong hiệu lực, còn lượt | `Subtotal = 500.000đ`, `Code = SALE20` | 1. Nhập `SALE20` → bấm Áp dụng | Giảm `-100.000đ` (20%); thanh toán `400.000đ` | **High** |
| **`TC_VCH_002`** | `FR-VCH-01` | `COND-05` | BVA | Như trên | `Subtotal = 499.999đ`, `Code = SALE20` | 1. Nhập `SALE20` → bấm Áp dụng | Từ chối: `"Đơn hàng tối thiểu 500.000đ để dùng mã SALE20"` | **High** |
| **`TC_VCH_003`** | `FR-VCH-01` | `COND-05` | BVA | Như trên | `Subtotal = 1.000.000đ`, `Code = SALE20` | 1. Nhập `SALE20` → bấm Áp dụng | Giảm đúng trần tối đa `-150.000đ` (không phải `-200.000đ`); còn `850.000đ` | **High** |
| **`TC_VCH_004`** | `FR-VCH-01` | `COND-05` | EP / BVA | Mã `SALE20` chưa tới `startDate` (hoặc đã qua `endDate`) | `Now = startDate - 1 minute` | 1. Nhập `SALE20` → bấm Áp dụng | Từ chối: `"Mã giảm giá chưa đến thời gian áp dụng / đã hết hạn"` | **Medium** |
| **`TC_VCH_005`** | `FR-VCH-02` | `COND-06` | DT | Giỏ có SP Thường `300k` + SP Flash Sale `300k` | `Eligible = 300k`, `FlashSale = 300k`, `Code = SALE20` | 1. Nhập `SALE20` → bấm Áp dụng | Từ chối (tổng hàng hợp lệ `300k < 500k`)! Nếu SP Thường `500k` + Flash Sale `300k` thì chỉ giảm `20%` trên `500k` (`-100k`) | **High** |
| **`TC_VCH_006`** | `FR-VCH-03` | `COND-07` | Error Guess | Mã `SALE20` có `usageLimit = 1` | 2 user (Tab A & Tab B) cùng áp dụng & Đặt hàng đồng thời | 1. Cả 2 tab bấm Đặt hàng cùng lúc | Chỉ tối đa 1 đơn áp thành công (`usageLimit = 0`), đơn còn lại bị chặn báo hết lượt | **Critical** |
| **`TC_VCH_007`** | `FR-VCH-01` | `COND-07` | Error Guess | Giỏ đang có `500k` và đã áp `SALE20` | Giảm số lượng SP xuống còn `250k` rồi bấm Đặt hàng | 1. Áp `SALE20` lúc giỏ `500k`<br>2. Giảm giỏ còn `250k`<br>3. Bấm Đặt hàng | Hệ thống tự động gỡ `SALE20` (hoặc chặn đặt hàng và yêu cầu kiểm tra lại giá) | **High** |

---

## 5. Ma trận Truy vết Yêu cầu — Requirements Traceability Matrix (RTM)

RTM đảm bảo **truy vết hai chiều (*Bidirectional Traceability*)**:
- **Forward (`Req → Condition → Test Case`):** Chứng minh mọi yêu cầu đều có Test Case bảo vệ (không sót tính năng).
- **Backward (`Test Case → Execution → Bug → Req`):** Khi 1 Test Case Fail hoặc 1 Yêu cầu thay đổi, biết ngay Bug/Yêu cầu nào bị ảnh hưởng.

| Req ID | Mô tả tóm tắt | Test Condition(s) | Test Case ID(s) | Execution Status *(Mẫu)* | Linked Bug ID *(Mẫu)* | Coverage |
| :--- | :--- | :--- | :--- | :--- | :--- | :---: |
| **`FR-CHK-01`** | Kiểm tra tồn kho khi đặt hàng | `COND-01` | `TC_CHK_001`, `TC_CHK_002` | `1 Pass, 1 Fail` | `BUG-CHK-01` *(race condition)* | **100%** |
| **`FR-CHK-02`** | Quy tắc tính phí ship & Freeship | `COND-02` | `TC_CHK_003`, `TC_CHK_004`, `TC_CHK_005` | `3 Pass` | — | **100%** |
| **`FR-CHK-03`** | Giới hạn thanh toán COD `<= 5M` | `COND-03` | `TC_CHK_006`, `TC_CHK_007` | `2 Pass` | — | **100%** |
| **`FR-CHK-04`** | Vòng đời & quyền hủy đơn | `COND-04` | `TC_CHK_008` | `1 Pass` | — | **100%** |
| **`FR-VCH-01`** | Quy tắc giảm giá `SALE20` & hiệu lực | `COND-05`, `COND-07` | `TC_VCH_001`, `TC_VCH_002`, `TC_VCH_003`, `TC_VCH_004`, `TC_VCH_007` | `4 Pass, 1 Fail` | `BUG-VCH-02` *(giỏ giảm dưới 500k vẫn giữ voucher)* | **100%** |
| **`FR-VCH-02`** | Loại trừ sản phẩm Flash Sale | `COND-06` | `TC_VCH_005` | `1 Fail` | `BUG-VCH-03` *(tính gộp Flash Sale vào 500k)* | **100%** |
| **`FR-VCH-03`** | Giới hạn lượt dùng voucher đồng thời | `COND-07` | `TC_VCH_006` | `Not Run` *(chờ môi trường test song song)* | — | **100% Designed** |

---

## 6. Tổng kết Độ bao phủ Thiết kế (Design Coverage Summary)

- **Tổng số yêu cầu trong phạm vi:** `7` (`FR-CHK-01..04`, `FR-VCH-01..03`)
- **Số yêu cầu đã có Test Case:** `7 / 7` → **Requirement Coverage = 100%**
- **Tổng số Test Cases đã thiết kế:** `15` ca trọng tâm (gồm Happy Path, Boundary, Decision Table, State Transition, và Error Guessing)
- **Yêu cầu chuẩn bị môi trường & dữ liệu (Test Implementation Input):**
  - Tài khoản mẫu: `customer_normal@techstore.vn`, `customer_vip@techstore.vn`.
  - Seed DB: Sản phẩm thường giá `250.000đ` / `499.999đ` / `500.000đ` (`stock = 5`), Sản phẩm `Flash Sale` giá `300.000đ`, Voucher `SALE20` (`usageLimit = 1`).
