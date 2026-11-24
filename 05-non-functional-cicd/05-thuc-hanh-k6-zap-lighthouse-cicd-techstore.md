# Thực hành Giai đoạn 5: Performance (k6), Security (Burp/ZAP), Accessibility & CI/CD cho TechStore

> **Mục tiêu:** Trực tiếp cấu hình, thực thi và đọc kết quả các bài kiểm thử phi chức năng (Performance, Security, Accessibility) kết hợp dựng môi trường Docker Compose + CI Pipeline cho hệ thống **TechStore**.

---

## Bài 1: Thực hành Performance Testing với Grafana k6 (Load + Spike)

### Bước 1: Tạo script `perf/techstore-flashsale.js`
Kiểm thử đồng thời 2 kịch bản song song (*Multi-Scenario k6*):
1. `browse_catalog` (80% lưu lượng): Khách duyệt `GET /api/v1/products` liên tục (tải ổn định `100 VUs` trong 3 phút).
2. `flashsale_spike` (20% lưu lượng): Giả lập giờ G Flash Sale 00:00 — tăng vọt từ `0 → 200 VUs` trong 10 giây gọi `POST /api/v1/checkout` để săn mã `SALE20`!

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  scenarios: {
    browse_catalog: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '30s', target: 100 },
        { duration: '2m', target: 100 },
        { duration: '30s', target: 0 },
      ],
      exec: 'browseProducts',
    },
    flashsale_spike: {
      executor: 'ramping-vus',
      startTime: '1m',
      startVUs: 0,
      stages: [
        { duration: '10s', target: 200 }, // Sốc tải đột ngột!
        { duration: '40s', target: 200 },
        { duration: '10s', target: 0 },
      ],
      exec: 'checkoutOrder',
    },
  },
  thresholds: {
    'http_req_failed': ['rate<0.01'],
    'http_req_duration{scenario:browse_catalog}': ['p(95)<300', 'p(99)<600'],
    'http_req_duration{scenario:flashsale_spike}': ['p(95)<600', 'p(99)<1500'],
  },
};

export function setup() {
  const res = http.post(`${__ENV.BASE_URL}/api/v1/auth/login`, JSON.stringify({
    email: 'buyer@techstore.vn',
    password: __ENV.BUYER_PASSWORD || 'Buyer@123',
  }), { headers: { 'Content-Type': 'application/json' } });
  return { token: res.json('accessToken') };
}

export function browseProducts() {
  const res = http.get(`${__ENV.BASE_URL}/api/v1/products?page=0&size=20`);
  check(res, { 'products 200': (r) => r.status === 200 });
  sleep(1);
}

export function checkoutOrder(data) {
  const res = http.post(`${__ENV.BASE_URL}/api/v1/checkout`, JSON.stringify({
    userId: 101,
    vip: false,
    hasFreeshipCode: false,
    paymentMethod: 'VNPAY',
    items: [{ productId: 1, quantity: 1 }],
  }), {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${data.token}`,
    },
  });
  check(res, { 'checkout 201 or 409 out-of-stock': (r) => r.status === 201 || r.status === 409 });
  sleep(0.5);
}
```

### Bước 2: Chạy & Phân tích
```bash
# Chạy bằng Docker (không cần cài k6 lên máy Windows!) và xuất summary JSON
docker run --rm -i --network host -v "${PWD}/perf:/scripts" grafana/k6 run \
  -e BASE_URL=http://localhost:8080 \
  --summary-export=/scripts/k6-summary.json \
  /scripts/techstore-flashsale.js
```
**Quan sát cần rút ra:** Nếu `p99` vượt `1500ms` hoặc thấy lỗi `HikariPool-1 - Connection is not available, request timed out after 30000ms` trong log Spring Boot → phát hiện ngay **tắc nghẽn Connection Pool DB**!

---

## Bài 2: Thực hành Security Testing với Burp Suite & OWASP ZAP

### Phần A: Kiểm thử Logic & Access Control thủ công bằng Burp Suite
Thực hiện các phép thử sau qua **Burp Proxy + Repeater**:
1. **IDOR / BOLA (`A01:2025`):** Đăng nhập bằng Buyer A, bắt request `GET /api/v1/orders/105`, đưa sang **Repeater**, đổi `105` thành `106` (đơn hàng của Buyer B) → quan sát có bị trả `200 OK` lộ địa chỉ & SĐT của Buyer B không?
2. **Sửa giá & số lượng (`A06:2025`):** Bắt `POST /api/v1/checkout`, thử đổi `"quantity": -5` hoặc thêm `"unitPrice": 1` vào JSON body → kiểm tra server có lấy giá từ DB hay tin client.
3. **Race Condition Voucher (`A06:2025`):** Đưa `POST /api/v1/checkout` (dùng mã `SALE20` còn 1 lượt) vào **Burp Repeater**, nhân bản thành **10 tab**, nhóm lại và bấm **`Send group in parallel (single-packet attack)`** → kiểm tra có bị âm `usage_limit` không!
4. **JWT Tampering (`A07:2025`):** Dùng **JWT Editor** đổi `role` từ `CUSTOMER` sang `ADMIN` và đổi `alg: "none"`.

### Phần B: Chạy OWASP ZAP DAST Scan bằng Docker
```bash
# 1. ZAP Baseline Scan (Passive — an toàn, bắt thiếu Security Headers & Cookie flags)
docker run --rm --network host -v "${PWD}/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
  -t http://localhost:3000 \
  -r zap-baseline-report.html

# 2. ZAP API Scan (Active fuzzing theo file OpenAPI swagger.json)
docker run --rm --network host -v "${PWD}/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py \
  -t http://localhost:8080/v3/api-docs -f openapi \
  -r zap-api-report.html
```

---

## Bài 3: Thực hành Accessibility (`axe-core` + Lighthouse + Keyboard)

1. **Automated Scan với Lighthouse CLI:**
   ```bash
   npx @lhci/cli autorun --collect.url=http://localhost:3000/checkout
   ```
2. **Thử thách "Không dùng chuột" trên trang Checkout TechStore:**
   - Mở `http://localhost:3000/checkout`, **tuyệt đối không chạm chuột/touchpad**.
   - Dùng `Tab` di chuyển qua ô số lượng, ô nhập mã `SALE20`, nút `Áp dụng`, radio chọn `COD / VNPAY`, và nút `Đặt hàng`.
   - Ghi lại mọi lỗi gặp phải (VD: nút `Áp dụng` làm bằng `<div>` nên phím `Tab` nhảy qua không chọn được; hoặc viền focus bị CSS `outline: none` ẩn mất!).

---

## Bài 4: Dựng Môi trường Docker Compose & Kiểm thử Toàn Pipeline

1. Khởi chạy stack test sạch hoàn toàn:
   ```bash
   docker compose -f docker-compose.test.yml up -d --wait
   ```
2. Chạy lần lượt 3 tầng kiểm thử như trên CI:
   - Tầng 1: `mvn clean verify` (Backend Unit + Testcontainers + JaCoCo 80% Gate)
   - Tầng 2: `mvn test -f automation/pom.xml -Dheadless=true` (REST Assured + Selenium UI Smoke)
   - Tầng 3: `k6` Smoke (`10 VUs, 30s`) + `ZAP Baseline`
3. Dọn dẹp sạch môi trường sau khi test:
   ```bash
   docker compose -f docker-compose.test.yml down -v
   ```
