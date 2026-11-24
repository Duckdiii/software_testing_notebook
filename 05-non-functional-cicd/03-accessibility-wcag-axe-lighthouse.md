# Accessibility & Usability Testing: WCAG 2.2, `axe-core` & Lighthouse

> **Nguồn tham khảo:** W3C WCAG 2.2 Guidelines, *Full Stack Testing* (Chương 7 — *Accessibility Testing*, tr. 288–325), Deque `axe-core` Docs, Google Lighthouse CI Docs.

**Accessibility Testing (Kiểm thử Khả năng Tiếp cận — viết tắt `a11y`)** đảm bảo sản phẩm có thể sử dụng được bởi tất cả mọi người, bao gồm người khuyết tật thị giác (dùng Screen Reader), khiếm thính, suy giảm vận động (chỉ dùng bàn phím, không dùng chuột) hoặc người cao tuổi. Tại châu Âu, Mỹ và các doanh nghiệp lớn, đạt chuẩn **WCAG Level AA** là yêu cầu bắt buộc về pháp lý lẫn chất lượng sản phẩm.

---

## 1. Bốn nguyên tắc P.O.U.R. của chuẩn WCAG 2.2 (Mức A & AA)

| Nguyên tắc | Câu hỏi kiểm tra cốt lõi | Ví dụ quy tắc cụ thể trên Web E-Commerce / CRM |
| :--- | :--- | :--- |
| **P — Perceivable** *(Nhận thức được)* | Nội dung có hiện hữu với các giác quan khác nhau (mắt, tai/trình đọc màn hình) không? | - Mọi ảnh sản phẩm phải có `alt` mô tả có nghĩa (`alt=""` cho ảnh trang trí).<br>- Độ tương phản màu (*Color Contrast*): chữ thường tối thiểu **`4.5:1`**, chữ lớn **`3:1`**.<br>- Không chỉ dùng màu đỏ/xanh để báo lỗi (phải kèm icon hoặc chữ!). |
| **O — Operable** *(Thao tác được)* | Người dùng **chỉ có bàn phím** có đi hết luồng mua hàng được không? | - Toàn bộ nút, link, dropdown, modal phải đi được bằng `Tab` / `Shift+Tab` / `Enter` / `Space` / `Esc`.<br>- Phải nhìn thấy rõ viền **Focus Indicator** khi bấm `Tab` (tránh CSS `outline: none` mà không thay thế!).<br>- **WCAG 2.2 mới:** Kích thước vùng bấm tối thiểu `24×24px` (*Target Size 2.5.8*) và phần tử đang focus không bị sticky header che khuất (*Focus Not Obscured 2.4.11*). |
| **U — Understandable** *(Dễ hiểu)* | Giao diện & thông báo lỗi có rõ ràng, nhất quán không? | - Input form phải có `<label for="...">` gắn đúng; thông báo lỗi phải chỉ rõ **sai ở ô nào và cách sửa** (không chỉ hiện viền đỏ trống trơn!).<br>- Khai báo đúng `<html lang="vi">` để trình đọc màn hình phát âm chuẩn. |
| **R — Robust** *(Bền vững)* | Mã HTML & thuộc tính `ARIA` có chuẩn để công nghệ hỗ trợ (NVDA, VoiceOver) đọc đúng không? | - Dùng thẻ HTML ngữ nghĩa (`<button>`, `<nav>`, `<main>`, `<h1>→<h2>`) thay vì lấy `<div>` gắn `onClick`.<br>- Tránh dùng `role` hoặc `aria-*` sai chuẩn. |

---

## 2. Giới hạn của Tool Tự động & Quy trình Kết hợp

> **Sự thật quan trọng:** Các công cụ tự động như **axe-core** hay **Lighthouse** chỉ phát hiện được khoảng **30% – 40%** các tiêu chí WCAG (ví dụ: thiếu `alt`, sai tỷ lệ tương phản màu, thiếu `<label>`, sai thứ tự heading). **60% còn lại** (như: `alt` có mô tả đúng ý nghĩa không, thứ tự `Tab` có hợp lý không, Screen Reader đọc có dễ hiểu không) **bắt buộc phải kiểm tra thủ công!**

---

## 3. Tự động hóa với `axe-core` (Playwright & Selenium Java)

**`axe-core`** (của Deque) là engine chuẩn công nghiệp đứng sau cả Chrome Extension axe DevTools lẫn Google Lighthouse Accessibility Audit.

### A. Tích hợp `axe-core` vào Selenium + Java (`pom.xml`)
```xml
<dependency>
    <groupId>com.deque.html.axe-core</groupId>
    <artifactId>selenium</artifactId>
    <version>4.9.0</version>
    <scope>test</scope>
</dependency>
```
```java
@Test
public void checkoutPageShouldPassWcag22AA() {
    driver.get("http://localhost:3000/checkout");

    AxeBuilder axeBuilder = new AxeBuilder()
            .withTags(List.of("wcag2a", "wcag2aa", "wcag21aa", "wcag22aa"));
    Results results = axeBuilder.analyze(driver);

    List<Rule> violations = results.getViolations();
    violations.forEach(v -> System.out.printf("[%s] %s (%d nodes)%n",
            v.getImpact(), v.getDescription(), v.getNodes().size()));

    Assert.assertTrue(violations.isEmpty(),
            "Phát hiện " + violations.size() + " lỗi Accessibility nghiêm trọng!");
}
```

### B. Tích hợp `@axe-core/playwright` (TypeScript — Siêu gọn)
```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('Checkout page has no WCAG 2.2 AA violations', async ({ page }) => {
  await page.goto('/checkout');
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

---

## 4. Google Lighthouse & Lighthouse CI (`@lhci/cli`)

- **Chạy nhanh thủ công:** Mở Chrome DevTools (`F12`) → tab **Lighthouse** → chọn **Accessibility** (+ Performance, Best Practices, SEO) → Analyze page load.
- **Chạy tự động trên CI (`lighthouserc.json`)** chặn PR nếu điểm Accessibility tụt dưới `90/100`:
  ```json
  {
    "ci": {
      "collect": {
        "url": ["http://localhost:3000/", "http://localhost:3000/checkout"],
        "numberOfRuns": 3
      },
      "assert": {
        "assertions": {
          "categories:accessibility": ["error", { "minScore": 0.90 }],
          "categories:performance": ["warn", { "minScore": 0.80 }]
        }
      },
      "upload": { "target": "temporary-public-storage" }
    }
  }
  ```

---

## 5. Checklist Kiểm thử Thủ công Bắt buộc (*Full Stack Testing* Chương 7, tr. 304)

1. **Thử thách "Không dùng chuột" (Keyboard-Only Test):** Rút chuột ra, thử tìm 1 sản phẩm, thêm vào giỏ, áp voucher `SALE20` và đặt hàng **chỉ dùng `Tab`, `Shift+Tab`, `Enter`, `Space`, `Esc`, phím mũi tên**. Kiểm tra focus có bị kẹt (*Keyboard Trap*) trong modal popup không?
2. **Kiểm tra Chrome Accessibility Tree:** DevTools → Elements → tab **Accessibility** → xem cấu trúc cây Accessibility và Computed Name của các nút icon (VD nút giỏ hàng chỉ có icon SVG mà thiếu `aria-label="Giỏ hàng"` thì Screen Reader sẽ đọc là `"button"` vô nghĩa!).
3. **Zoom 200% & Grayscale:** Phóng to trình duyệt lên `200%` xem chữ/nút có bị chồng chéo, mất nút Đặt hàng hay không; bật bộ lọc xám (*Grayscale*) xem người mù màu có phân biệt được trạng thái lỗi/thành công không.
4. **Đo bằng Screen Reader thật:** Thử đi qua form Checkout bằng **NVDA** (miễn phí trên Windows) hoặc **VoiceOver** (`Cmd + F5` trên macOS).
