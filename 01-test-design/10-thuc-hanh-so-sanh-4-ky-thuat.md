# Thực hành: Thiết kế Test cho 3 Form bằng 4 Kỹ thuật & So sánh Hiệu quả

> **Mục tiêu (Roadmap GĐ 1 — Thực hành):** Lấy từng form (**Đăng ký**, **Checkout**, **Voucher**) trong project của bạn (**TechStore**), thiết kế test bằng **4 nhóm kỹ thuật** rồi so sánh số lượng test case và khả năng phát hiện bug của từng kỹ thuật.

Để bài thực hành khách quan và có giá trị tham chiếu thực tế, ta xây dựng **đặc tả chuẩn + đoạn logic xử lý minh họa** (dùng cho góc nhìn White-box) cho từng form, sau đó thiết kế bộ test theo 4 kỹ thuật:
1. **Kỹ thuật 1 — Black-box Cơ bản:** Equivalence Partitioning (EP) + Boundary Value Analysis (BVA).
2. **Kỹ thuật 2 — Black-box Tổ hợp & Trạng thái:** Decision Table / State Transition / Pairwise.
3. **Kỹ thuật 3 — White-box:** Branch / Condition / Basis Path Coverage trên mã nguồn xử lý.
4. **Kỹ thuật 4 — Experience-based:** Error Guessing + Exploratory Testing (Charter).

---

## Form 1: Form Đăng ký Tài khoản (`Registration`) — TechStore

### Đặc tả & Logic xử lý mẫu
- **Email:** Bắt buộc, đúng định dạng `a@b.c`, tối đa `254` ký tự, không trùng trong hệ thống.
- **Mật khẩu (`password`):** `8–32` ký tự, bắt buộc có ít nhất 1 chữ hoa, 1 chữ thường và 1 chữ số.
- **Xác nhận mật khẩu (`confirmPassword`):** Phải khớp `password`.
- **Tuổi (`age` — tùy chọn/bắt buộc theo policy):** Số nguyên từ `16` đến `100`.
- **Điều khoản (`agreeTerms`):** Bắt buộc tick (`true`).

```java
// Logic kiểm tra mẫu (dùng cho góc nhìn White-box)
public RegisterResult register(String email, String pwd, String confirmPwd, int age, boolean agreeTerms) {
    if (!agreeTerms) return RegisterResult.ERR_TERMS;                      // D1
    if (email == null || !isValidEmail(email) || email.length() > 254)     // D2 (C1 || C2 || C3)
        return RegisterResult.ERR_EMAIL;
    if (pwd == null || pwd.length() < 8 || pwd.length() > 32 || !hasComplexity(pwd)) // D3
        return RegisterResult.ERR_PASSWORD;
    if (!pwd.equals(confirmPwd)) return RegisterResult.ERR_MISMATCH;       // D4
    if (age < 16 || age > 100) return RegisterResult.ERR_AGE;              // D5
    if (userRepo.existsByEmail(email)) return RegisterResult.ERR_EXISTS;   // D6
    userRepo.save(...);
    return RegisterResult.SUCCESS;
}
```

### Thiết kế Test bằng 4 kỹ thuật cho Form Đăng ký

| Kỹ thuật | Test Cases thiết kế | Số lượng TC | Lỗi đặc thù phát hiện được |
| :--- | :--- | :---: | :--- |
| **1. EP + BVA** | - `age`: `15` (Invalid), `16` (Valid biên dưới), `17`, `99`, `100` (Valid biên trên), `101` (Invalid)<br>- `pwd` độ dài: `7`, `8`, `32`, `33` ký tự; thiếu chữ hoa / thiếu số<br>- `email`: hợp lệ (`254` ký tự), `255` ký tự, thiếu `@`, trùng email<br>- `confirmPwd` không khớp; `agreeTerms = false` | **14 TC** | - Lỗi off-by-one tại biên tuổi (`age > 16` thay vì `>= 16`) hoặc biên độ dài mật khẩu `8`/`32`/`254`.<br>- Lỗi không kiểm tra độ dài tối đa email `254`. |
| **2. Decision Table** | Kết hợp `agreeTerms (T/F)` × `emailValid (T/F)` × `emailNew (T/F)` × `pwdValid (T/F)` × `match (T/F)` (rút gọn theo thứ tự ưu tiên validate). | **6 TC** | - Lỗi ưu tiên validate (VD: hiển thị `"Email đã tồn tại"` trước khi check định dạng hoặc không đồng bộ thông báo lỗi nhiều trường cùng lúc). |
| **3. White-box (Branch + Condition)** | Phủ đủ nhánh True/False của `D1..D6` và **từng điều kiện con** trong `D2` (`null`, invalid format, `> 254`), `D3` (`null`, `< 8`, `> 32`, `!complexity`), `D5` (`< 16`, `> 100`). | **10 TC** | - Lỗi `NullPointerException` nếu nhánh check `email == null` viết sau `email.length() > 254`!<br>- Lỗi short-circuit bỏ qua `hasComplexity(pwd)` khi test độ dài. |
| **4. Error Guessing + Exploratory** | - Email có khoảng trắng đầu/cuối `" test@gmail.com "` hoặc viết hoa `Test@Gmail.com` (trùng với `test@gmail.com` trong DB!)<br>- Mật khẩu chỉ toàn khoảng trắng hoặc chứa emoji/Unicode<br>- **Double-click** nút Đăng ký nhanh 2 lần (Race condition) | **5 TC** | - **Case-sensitivity bug:** cho phép đăng ký trùng `User@mail.com` và `user@mail.com`!<br>- **Race condition:** bấm đúp tạo 2 user hoặc trả về `500 DuplicateKeyException` thay vì `409`. |

---

## Form 2: Form Thanh toán / Đặt hàng (`Checkout`) — TechStore

### Đặc tả & Logic xử lý mẫu
- Chọn địa chỉ giao hàng (bắt buộc).
- Phí ship: Miễn phí nếu **VIP** hoặc **Tổng hàng `>= 500.000đ`** hoặc có mã **`FREESHIP`**; ngược lại `30.000đ`.
- Thanh toán: `COD` (giới hạn đơn `<= 5.000.000đ`) hoặc `VNPAY` (không giới hạn).
- Tồn kho: Mọi sản phẩm phải có `quantity <= stock` tại thời điểm bấm Đặt hàng.

### Thiết kế Test bằng 4 kỹ thuật cho Form Checkout

| Kỹ thuật | Test Cases thiết kế | Số lượng TC | Lỗi đặc thù phát hiện được |
| :--- | :--- | :---: | :--- |
| **1. EP + BVA** | - Tổng hàng quanh mốc freeship: `499.999đ`, `500.000đ`, `500.001đ`<br>- Đơn COD quanh trần `5.000.000đ`: `5.000.000đ` (OK), `5.000.001đ` (Chặn COD)<br>- Số lượng đặt quanh `stock`: `0`, `1`, `stock`, `stock + 1`<br>- Địa chỉ trống / hợp lệ | **10 TC** | - Lỗi tính phí ship ở đúng `500.000đ` (`>` vs `>=`).<br>- Lỗi cho phép thanh toán COD vượt `5.000.000đ`. |
| **2. Decision Table + State Transition** | - **Decision Table:** `VIP (T/F)` × `Total >= 500k (T/F)` × `FREESHIP (T/F)` × `Payment (COD/VNPAY)` (rút gọn còn ~7 rules)<br>- **State Transition:** `Pending → Paid → Shipping → Completed`; thử các **Invalid transitions**: Hủy đơn khi đã `Shipping`, Webhook `PAID` đến khi đơn đã `Cancelled` | **12 TC** | - **Lỗi tổ hợp:** Khách chọn `COD` nhưng tổng đơn `490.000đ` + phí ship `30.000đ = 520.000đ` → hệ thống lại tưởng đơn `>= 500k` và miễn ship sai (vòng lặp logic)!<br>- **Lỗi state:** Webhook VNPAY đến muộn làm đơn đã hủy hồi sinh thành `Paid`. |
| **3. White-box (Branch + Basis Path)** | Đọc hàm `checkout()` & `createOrder()`: kiểm tra nhánh `catch (PaymentGatewayTimeoutException)`, nhánh rollback transaction `@Transactional` khi trừ kho sản phẩm thứ 2 bị fail, nhánh tính tổng tiền từ giá DB thay vì giá client gửi lên. | **7 TC** | - **Lỗi Transaction Rollback:** Sản phẩm 1 bị trừ kho xong, sản phẩm 2 hết hàng ném exception nhưng thiếu `@Transactional` nên sản phẩm 1 không được hoàn kho!<br>- **Lỗi tin tưởng giá từ client** nếu không đọc từ DB. |
| **4. Error Guessing + Exploratory** | - Mở 2 tab mua **cái cuối cùng trong kho (`stock = 1`)** bấm Đặt hàng cùng lúc (*Concurrency*)<br>- Chọn VNPAY → bấm Hủy trên cổng thanh toán → kiểm tra giỏ hàng và tồn kho<br>- Nhấn nút **Back** trình duyệt sau khi đặt hàng xong rồi submit lại<br>- Thử gửi `quantity = -1` qua API / DevTools | **6 TC** | - **Tồn kho âm (`stock = -1`)** khi 2 người mua cùng lúc.<br>- **Mất sạch giỏ hàng** khi khách bấm Hủy ở cổng VNPAY.<br>- Tạo 2 đơn giống hệt nhau khi bấm Back hoặc double-submit. |

---

## Form 3: Form Áp dụng Mã giảm giá (`Voucher`) — TechStore

### Đặc tả & Logic xử lý mẫu
- Mã `SALE20`: Giảm `20%`, giảm tối đa `150.000đ`, áp dụng cho đơn tối thiểu `500.000đ`.
- Mã `FLAT50K`: Giảm thẳng `50.000đ`, áp dụng cho đơn tối thiểu `200.000đ`.
- Mỗi đơn chỉ áp dụng **1 voucher**; voucher có ngày hiệu lực `[startDate, endDate]`, giới hạn `usageLimit`, và mỗi user chỉ dùng 1 lần.
- Không áp dụng cho sản phẩm đang gắn cờ `isFlashSale = true`.

```java
// Logic tính giảm giá mẫu (ẩn chứa lỗi tiềm ẩn để White-box/BVA tìm!)
public int applyVoucher(Voucher v, Order order) {
    if (v == null || !v.isActive()) throw new BadRequestException("Mã không hợp lệ");
    if (now.isAfter(v.getEndDate())) throw new BadRequestException("Mã hết hạn");
    if (order.getEligibleSubtotal() < v.getMinOrder()) throw new BadRequestException("Chưa đạt giá trị tối thiểu");
    int discount = v.getType() == PERCENT
        ? (int) (order.getEligibleSubtotal() * v.getValue() / 100.0)
        : v.getValue();
    if (v.getMaxDiscount() > 0) {
        discount = Math.min(discount, v.getMaxDiscount());
    }
    return Math.min(discount, order.getTotal()); // Tránh giảm âm tiền
}
```

### Thiết kế Test bằng 4 kỹ thuật cho Form Voucher

| Kỹ thuật | Test Cases thiết kế | Số lượng TC | Lỗi đặc thù phát hiện được |
| :--- | :--- | :---: | :--- |
| **1. EP + BVA** | - Đơn hàng quanh ngưỡng tối thiểu `SALE20`: `499.999đ`, `500.000đ`, `500.001đ`<br>- Đơn hàng quanh trần giảm tối đa `150.000đ` (tức đơn `750.000đ`): `749.000đ`, `750.000đ`, `751.000đ`, `1.000.000đ`<br>- `FLAT50K` với đơn `200.000đ` vs `199.999đ`<br>- Thời gian hiệu lực: `startDate - 1s`, `startDate`, `endDate`, `endDate + 1s`<br>- `usageLimit`: còn `1` lượt vs `0` lượt | **12 TC** | - Lỗi bỏ quên cap tối đa `150.000đ` (đơn 2 triệu bị giảm 400k!).<br>- Lỗi biên ngày hết hạn (hết hạn ngay lúc `00:00:00` ngày cuối thay vì `23:59:59`). |
| **2. Decision Table** | Kết hợp các điều kiện: `VoucherValid (T/F)` × `InDateRange (T/F)` × `UsageRemaining (T/F)` × `UserNotUsedYet (T/F)` × `HasFlashSaleOnly (T/F)` × `MeetsMinOrder (T/F)` (rút gọn còn 8 rules). | **8 TC** | - **Lỗi xung đột Flash Sale:** Giỏ có 1 sp thường `300k` + 1 sp Flash Sale `300k` = tổng `600k` → hệ thống lấy `600k` đi so sánh `>= 500k` và giảm 20% lên cả sp Flash Sale! |
| **3. White-box (Branch + Condition)** | Kiểm tra từng nhánh trong `applyVoucher()`: `v == null`, `!v.isActive()`, `now.isBefore(startDate)` (phát hiện **code thiếu check `startDate`!**), `v.getMaxDiscount() > 0` vs `== 0`, làm tròn số thực `(int)` khi `20%` ra số lẻ, nhánh `Math.min(discount, order.getTotal())`. | **8 TC** | - **Bug cực hiểm thấy ngay qua White-box:** Đoạn code mẫu trên có check `now.isAfter(endDate)` nhưng **QUÊN check `now.isBefore(startDate)`** — cho phép dùng voucher chưa đến ngày mở! |
| **4. Error Guessing + Exploratory** | - Nhập `" sale20 "` (khoảng trắng + chữ thường)<br>- Áp `SALE20` cho đơn `500k`, **sau đó quay lại xóa bớt sản phẩm** xuống còn `300k` rồi bấm Đặt hàng!<br>- Đổi từ `SALE20` sang `FLAT50K` rồi gỡ voucher<br>- Dùng 2 request song song khi `usageLimit` chỉ còn `1` lượt | **6 TC** | - **Bug nghiệp vụ kinh điển:** Áp voucher lúc giỏ đủ `500k`, sau đó giảm số lượng xuống `200k` nhưng session vẫn giữ nguyên mức giảm `-100k` (hoặc giảm âm tiền)!<br>- Không tự trim/uppercase mã voucher. |

---

## Tổng hợp So sánh 4 Kỹ thuật trên cả 3 Form

| Tiêu chí so sánh | 1. EP + BVA | 2. Decision Table & State | 3. White-box (Branch/Path) | 4. Error Guessing & Exploratory |
| :--- | :---: | :---: | :---: | :---: |
| **Tổng số Test Case (3 form)** | **36 TC** | **26 TC** | **25 TC** | **17 TC** |
| **Nhóm bug bắt tốt nhất** | Sai toán tử biên (`<` vs `<=`), sai ngưỡng min/max, dữ liệu ngoài miền | Sai thứ tự ưu tiên validate, xung đột điều kiện (`FlashSale + Voucher`), chuyển trạng thái trái phép | Thiếu check `startDate`, lỗi `NullPointerException` do thứ tự `\|\|`, thiếu `@Transactional` rollback | Race condition (double-click, 2 tab), đổi giỏ hàng sau khi áp voucher, case-sensitive email, Back button |
| **Khả năng phát hiện bug tính trên 1 TC** | Rất cao cho lỗi nhập liệu & biên | Rất cao cho lỗi nghiệp vụ phức tạp | Rất cao cho nhánh ẩn & xử lý ngoại lệ | Cực cao cho lỗi thực tế người dùng & bảo mật |
| **Phát hiện được lỗi "quên chức năng" (Missing Requirement)?** | Trung bình | Tốt (Decision Table làm lộ ô trống) | **Kém** (White-box chỉ test code *đã viết*, không thấy thứ Dev *quên viết* trừ khi đối chiếu lại spec!) | **Rất tốt** (tư duy phản biện & *Antisocial Tour*) |
| **Độ phụ thuộc vào mã nguồn** | Không (Black-box) | Không (Black-box) | Bắt buộc có source code | Không (hoặc kết hợp cả 2) |

### Kết luận & Chiến lược kết hợp tối ưu
- **Không một kỹ thuật đơn lẻ nào bắt được hết 100% lỗi!**
- Bộ khung chuẩn cho mọi form nghiệp vụ:
  1. Dùng **EP + BVA** để khóa chặt tất cả giới hạn biên của từng ô nhập liệu.
  2. Dùng **Decision Table / State Transition** để khóa chặt logic phối hợp nhiều điều kiện và vòng đời trạng thái.
  3. Dùng **White-box** (ở tầng Unit/Integration test — GĐ 2) để soi nhánh ẩn, short-circuit và rollback giao dịch.
  4. Dùng **Exploratory / Error Guessing** ở cuối để săn các lỗi *race condition*, *state inconsistency* khi người dùng thao tác bất thường.
