# Pairwise Testing — Kiểm thử Tổ hợp theo Cặp (All-Pairs)

> **Nguồn tham khảo:** ISTQB CTFL/CTAL Combinatorial Testing, NIST Combinatorial Testing Studies, Microsoft PICT Documentation.

Khi hệ thống có nhiều tham số cấu hình hoặc trường lựa chọn độc lập nhau, việc thử mọi tổ hợp dẫn đến **bùng nổ tổ hợp (combinatorial explosion)**. **Pairwise Testing** (còn gọi là *All-Pairs*) giải quyết bài toán này bằng toán học: giảm hàng trăm/hàng nghìn ca xuống còn vài chục ca mà vẫn giữ tỷ lệ phát hiện lỗi cực cao.

---

## 1. Cơ sở khoa học của Pairwise Testing

- Các nghiên cứu thực nghiệm của **NIST (National Institute of Standards and Technology)** và ngành công nghiệp phần mềm chỉ ra:
  - Khoảng **50%–70%** lỗi phần mềm gây ra bởi **1 tham số đơn lẻ** (1-way interaction).
  - Khoảng **70%–95%** lỗi được kích hoạt bởi sự tương tác giữa **2 tham số** (2-way / pairwise interaction — ví dụ: *Trình duyệt Safari + Thanh toán VNPAY*, hoặc *Voucher phần trăm + Tài khoản VIP*).
  - Rất ít lỗi chỉ xuất hiện khi có đúng 3 hoặc 4 tham số cùng lúc ở giá trị đặc biệt.
- **Định nghĩa Pairwise:** Tạo tập hợp test case nhỏ nhất sao cho **mọi cặp giá trị có thể có giữa bất kỳ 2 tham số nào đều xuất hiện ít nhất 1 lần**.

---

## 2. Bài toán minh họa: Ma trận Checkout trên TechStore

Giả sử trang Checkout TechStore cần test tương thích trên 4 tham số độc lập:
1. **Browser (3):** `Chrome`, `Firefox`, `Safari`
2. **OS (3):** `Windows`, `macOS`, `Android`
3. **Payment (3):** `COD`, `VNPAY`, `MOMO`
4. **Voucher (3):** `None`, `Percent (20%)`, `Fixed (50k)`

- **Test toàn bộ (Exhaustive / Full Combinatorial):** $3 \times 3 \times 3 \times 3 = \mathbf{81}$ test cases! (Nếu thêm 2 tham số nữa là gần 1.000 ca).
- **Tổng số cặp giữa 2 tham số cần phủ:** Có $\binom{4}{2} = 6$ cặp tham số; mỗi cặp tham số có $3 \times 3 = 9$ cặp giá trị → Tổng cộng $6 \times 9 = \mathbf{54}$ cặp giá trị cần xuất hiện.
- **Với Pairwise:** Chỉ cần **9 Test Cases** (giảm gần **90%** công sức!) là phủ 100% toàn bộ 54 cặp đó!

### Bảng Orthogonal Array $L_9(3^4)$ — 9 ca phủ 100% mọi cặp

| Test ID | Browser | OS | Payment | Voucher |
| :---: | :--- | :--- | :--- | :--- |
| `TC_PW_01` | Chrome | Windows | COD | None |
| `TC_PW_02` | Chrome | macOS | VNPAY | Percent |
| `TC_PW_03` | Chrome | Android | MOMO | Fixed |
| `TC_PW_04` | Firefox | Windows | VNPAY | Fixed |
| `TC_PW_05` | Firefox | macOS | MOMO | None |
| `TC_PW_06` | Firefox | Android | COD | Percent |
| `TC_PW_07` | Safari | Windows | MOMO | Percent |
| `TC_PW_08` | Safari | macOS | COD | Fixed |
| `TC_PW_09` | Safari | Android | VNPAY | None |

**Kiểm chứng nhanh:** Chọn bất kỳ 2 cột nào (ví dụ `Browser` & `Voucher`), bạn sẽ thấy đủ cả 9 cặp:
`(Chrome, None)`, `(Chrome, Percent)`, `(Chrome, Fixed)`, `(Firefox, None)`, `(Firefox, Percent)`, `(Firefox, Fixed)`, `(Safari, None)`, `(Safari, Percent)`, `(Safari, Fixed)`!

---

## 3. Xử lý Ràng buộc Bất khả thi (Constraints) — Cực kỳ thực tế!

Trong thực tế, **Safari không tồn tại trên Windows/Android hiện đại** (hoặc giả sử Safari chỉ test trên macOS/iOS)! Nếu dùng mảng trực giao mù quáng, bạn sẽ sinh ra ca vô nghĩa như `(Safari, Windows)` hoặc `(Safari, Android)`.

### Cách dùng công cụ **Microsoft PICT** để sinh Pairwise có ràng buộc

Microsoft PICT (Pairwise Independent Combinatorial Tool) là công cụ miễn phí rất phổ biến để tạo bộ test pairwise. Bạn viết file model `checkout_model.txt`:

```text
# Định nghĩa các tham số và giá trị
Browser: Chrome, Firefox, Safari
OS:      Windows, macOS, Android, iOS
Payment: COD, VNPAY, MOMO
Voucher: None, Percent, Fixed
Guest:   Yes, No

# Ràng buộc thực tế (Constraints)
IF [Browser] = "Safari" THEN [OS] IN {"macOS", "iOS"};
IF [Guest] = "Yes" THEN [Voucher] = "None";
```

Sau đó chạy lệnh:
```bash
pict checkout_model.txt
```
PICT sẽ tự động sinh tập test tối giản (~12–14 ca) đảm bảo:
1. Phủ 100% mọi cặp hợp lệ.
2. Không bao giờ sinh tổ hợp vi phạm constraint (`Safari + Windows` hay `Guest + Voucher`).

*(Nếu không cài PICT, bạn có thể dùng Python `allpairspy` hoặc các công cụ online như Pairwise PictOnline / ACTS của NIST).*

---

## 4. Phương pháp thiết kế Pairwise thủ công nhanh (khi phỏng vấn không có tool)

Nếu đi phỏng vấn gặp câu hỏi yêu cầu lập nhanh bộ pairwise cho 3–4 biến ít giá trị:
1. Sắp xếp các tham số theo **số lượng giá trị giảm dần** (biến nhiều giá trị nhất đứng trước).
2. Chọn 2 biến đầu tiên (`A` có $m$ giá trị, `B` có $n$ giá trị) → liệt kê đủ $m \times n$ cặp làm $m \times n$ dòng đầu tiên.
3. Với biến thứ 3 (`C`), điền từng giá trị vào từng dòng sao cho **tạo ra nhiều cặp mới nhất** với `A` và `B`; nếu còn cặp nào với `A` hoặc `B` bị thiếu thì thêm dòng phụ và điền các giá trị còn thiếu.
4. Lặp lại cho biến thứ 4 (`D`)...

---

## 5. Khi nào NÊN và KHÔNG NÊN dùng Pairwise?

| Nên dùng | Không nên chỉ dùng Pairwise |
| :--- | :--- |
| - Kiểm thử ma trận cấu hình (OS × Browser × Device × Network).<br>- Form tìm kiếm/lọc có 5–10 dropdown độc lập.<br>- Kiểm thử tham số cấu hình API/hệ thống có nhiều flag bật/tắt. | - Khi có **1 tổ hợp 3–4 biến cực kỳ rủi ro về nghiệp vụ** (VD: `VIP + FlashSale + Voucher + Hoàn tiền`) → hãy bổ sung ca đó thủ công hoặc dùng **Decision Table**!<br>- Không dùng Pairwise thay thế cho BVA của giá trị số liên tục (vẫn phải chọn giá trị biên bằng BVA làm input cho mỗi phân vùng trước khi ghép cặp). |
