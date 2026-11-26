# Làm rõ Điểm Mơ hồ trong Note Trường về Thiết kế Test (Stage 1)

> **Mục đích:** Giải thích các chỗ dễ nhầm khi đối chiếu phần kỹ thuật thiết kế test trong `Dynamic testing.pdf` (trang 3–4) và `Ôn tập.pdf` (trang 8–11) với bộ tài liệu chuẩn tại `01-test-design/`.

---

## 1. Làm rõ BVA trong `Dynamic testing.pdf` vs. `Ôn tập.pdf`

- **Hiện tượng dễ nhầm:** Trong `Dynamic testing.pdf` (trang 3), mục BVA viết: *"Bắt buộc test tại: Điểm biên, ngay dưới biên, và ngay trên biên"* rồi lấy luôn ví dụ `{0, 1, 2}` và `{23, 24, 25}` cho khoảng `1–24`, nhưng **không nêu tên kỹ thuật là 3-value BVA**. Trong khi đó, `Ôn tập.pdf` (trang 9–10) tách rõ hai mức **2-value BVA** (`{0, 1, 24, 25}`) và **3-value BVA** (`{0, 1, 2, 23, 24, 25}`).
- **Cách hiểu chuẩn ISTQB CTFL v4.0.1:**
  - **2-value BVA:** Mỗi ranh giới chọn **2 điểm** *(tại biên + 1 điểm kế bên thuộc phân vùng khác)* → khoảng `[1, 24]` cần **`4` giá trị**: `{0, 1, 24, 25}`. Đây là mức chuẩn mặc định khi làm Lab nhanh.
  - **3-value BVA:** Mỗi ranh giới chọn **3 điểm** *(dưới biên, tại biên, trên biên)* → khoảng `[1, 24]` cần **`6` giá trị**: `{0, 1, 2, 23, 24, 25}`. Dùng khi hệ thống rủi ro cao (tài chính, y tế, hàng không) hoặc đề thi ISTQB ghi rõ yêu cầu **3-value BVA**.
  - *Xem chi tiết công thức tính coverage & ví dụ TechStore tại [02-boundary-value-analysis.md](../02-boundary-value-analysis.md).*

---

## 2. Đính chính lỗi công thức toán học trong `Ôn tập.pdf` (trang 9)

Bản xuất PDF của Notion bị lặp chuỗi LaTeX và Unicode dính liền nhau:
- Bản PDF gốc in: `1≤credits≤241 \le \text{credits} \le 241≤credits≤24` và `như nhau.=`
- **Nội dung đúng:** $$1 \le \text{credits} \le 24$$ *(hai số nằm trong cùng một phân vùng tương đương được kỳ vọng xử lý theo cùng một logic như nhau).*

---

## 3. Phạm vi kỹ thuật White-box giữa Note Trường và Giai đoạn 1

- `Dynamic testing.pdf` (trang 4) liệt kê ngắn 3 mức **White-box**: `Statement Coverage`, `Branch / Decision Coverage` và `Path Coverage`.
- Đối với phỏng vấn SDET/Java và đề thi nâng cao, đừng dừng ở 3 khái niệm này — hãy học đủ thêm **Condition Coverage**, **MC/DC** và cách tính **Cyclomatic Complexity** tại [07-white-box-coverage.md](../07-white-box-coverage.md) và [08-cyclomatic-complexity.md](../08-cyclomatic-complexity.md).
