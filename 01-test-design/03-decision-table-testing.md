# Decision Table Testing — Kiểm thử Bảng Quyết định

> **Nguồn tham khảo:** ISTQB CTFL Syllabus v4.0.1 (Section 4.2.3), `Dynamic_testing.pdf`, `SRS-SmartQueue.pdf` (`FR-AUTH-02`, `BR-02`).

Khi một chức năng không chỉ phụ thuộc vào 1 ô nhập liệu đơn lẻ mà phụ thuộc vào **sự kết hợp của nhiều điều kiện logic cùng lúc** (chứa `AND`, `OR`, ngoại lệ chéo), EP và BVA đơn lẻ rất dễ bỏ sót tổ hợp. **Decision Table Testing** là công cụ mạnh nhất để đảm bảo **không sót bất kỳ tổ hợp logic nào**.

---

## 1. Cấu trúc của một Decision Table

Một bảng quyết định gồm 4 phần:
1. **Conditions (Điều kiện đầu vào):** Liệt kê theo từng hàng (thường là giá trị Boolean `T/F` hoặc nhiều mức như `Bronze/Silver/Gold`).
2. **Actions (Hành động / Kết quả đầu ra):** Liệt kê các kết quả hoặc hành vi tương ứng của hệ thống theo từng hàng bên dưới.
3. **Rules (Các cột luật — R1, R2...):** Mỗi cột là một tổ hợp cụ thể của các điều kiện và hành động đi kèm.
4. **Entries (Giá trị trong ô):**
   - Điều kiện: `T` (True), `F` (False), `-` (*Don't care* — không ảnh hưởng tới kết quả), hoặc giá trị cụ thể.
   - Hành động: `X` / `Yes` (thực hiện hành động đó) hoặc trống / `No` (không thực hiện).

---

## 2. Quy trình 5 bước thiết kế & rút gọn Decision Table

1. **Bước 1 — Xác định Conditions & Actions:** Đọc kỹ SRS/User Story, liệt kê tất cả điều kiện độc lập và các hành động đầu ra có thể xảy ra.
2. **Bước 2 — Tính số tổ hợp tối đa:** Nếu có $n$ điều kiện nhị phân (`T/F`), bảng đầy đủ có **$2^n$ cột**. Nếu điều kiện có số trạng thái khác nhau ($k_1, k_2, \dots$), tổng số cột là $k_1 \times k_2 \times \dots$
3. **Bước 3 — Điền Actions cho từng cột:** Dựa trên Test Oracle (quy tắc nghiệp vụ) để xác định kết quả của từng tổ hợp.
4. **Bước 4 — Rút gọn bảng (Collapsing / Simplification):**
   - **Loại bỏ tổ hợp bất khả thi (*Infeasible combinations*):** Ví dụ *Khách chưa đăng nhập = False* nhưng *Hạng VIP = True* (về logic không tồn tại cùng lúc).
   - **Gộp các cột giống nhau (*Don't care `-`*):** Nếu 2 cột chỉ khác nhau ở đúng 1 điều kiện (`T` và `F`) nhưng cho ra **cùng một tập Actions**, ta gộp 2 cột thành 1 và đánh dấu điều kiện đó là `-` *(không quan tâm)*.
5. **Bước 5 — Sinh Test Cases:** Mỗi cột (Rule) sau khi rút gọn trở thành **ít nhất 1 Test Case** cụ thể.

---

## 3. Công thức tính Coverage (ISTQB v4.0)

$$\text{Decision Table Coverage (\%)} = \left( \frac{\text{Số cột (rules) khả thi đã được test}}{\text{Tổng số cột (rules) khả thi trong bảng}} \right) \times 100\%$$

---

## 4. Ví dụ thực tế 1: Chính sách Miễn phí vận chuyển & Giảm giá (TechStore)

**Đặc tả nghiệp vụ:**
Khách hàng mua hàng trên TechStore chịu 3 điều kiện:
- **C1:** Là thành viên VIP (`T/F`)
- **C2:** Tổng đơn hàng `>= 500.000đ` (`T/F`)
- **C3:** Nhập mã `FREESHIP` hợp lệ (`T/F`)

**Quy tắc xử lý (Actions):**
- **A1 (Miễn phí ship):** Nếu là **VIP**, HOẶC có mã **`FREESHIP`**, HOẶC đơn **`>= 500.000đ`** đều được miễn phí ship.
- **A2 (Giảm thêm 10%):** Chỉ áp dụng nếu là **VIP VÀ** đơn **`>= 500.000đ`**.
- **A3 (Thu phí ship chuẩn 30k):** Nếu không thỏa A1.

### Bảng đầy đủ ($2^3 = 8$ rules)

| Điều kiện / Hành động | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
| :--- | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| **C1: Thành viên VIP?** | T | T | T | T | F | F | F | F |
| **C2: Đơn `>= 500.000đ`?** | T | T | F | F | T | T | F | F |
| **C3: Có mã `FREESHIP`?** | T | F | T | F | T | F | T | F |
| **A1: Miễn phí ship** | X | X | X | X | X | X | X | |
| **A2: Giảm thêm 10%** | X | X | | | | | | |
| **A3: Thu phí ship 30k** | | | | | | | | X |

### Rút gọn bảng bằng ký hiệu `-` (*Don't care*)
- `R1` & `R2` có `C1=T, C2=T`, chỉ khác `C3` nhưng đều ra `A1 + A2` → gộp thành **Rule 1 (`T, T, -`)**.
- `R3` & `R4` có `C1=T, C2=F`, chỉ khác `C3` nhưng đều ra `A1` → gộp thành **Rule 2 (`T, F, -`)**.
- `R5` & `R6` có `C1=F, C2=T`, chỉ khác `C3` nhưng đều ra `A1` → gộp thành **Rule 3 (`F, T, -`)**.
- `R7` (`F, F, T`) → ra `A1` → **Rule 4**.
- `R8` (`F, F, F`) → ra `A3` → **Rule 5**.

| Điều kiện / Hành động | Rule 1 | Rule 2 | Rule 3 | Rule 4 | Rule 5 |
| :--- | :-: | :-: | :-: | :-: | :-: |
| **C1: Thành viên VIP?** | T | T | F | F | F |
| **C2: Đơn `>= 500.000đ`?** | T | F | T | F | F |
| **C3: Có mã `FREESHIP`?** | `-` | `-` | `-` | T | F |
| **A1: Miễn phí ship** | X | X | X | X | |
| **A2: Giảm thêm 10%** | X | | | | |
| **A3: Thu phí ship 30k** | | | | | X |

*(Từ 8 ca giảm xuống còn **5 ca chuẩn**, phủ 100% logic mà không bỏ sót trường hợp nào!)*

---

## 5. Ví dụ thực tế 2: Đăng nhập trên SmartQueue (`SRS-SmartQueue.pdf` — `FR-AUTH-02`)

Dựa trên đặc tả `FR-AUTH-02` (`POST /api/auth/login`):
- **C1:** Nhập đủ `email` và `password`? (`T/F`)
- **C2:** `email` + `password` khớp DB? (`T/F`)
- **C3:** Tài khoản đang `active`? (`T/F`)
- **C4:** `role` đã chọn khớp với tài khoản? (`T/F`)

*(Lưu ý thứ tự ưu tiên kiểm tra lỗi trong backend! Nếu `C1 = F` thì trả về `400` ngay; nếu `C1 = T, C2 = F` thì trả về `401` không cần check `C3, C4` để tránh lộ thông tin tài khoản bị khóa hay không!)*

| Điều kiện / Kết quả | R1 | R2 | R3 | R4 | R5 |
| :--- | :-: | :-: | :-: | :-: | :-: |
| **C1: Đủ email & password?** | F | T | T | T | T |
| **C2: Đúng thông tin đăng nhập?** | `-` | F | T | T | T |
| **C3: Tài khoản `active`?** | `-` | `-` | F | T | T |
| **C4: Khớp `role` đã chọn?** | `-` | `-` | `-` | F | T |
| **Kết quả HTTP Status** | `400` | `401` | `403` | `403` | `200` (Cấp JWT) |

> **Giá trị lớn nhất của Decision Table trong ví dụ này:** Nó buộc team (Dev & PO & Tester) phải làm rõ **thứ tự ưu tiên khi 2 lỗi xảy ra cùng lúc** — ví dụ: nếu nhập **sai mật khẩu** của một tài khoản **đang bị khóa**, hệ thống phải trả về `401` (sai thông tin) chứ KHÔNG được trả về `403` (tài khoản bị khóa), vì trả về `403` sẽ làm lộ rằng mật khẩu đó đúng hoặc email đó có tồn tại!

---

## 6. Khi nào KHÔNG nên dùng Decision Table?

- Khi số lượng điều kiện quá lớn (ví dụ `> 6–7` điều kiện độc lập → $2^7 = 128$ cột khó quản lý); lúc này hãy cân nhắc tách nhỏ logic, hoặc dùng **Pairwise Testing** nếu các tham số không ràng buộc chéo phức tạp.
- Khi kết quả phụ thuộc vào **lịch sử các bước trước đó** theo chuỗi thời gian → Hãy dùng **State Transition Testing**.
