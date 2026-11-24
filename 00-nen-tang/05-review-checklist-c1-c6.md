# Review theo Checklist C1–C6 (Checklist-Based Requirements Review)

> **Nguồn tham khảo:** ISO/IEC/IEEE 29148, ISTQB CTFL v4.0.1 (Section 3.2), `Week2_Further_Reading.pdf`, `Requirements_quality_v_checklist_review.pdf`, `Software_Testing_Week2_Lab02_Checklist_Based_Review (1).pdf`.

Trong kỹ nghệ phần mềm có câu kinh điển: **"Garbage In, Garbage Out"**. Hơn 50% lỗi nghiêm trọng của phần mềm bắt nguồn từ khâu đặc tả yêu cầu (*Requirements*). **Checklist-Based Review** là một kỹ thuật **Static Testing** có cấu trúc: dùng bộ câu hỏi chuẩn hóa (checklist) để soi từng câu yêu cầu trước khi lập trình viên viết code.

---

## 1. Bước chuẩn bị: Tách FR/NFR & Tiêu chuẩn Yêu cầu chất lượng cao

Trước khi chấm theo checklist, cần tách câu nói tự nhiên của khách hàng thành các yêu cầu **nguyên tử (atomic)** và phân loại **FR vs. NFR**:

- **Functional Requirement (FR — Yêu cầu chức năng):** Hệ thống **LÀM ĐƯỢC CÁI GÌ** (hành vi, tính toán, quy tắc nghiệp vụ).  
  *Ví dụ:* Tính tổng tín chỉ, từ chối lớp đã đầy, áp dụng mã giảm giá.
- **Non-Functional Requirement (NFR — Yêu cầu phi chức năng):** Hệ thống **ĐẠT CHẤT LƯỢNG THẾ NÀO** (hiệu năng, bảo mật, độ sẵn sàng, khả năng dùng).  
  *Ví dụ:* Thời gian phản hồi `< 2s` cho `95%` request dưới tải `500` người dùng đồng thời.

### 5 tiêu chuẩn yêu cầu chất lượng cao (ISO/IEC/IEEE 29148)
1. **Unambiguous / Clear (Rõ ràng, không mơ hồ):** Chỉ có duy nhất một cách hiểu.
2. **Complete (Đầy đủ):** Nêu rõ cả luồng chính (*happy path*), điều kiện biên và luồng lỗi (*exception path*).
3. **Verifiable / Testable (Khả kiểm):** Có ngưỡng định lượng hoặc hành vi quan sát được để phân định Pass/Fail.
4. **Consistent (Nhất quán):** Không tự mâu thuẫn giữa các đoạn trong tài liệu hay giữa các stakeholder.
5. **Atomic (Nguyên tử):** Mỗi câu đặc tả một yêu cầu độc lập để dễ gán ID và truy vết.

---

## 2. Bộ Checklist C1–C6 (và mở rộng C7–C10 theo Lab 02)

> **Lưu ý về đánh số trong giáo trình:** Tài liệu `Week2_Further_Reading.pdf` (mục 4.2) liệt kê **C1–C6** làm lõi cơ bản và mở rộng tới **C9**, còn `Requirements_quality_v_checklist_review.pdf` & `Lab 02` đánh mã **C01–C10** (đổi chỗ thứ tự Clarity/Completeness một chút nhưng cùng bản chất). Bảng dưới đây trình bày trọng tâm **C1–C6** theo roadmap và bổ sung **C7–C10** để bạn không bỏ sót bất kỳ góc nhìn nào.

### Nhóm cốt lõi C1–C6

| Mã | Tiêu chí | Câu hỏi người Review phải tự vấn | Dấu hiệu nhận biết lỗi | Ví dụ TỒI ❌ → Sửa CHUẨN ✔️ |
| :---: | :--- | :--- | :--- | :--- |
| **C1** | **Completeness** *(Tính đầy đủ)* | Đã mô tả đủ input, output, điều kiện, luồng chính và các trường hợp ngoại lệ chưa? | Thiếu bước xử lý khi điều kiện không thỏa, thiếu thông báo lỗi, thiếu trạng thái rỗng/đầy. | ❌ *"Hệ thống cho phép đăng ký môn học."*<br>✔️ Bổ sung: khi lớp kín chỗ thì từ chối hay đưa vào Waitlist? Thông báo gì? |
| **C2** | **Clarity** *(Tính rõ ràng)* | Câu chữ có chính xác, chỉ hiểu theo một nghĩa duy nhất không? | Xuất hiện từ cảm tính: *"nhanh"*, *"phù hợp"*, *"thân thiện"*, *"khi cần thiết"*, *"tối ưu"*. | ❌ *"Hệ thống phản hồi thật nhanh."*<br>✔️ *"Trả về kết quả trong ≤ 2 giây cho 95% request dưới tải đỉnh."* |
| **C3** | **Consistency** *(Tính nhất quán)* | Các câu yêu cầu và phát biểu của các bên có mâu thuẫn ("đá nhau") không? | Đoạn trên ghi tối đa 24, đoạn dưới ghi > 20; bên nói bắt buộc, bên nói có thể bỏ qua. | ❌ Phòng Đào tạo: tối đa 24 tín chỉ; Khoa: cho phép 27 tín chỉ nhưng không ghi quy trình ngoại lệ. |
| **C4** | **Testability** *(Tính khả kiểm)* | Có thể kiểm chứng khách quan bằng test case cụ thể để ra kết luận Pass/Fail không? | Thiếu con số đo lường, thiếu hành vi quan sát được trên UI/API/DB. | ❌ *"Giao diện phải thân thiện với người dùng mới."*<br>✔️ *"Người dùng mới hoàn thành đặt hàng trong ≤ 3 phút không cần hướng dẫn."* |
| **C5** | **Feasibility** *(Tính khả thi)* | Với công nghệ, hạ tầng, ngân sách và ràng buộc hiện tại, hệ thống có làm nổi không? | Cam kết tuyệt đối vô lý (*uptime 100%*, *0ms latency*, *chịu 10 triệu request/giây* không ngân sách). | ❌ *"Không bao giờ nghẽn mạng dù có 10 triệu lượt bấm cùng lúc."*<br>✔️ Định nghĩa mức tải mục tiêu thực tế và cơ chế xếp hàng (*rate limit/queue*). |
| **C6** | **Boundary Conditions** *(Điều kiện biên)* | Giá trị tối thiểu, tối đa, biên đóng/mở (`<` vs `<=`), mốc 0 và rỗng đã quy định rõ chưa? | Ghi *"đến 24"*, *"dưới 40"*, *"từ 500k"* mà không nói rõ tại đúng điểm biên thì chấp nhận hay từ chối. | ❌ *"Đăng ký tối đa đến 24 tín chỉ."*<br>✔️ *"Từ 1 đến 24 tín chỉ (bao gồm cả 1 và 24)."* |

### Nhóm mở rộng C7–C10 (từ Lab 02 & Week 2)

| Mã | Tiêu chí | Câu hỏi người Review phải tự vấn | Ví dụ kiểm tra thực tế |
| :---: | :--- | :--- | :--- |
| **C7** | **Exception Handling** *(Xử lý ngoại lệ)* | Khi mất mạng, timeout cổng thanh toán, nhập sai định dạng, 2 người cùng đặt ghế cuối (*concurrency*) thì xử lý thế nào? | Quy định rõ cơ chế rollback giao dịch, khóa lạc quan/bi quan (*locking*) khi tranh chấp slot cuối. |
| **C8** | **Security & Access** *(Bảo mật & Phân quyền)* | Ai được xem, tạo, sửa, xóa, duyệt dữ liệu? Dữ liệu nhạy cảm có được bảo vệ không? | Sinh viên A có xem được lịch sử/điểm của sinh viên B không? Ai được duyệt đơn overload? |
| **C9** | **Traceability** *(Khả năng truy vết)* | Mỗi yêu cầu có gắn với nguồn stakeholder hoặc mục tiêu nghiệp vụ nào không? | Phát hiện tính năng "tự nghĩ thêm" không có stakeholder nào yêu cầu. |
| **C10** | **Prioritization** *(Phân cấp ưu tiên)* | Đã phân biệt được tính năng bắt buộc (*Must-have*) với tính năng tùy chọn (*Nice-to-have*) chưa? | Phân biệt chức năng đăng ký/thanh toán cốt lõi với chức năng gợi ý thông minh. |

---

## 3. Quy tắc đánh giá & Quy tắc vàng cho Tester

Khi áp dụng checklist lên từng yêu cầu (`REQ-01`, `REQ-02`...), ghi nhận 1 trong 4 trạng thái:
1. **Pass (Đạt):** Yêu cầu rõ ràng, đầy đủ, đo lường được, không mâu thuẫn.
2. **Issue Found / Defect (Lỗi):** Yêu cầu sai logic rõ ràng hoặc mâu thuẫn trực tiếp với yêu cầu khác.
3. **Needs Clarification (Cần làm rõ):** Yêu cầu thiếu thông tin, thiếu ngưỡng định lượng hoặc từ ngữ mơ hồ.
4. **N/A:** Tiêu chí không áp dụng cho câu yêu cầu đó (phải giải thích lý do hợp lý).

> **QUY TẮC VÀNG:** Khi gặp **Needs Clarification**, Tester **TUYỆT ĐỐI KHÔNG TỰ BỊA RA CON SỐ HAY QUY ĐỊNH** (ví dụ tự ý chốt 2 giây hay tự chốt cho phép waitlist) để lấp chỗ trống. Hãy đặt placeholder (ví dụ `[threshold]`) và gửi câu hỏi chính thức cho Stakeholder/PO xác nhận!

---

## 4. Mẫu Phiếu Ghi nhận Phát hiện (Defect Finding Log)

Một nhận xét kiểu *"Câu này viết sai"* là vô ích. Mỗi finding phải chỉ rõ **vị trí, bằng chứng trích dẫn, tác động và hành động đề xuất**:

| Trường | Ví dụ minh họa (Hệ thống Đăng ký / TechStore) |
| :--- | :--- |
| **Finding ID** | `F-01` |
| **REQ / Checklist** | `REQ-03` · `C2 (Clarity)` & `C4 (Testability)` |
| **Phân loại** | `Needs Clarification` |
| **Vị trí & Bằng chứng** | Đoạn 2, phát biểu của đại diện sinh viên: *"Quá trình đăng ký phải nhanh và không phải chờ lâu khi đông người truy cập."* |
| **Tác động (Impact)** | Dev không biết tối ưu ngưỡng độ trễ bao nhiêu; Tester không thể viết Performance Test Pass/Fail khách quan. |
| **Hành động đề xuất** | Gửi câu hỏi cho Phòng IT & Ban đào tạo chốt ngưỡng cụ thể: *"Dưới tải đồng thời `[X]` người dùng, thời gian phản hồi p95 mục tiêu là bao nhiêu giây?"* |
| **Bản nháp cải thiện** | *"Với `[X]` người dùng đồng thời, 95% yêu cầu đăng ký phải hiển thị kết quả xác nhận hoặc lý do từ chối trong vòng `[Y]` giây."* *(Chờ stakeholder duyệt `[X]`, `[Y]`)* |

---

## 5. Áp dụng nhanh cho Deliverable (TechStore / CRM)

Trước khi code hoặc test một tính năng mới (ví dụ **Form Checkout + Voucher** của TechStore), hãy lướt nhanh **C1–C6** ngay trong buổi Three Amigos:
- **C1 (Đủ):** Đã có trường hợp giỏ hàng rỗng, sản phẩm hết hàng giữa chừng khi bấm Checkout chưa?
- **C2 (Rõ):** *"Giảm giá tối đa"* là tính trước hay sau phí vận chuyển?
- **C3 (Nhất quán):** Chính sách hoàn tiền trên trang FAQ có khớp với logic hủy đơn trong tài liệu kỹ thuật không?
- **C4 (Khả kiểm):** Có mô tả rõ HTTP status / thông báo UI / trạng thái đơn hàng trong DB sau khi bấm Thanh toán không?
- **C5 (Khả thi):** Nếu gọi API cổng thanh toán bên thứ ba timeout sau 30s thì UI có chờ được không?
- **C6 (Biên):** Đơn hàng đúng bằng ngưỡng tối thiểu áp dụng voucher (ví dụ `500.000đ`), số lượng `1`, hoặc số lượng tồn kho còn đúng `1` xử lý thế nào?
