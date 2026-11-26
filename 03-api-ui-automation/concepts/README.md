# Concepts Bổ sung — Giai đoạn 3: API + UI Automation

## Tình trạng tài liệu bổ sung cho Giai đoạn 3

Bộ 15 file ghi chú bài học trên lớp mới gửi thêm thuộc phạm vi Tuần 1–2 (nền tảng V&V, Reviews, Test Levels), chưa đi sâu vào công cụ API/UI Automation (Postman, REST Assured, Selenium, Playwright, Cucumber).

- **Các điểm giao cắt đã được lọc trùng lặp:**
  - Phần **Manual Testing vs. Automated Testing** và **4 tầng Full-Stack Testing (`UI -> API/Service -> Data/DB -> Component/Unit`)** trong `Ôn tập.pdf` (trang 8 và 12) chỉ là phần dẫn nhập lý thuyết, đã được triển khai thực chiến kèm code Java/CI tại:
    - [03-api-ui-automation/README.md](../README.md) *(API REST Assured, Contract Pact, Selenium POM, Playwright)*
    - [02-unit-integration-java/07-test-pyramid-automation-strategy.md](../../02-unit-integration-java/07-test-pyramid-automation-strategy.md) *(Test Pyramid & Automated Strategy)*
- **Nguyên tắc cần giữ vững từ bài học trường:** Tự động hóa là đòn bẩy cho Regression Testing, Load Testing và CI/CD, nhưng **không thay thế tư duy phản biện** của con người và **không nên dồn hết logic nghiệp vụ lên UI E2E** (vì chạy chậm, giòn/flaky và màn hình báo *"Accepted"* vẫn có thể sai dưới API/DB).
