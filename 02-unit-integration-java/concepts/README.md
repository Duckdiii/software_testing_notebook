# Concepts Bổ sung — Giai đoạn 2: Unit & Integration Test với Java

Thư mục này chuyển thể phần lý thuyết nền tảng còn thiếu từ tài liệu `Unit Testing & Integration Testing.pdf`, đồng thời lọc trùng lặp và đính chính các điểm dễ nhầm so với bộ hướng dẫn Java Giai đoạn 2.

## Danh sách các bài học chuyên sâu (`concepts/`)

| File | Chủ đề bài học | Nội dung trọng tâm |
| :--- | :--- | :--- |
| [01-unit-and-integration-fundamentals.md](./01-unit-and-integration-fundamentals.md) | **Nguyên lý Unit & Integration Testing & Chiến lược Tích hợp** | Ẩn dụ *Xe đạp điện / Viên gạch*, 3 chiến lược **Big Bang / Top-Down (Stub) / Bottom-Up (Driver)**, minh họa `pytest` vs Java & case ĐKHP |

---

## Nhật ký lọc trùng lặp & Đính chính (Giai đoạn 2)

- **Đã bỏ qua phần trùng lặp:** Định nghĩa 5 loại *Test Doubles* (`Dummy, Stub, Fake, Spy, Mock`) và `Stub` (`when().thenReturn()`) vs `Mock` (`verify()`) đã được trình bày chi tiết tại [02-mockito-assertj.md](../02-mockito-assertj.md).
- **Điểm đính chính quan trọng:** File `Verification và Validation.pdf` (trang 3) mắc lỗi copy-paste đặt dòng *"Điều này được thực hiện mà không cần chạy phần mềm"* ngay dưới tiêu đề *Unit Testing & Integration Testing*. Chuẩn xác: **Unit & Integration Testing là Dynamic Testing (CÓ biên dịch và chạy code thật), nhưng phục vụ Verification** (kiểm chứng hàm/module/API khớp `Module Design` và `Interface Spec`). → Xem [03-vv-standards-and-note-clarifications.md](../../00-nen-tang/concepts/03-vv-standards-and-note-clarifications.md).
