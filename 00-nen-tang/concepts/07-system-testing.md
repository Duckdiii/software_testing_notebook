# System Testing (Kiểm thử Hệ thống Toàn diện End-to-End)

> **Nguồn bài học:** `System Testing Kiểm thử toàn bộ hệ thống tích hợp hoàn chỉnh theo góc nhìn người dùng thực tế..pdf` *(Cấp độ kiểm thử thứ 3 trên nhánh phải V-Model — xem đối chiếu tại [02-v-model-and-vv.md](./02-v-model-and-vv.md))*.

---

## 1. Hình ảnh ví von: Sản xuất một chiếc xe máy

- **Unit Testing:** Thử riêng từng chiếc bugi, săm lốp đạt chuẩn chưa.
- **Integration Testing:** Lắp động cơ vào khung xe, nối bình xăng với buồng đốt xem nhiên liệu chảy vào chuẩn không.
- **System Testing:** Ráp **100% hoàn chỉnh** thành chiếc xe nguyên vẹn, dắt ra đường thật, chở hai người, bật đèn pha, rồ ga, phanh gấp trên đường trơn để kiểm chứng toàn hệ thống vận hành thực tế.

> *(Ghi chú nhỏ về tính nhất quán trong bộ note trường: file `Unit & Integration Testing` ở Tuần 2 dùng ví dụ **xe đạp điện**, còn file `System Testing` này dùng **xe máy** — bản chất minh họa test cấp độ chi tiết vs tích hợp vs toàn xe là hoàn toàn giống nhau).*

---

## 2. "Hệ thống tích hợp hoàn chỉnh" nghĩa là gì?

Khác với Unit Test và Integration Test (có thể dùng *Mock/Stub* cho phần phụ thuộc ngoài), ở **System Testing không còn giả lập nội bộ cốt lõi nữa**:
- **Frontend (Web / Mobile App)** nối thật với **Backend API**.
- **Backend** nối thật với **Database**.
- Kết nối tới **dịch vụ bên thứ ba** (môi trường sandbox/pre-prod chuẩn: cổng thanh toán VNPay/MoMo, SMS OTP, Email notification).
- Chạy trên môi trường **Staging / Pre-production** có cấu hình phần cứng, mạng và phân quyền sát môi trường thật nhất.

---

## 3. Kiểm thử E2E theo góc nhìn Người dùng thực tế (User Journey)

Người dùng cuối không quan tâm backend dùng Java hay Python, DB là SQL hay NoSQL — họ chỉ quan tâm **hành trình nghiệp vụ từ đầu đến cuối (End-to-End)** có hoàn tất trơn tru không.

**Ví dụ luồng E2E trên App đặt đồ ăn (ShopeeFood / GrabFood):**
1. Mở app trên điện thoại thật (thử cả điều kiện mạng 4G chập chờn).
2. Tìm quán phở, chọn món, nhập ghi chú *"Không hành"*.
3. Áp mã giảm giá 20.000đ.
4. Thanh toán qua thẻ ngân hàng → trừ tiền thành công.
5. Tài xế nhận đơn → cập nhật vị trí GPS thời gian thực.
6. Tài xế giao xong → app bắn Push Notification và gửi hóa đơn về email.

Nếu bất kỳ mắt xích nào gãy (VD: *đã trừ tiền nhưng đơn hàng không được tạo*, hoặc *email hóa đơn không gửi về*), **System Testing kết luận Fail**.

---

## 4. System Testing kiểm tra CẢ Chức năng lẫn Phi chức năng

Điểm đắt giá nhất của System Testing là kiểm chứng tổng thể trên cả 2 mảng:

### A. Kiểm thử Chức năng (*Functional Testing*)
Đăng ký, đăng nhập, tìm kiếm, lọc dữ liệu, tính tổng tiền/khuyến mãi, phân quyền (User thường không truy cập được màn Admin).

### B. Kiểm thử Phi chức năng (*Non-Functional Testing*)
Người dùng không chỉ cần app *"chạy được"*, mà cần **nhanh, an toàn, tương thích và ổn định**:
1. **Performance & Load Testing:** 50.000 sinh viên cùng đăng ký môn lúc 8h sáng → server có sập trắng trang không, tốc độ phản hồi có giữ `< 2 giây` không?
2. **Security Testing:** Kẻ gian có thể sửa giá tiền đơn hàng từ `1.000.000đ` xuống `1.000đ` ở request gửi lên không? Có đọc lén được mật khẩu/SĐT người khác không?
3. **Compatibility Testing:** Giao diện đẹp trên desktop nhưng mở trên iPhone 13 mini hoặc Android màn nhỏ có bị nhảy nút, mất chữ không?
4. **Reliability & Recovery Testing:** Đang thanh toán dở mà rớt mạng hoặc tắt app đột ngột thì tiền/trạng thái đơn có bị hỏng không? Server mất điện bật lại dữ liệu còn toàn vẹn không?

---

## 5. So sánh nhanh: Integration Testing vs. System Testing vs. UAT

| Cấp độ | Ai thực hiện chủ yếu? | Môi trường / Dữ liệu | Trọng tâm kiểm tra | Bản chất V&V *(Đã đính chính)* |
| :--- | :--- | :--- | :--- | :--- |
| **Integration Testing** | Developers *(hoặc SDET)* | Module A ráp Module B, nối DB/API (có thể dùng Testcontainers/Stub ngoài) | Interface, hợp đồng JSON/API, dữ liệu truyền giữa các khối có khớp thiết kế không | **Verification** |
| **System Testing** | **Đội ngũ QA / Testers chuyên nghiệp** | Toàn bộ hệ thống 100% tích hợp trên Staging | Cả hệ thống chạy **E2E** có đúng SRS, chịu tải, bảo mật và mượt theo góc nhìn user không | **Giao thoa V&V** |
| **Acceptance Testing (UAT)** | **Khách hàng thật / PO / End-users** | Môi trường nghiệm thu + dữ liệu nghiệp vụ thật | Có đáp ứng hợp đồng & giải quyết đúng bài toán kinh doanh để ký nhận & trả tiền không | **Validation** |

> [!IMPORTANT]
> **Giải thích mâu thuẫn nhãn trong tài liệu trường:** Ở trang 3 của `System Testing.pdf` (và `UAT.pdf`), bảng so sánh tạm gán System Testing ở cột góc nhìn là *"Kỹ thuật, quy trình phát triển phần mềm (Verification)"*. Tuy nhiên, đối chiếu với chính ví dụ ShopeeFood ở trang 2 (*"theo góc nhìn người dùng thực tế"*) và mô hình chữ V chuẩn (`Vị trí của V&V trong Mô hình chữ V.pdf` trang 3–4), **System Testing mang bản chất Giao thoa V&V**:
> - **Verification:** Xác minh toàn hệ thống tuân thủ đúng đặc tả SRS & kiến trúc phi chức năng (SLA `< 2s`, 50k users, bảo mật).
> - **Validation:** Xác thực sớm các hành trình người dùng thực tế E2E (đặt món, mất mạng, màn hình mobile) trước khi giao khách hàng làm UAT.
