# Equivalence Partitioning — Phân vùng Tương đương (EP)

> **Nguồn tham khảo:** ISTQB CTFL Syllabus v4.0.1 (Section 4.2.1), `Week1_Further_Reading.pdf`, `Dynamic_testing.pdf`, `SRS-SmartQueue.pdf` (`FR-QUE-02`, `BR-10`).

**Equivalence Partitioning (EP)** là kỹ thuật thiết kế test **Black-box** nền tảng nhất, xuất phát trực tiếp từ **Nguyên lý kiểm thử số 2** (*Exhaustive testing is impossible* — không thể test vét cạn mọi giá trị đầu vào).

---

## 1. Bản chất & Nguyên lý hoạt động

- **Định nghĩa (ISTQB v4.0):** Chia tập dữ liệu (đầu vào, đầu ra, giá trị nội bộ, tham số giao diện, giá trị liên quan đến thời gian...) thành các **phân vùng (partitions / equivalence classes)** sao cho mọi phần tử nằm trong cùng một phân vùng được kỳ vọng là sẽ được hệ thống **xử lý theo cùng một logic**.
- **Giả định cốt lõi:** Nếu một giá trị đại diện trong phân vùng phát hiện ra lỗi, mọi giá trị khác trong phân vùng đó cũng sẽ phát hiện cùng lỗi đó (và ngược lại, nếu một giá trị đúng thì cả phân vùng được coi là hành xử tương đương). Do đó, **chỉ cần chọn 1 giá trị đại diện cho mỗi phân vùng**.
- **Hai loại phân vùng bắt buộc phải tìm đủ:**
  1. **Valid Partition (Phân vùng hợp lệ):** Chứa các giá trị hệ thống phải **chấp nhận** và xử lý bình thường.
  2. **Invalid Partition (Phân vùng không hợp lệ):** Chứa các giá trị hệ thống phải **từ chối** hoặc xử lý qua nhánh ngoại lệ/báo lỗi.

> **Lưu ý:** Không chỉ input mới có EP! Bạn có thể chia phân vùng cho **Output** (ví dụ: mức thuế `0%`, `10%`, `20%` hoặc loại vé `VIP`, `Standard`, `Free`), rồi truy ngược xem input nào tạo ra từng output partition đó.

---

## 2. Phân loại & Quy tắc chia phân vùng chuẩn

### A. Dữ liệu liên tục / có thứ tự (Ordered / Range)
*Ví dụ 1 (Lab 01):* Số tín chỉ `Credits` là số nguyên từ `1` đến `24`.
- Phân vùng không hợp lệ dưới: `Credits < 1` *(VD chọn `-5` hoặc `0`)*
- Phân vùng hợp lệ: `1 ≤ Credits ≤ 24` *(VD chọn `12`)*
- Phân vùng không hợp lệ trên: `Credits > 24` *(VD chọn `30`)*
- *(Mở rộng kiểu dữ liệu)*: Không phải số nguyên (`12.5`, `"abc"`), để trống (`""`/`null`).

### B. Dữ liệu rời rạc / danh sách liệt kê (Discrete / Enum)
*Ví dụ 2 (TechStore — Phương thức thanh toán):* Chỉ hỗ trợ `COD`, `VNPAY`, `MOMO`.
- Mỗi phương thức hợp lệ xử lý logic khác nhau → **nên tách thành 3 phân vùng hợp lệ riêng** (`{COD}`, `{VNPAY}`, `{MOMO}`), không gộp chung vì code rẽ nhánh `switch/if` khác nhau!
- Phân vùng không hợp lệ: Giá trị ngoài danh sách (`"CRYPTO"`, `""`, `null`).

### C. Dữ liệu chuỗi & định dạng (String / Pattern)
*Ví dụ 3 (SmartQueue `FR-AUTH-15` — Email quên mật khẩu tối đa 254 ký tự):*
- Độ dài hợp lệ: `1–254` ký tự đúng định dạng `local@domain.tld`.
- Độ dài không hợp lệ: rỗng (`0` ký tự), quá dài (`> 254` ký tự).
- Định dạng không hợp lệ: thiếu `@`, thiếu domain (`user@`), chứa khoảng trắng, ký tự đặc biệt sai chuẩn.

### D. Dữ liệu thời gian (Date / Range)
*Ví dụ 4 (SmartQueue `BR-10` — Đặt lịch hẹn từ hôm nay đến 30 ngày tới):*
- Hợp lệ: Ngày trong `[today, today + 30]` **VÀ** thuộc ngày làm việc **VÀ** không nằm trong danh sách ngày loại trừ.
- Không hợp lệ: Ngày trong quá khứ (`< today`), quá xa (`> today + 30`), ngày phòng ban nghỉ/ngày lễ loại trừ.

---

## 3. EP cho nhiều tham số: Each Choice vs. All Combinations

Khi hệ thống có nhiều ô nhập liệu (ví dụ `k` trường), có 2 chiến lược kết hợp các phân vùng theo ISTQB v4.0:

### 1. Each Choice Coverage (Phủ từng lựa chọn — Phổ biến nhất)
- Mỗi phân vùng (hợp lệ và không hợp lệ) của **từng** tham số xuất hiện ít nhất 1 lần trong bộ test.
- **Quy tắc quan trọng khi ghép ca Invalid:** Mỗi test case chỉ nên chứa **tối đa 1 giá trị Invalid** (còn các tham số khác giữ nguyên ở **Valid**).
  - *Tại sao?* Nếu cùng 1 lúc nhập `Email sai` + `Mật khẩu ngắn` + `Tuổi âm`, khi hệ thống báo lỗi, lỗi này sẽ **che khuất (defect masking)** các kiểm tra của trường còn lại — bạn không biết hàm validate của các trường kia có chạy hay không!
- **Số test case tối thiểu:** Bằng số phân vùng lớn nhất của 1 tham số (nếu ghép hợp lệ chung) hoặc `1 (all valid) + tổng số invalid partitions` (nếu cô lập từng ca invalid).

### 2. All Combinations (Tất cả tổ hợp — Đầy đủ nhưng tốn kém)
- Kết hợp mọi phân vùng của tham số 1 với mọi phân vùng của tham số 2...
- Nếu có 3 tham số, mỗi tham số có 4 phân vùng → `4 × 4 × 4 = 64` ca. Chỉ nên dùng khi ít tham số và logic phụ thuộc chéo nhau mạnh (hoặc chuyển sang dùng **Decision Table** / **Pairwise**).

---

## 4. Công thức tính Coverage (Độ bao phủ EP)

$$\text{EP Coverage (\%)} = \left( \frac{\text{Số phân vùng đã được test ít nhất 1 lần}}{\text{Tổng số phân vùng đã xác định (Valid + Invalid)}} \right) \times 100\%$$

Để đạt **100% EP Coverage**, mọi phân vùng hợp lệ **và** không hợp lệ đều phải có ít nhất 1 test case chạm tới.

---

## 5. Ví dụ hoàn chỉnh: Form Giảm giá theo hạng Thành viên (TechStore)

**Đặc tả nghiệp vụ:** Hệ thống TechStore áp dụng mức chiết khấu dựa trên **Tổng giá trị đơn hàng** (`total`, số nguyên VNĐ `>= 0`) và **Hạng thành viên** (`tier`):
- Đơn `< 500.000đ`: Giảm `0%`
- Đơn từ `500.000đ` đến `< 2.000.000đ`: Giảm `5%`
- Đơn `>= 2.000.000đ`: Giảm `10%`
- Hạng thành viên: `Standard`, `Silver`, `Gold` (nhận thêm ưu đãi riêng theo từng nhánh).

### Bước 1: Liệt kê các phân vùng
- **Tham số `total`:**
  - `EP_T_INV1`: `total < 0` *(Không hợp lệ — số âm)*
  - `EP_T_INV2`: Không phải số / rỗng *(Không hợp lệ — sai kiểu)*
  - `EP_T_V1`: `0 <= total < 500.000` *(Hợp lệ — nhóm giảm 0%)*
  - `EP_T_V2`: `500.000 <= total < 2.000.000` *(Hợp lệ — nhóm giảm 5%)*
  - `EP_T_V3`: `total >= 2.000.000` *(Hợp lệ — nhóm giảm 10%)*
- **Tham số `tier`:**
  - `EP_M_V1`: `"Standard"`
  - `EP_M_V2`: `"Silver"`
  - `EP_M_V3`: `"Gold"`
  - `EP_M_INV1`: Giá trị khác / `null` / `""`

### Bước 2: Thiết kế bộ Test Case (Each Choice + Cô lập Invalid)

| Test ID | `total` | `tier` | Phân vùng được phủ | Expected Result |
| :--- | :--- | :--- | :--- | :--- |
| `TC_EP_01` | `200.000` | `Standard` | `EP_T_V1`, `EP_M_V1` | Giảm `0%` |
| `TC_EP_02` | `1.000.000` | `Silver` | `EP_T_V2`, `EP_M_V2` | Áp dụng mức giảm `5%` + ưu đãi `Silver` |
| `TC_EP_03` | `3.500.000` | `Gold` | `EP_T_V3`, `EP_M_V3` | Áp dụng mức giảm `10%` + ưu đãi `Gold` |
| `TC_EP_04` | `-10.000` | `Standard` | `EP_T_INV1` | Báo lỗi: Tổng tiền không hợp lệ (`400`) |
| `TC_EP_05` | `"abc"` | `Standard` | `EP_T_INV2` | Báo lỗi: Định dạng số không hợp lệ (`400`) |
| `TC_EP_06` | `1.000.000` | `"Diamond"` | `EP_M_INV1` | Báo lỗi: Hạng thành viên không tồn tại (`400`) |

→ Chỉ với **6 Test Cases**, ta đã đạt **100% EP Coverage** (`9/9` phân vùng)!

---

## 6. Lỗi thường gặp & Giới hạn của EP

- **Lỗi 1 — Gộp các phân vùng xử lý khác nhau làm một:** Ví dụ gộp `0 <= total < 500k` và `500k <= total < 2M` thành 1 phân vùng "hợp lệ" là sai, vì hành vi/công thức tính giảm giá của chúng khác nhau.
- **Lỗi 2 — Quên các phân vùng Invalid ngầm định:** Quên test `null`, chuỗi rỗng, dấu âm, ký tự chữ cái, sai kiểu dữ liệu.
- **Giới hạn của EP:** EP chỉ chọn giá trị *bên trong* phân vùng (ví dụ `1.000.000`), nên **dễ bỏ sót lỗi tại ranh giới** (ví dụ Dev viết `total > 500000` thay vì `total >= 500000`). Vì vậy EP **luôn phải đi kèm Boundary Value Analysis (BVA)**!
