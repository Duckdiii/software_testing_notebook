# Giai đoạn 5: Non-functional & CI/CD (Tuần 12–14 · 4 tuần)

> **Bám syllabus:** Tuần 12–14  
> **Điểm khác biệt:** Bạn đã học Burp Suite, WebGoat và Juice Shop cho mảng Security.

## Checklist kiến thức & kỹ năng

- [ ] [**Performance:** JMeter hoặc k6; phân biệt load, stress, spike, soak test; đọc kết quả `p95`/`p99`, throughput, error rate](./01-performance-testing-jmeter-k6.md)
- [ ] [**Security testing:** OWASP Top 10 (2021 & 2025) theo góc nhìn tester, Burp Suite & tích hợp DAST (OWASP ZAP) vào pipeline](./02-security-testing-owasp-burp-zap.md)
- [ ] [**Accessibility & usability:** WCAG 2.2 (POUR), `axe-core`, Lighthouse CI & kiểm tra bàn phím](./03-accessibility-wcag-axe-lighthouse.md)
- [ ] [**CI/CD:** GitHub Actions & Jenkins chạy test tự động mỗi PR; Docker Compose cho môi trường test, quản lý flaky test, test report, quality gate](./04-cicd-github-actions-docker-flaky-gates.md)

### Concepts bổ sung (`./concepts/`)
- [Tổng mục lục Giai đoạn 5](./concepts/README.md)
- [ ] [Mô hình chất lượng ISO/IEC 25010 (Đủ 8 đặc tính & cập nhật 2023)](./concepts/01-iso-25010-quality-characteristics.md)
- [ ] [Operational Acceptance Testing (OAT): Backup/Restore (RPO), DR (RTO), Rollback & Alerting](./concepts/02-operational-acceptance-testing-oat.md)

## Thực hành

- [ ] [Thực hành: k6 Load/Spike, Burp Suite + OWASP ZAP Docker, Lighthouse/axe & Docker Compose cho TechStore](./05-thuc-hanh-k6-zap-lighthouse-cicd-techstore.md)

## Deliverable

- [ ] [Deliverable 1: Pipeline CI chạy `unit + API + UI + non-functional + Allure Report`](./06-deliverable-ci-pipeline-techstore.md)
- [ ] [Deliverable 2: Báo cáo Kiểm thử Hiệu năng (`PERF-TECHSTORE-2026-01`)](./07-deliverable-performance-test-report.md)
- [ ] [Deliverable 3: Báo cáo Kiểm thử Bảo mật (`SEC-TECHSTORE-2026-01`)](./08-deliverable-security-test-report.md)
