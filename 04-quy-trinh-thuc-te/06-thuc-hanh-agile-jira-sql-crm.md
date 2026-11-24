# Thực hành Giai đoạn 4: Quy trình Agile, Jira, SBTM & Database Testing trên Project CRM

> **Mục tiêu:** Áp dụng trực tiếp toàn bộ quy trình thực tế (Agile Testing, Three Amigos, SBTM, Jira JQL, RCA, SQL Audit & Migration Testing) vào project **CRM** — đồng thời phát huy vai trò **Scrum Master / QA** trong đội dự án.

---

## Bài 1: Tổ chức Three Amigos & Chốt Acceptance Criteria + DoD cho Story CRM

**User Story mẫu (`CRM-104`):** *"Là một nhân viên Sales, tôi muốn chuyển đổi (Convert) một Lead đủ điều kiện (`QUALIFIED`) thành Customer + Contact + Deal mới để theo dõi doanh thu trong Pipeline."*

### Bước 1: Điều phối buổi Three Amigos (15–20 phút trước khi code)
Ghi lại các câu hỏi & quyết định thống nhất giữa **PO + Dev + QA**:

| Góc nhìn | Câu hỏi đặt ra trong buổi Three Amigos | Quyết định chốt vào Acceptance Criteria |
| :--- | :--- | :--- |
| **PO (Business)** | Nếu email công ty của Lead đã tồn tại trong bảng `customers` thì sao? | Cho phép chọn **"Gắn vào Customer đã có"** hoặc cảnh báo trùng email nếu chọn tạo mới. |
| **Dev (Tech)** | Quá trình tạo 3 bản ghi (`Customer`, `Contact`, `Deal`) và khóa `Lead` có cần atomic không? | Bọc trong 1 `@Transactional` duy nhất; nếu bước tạo Deal lỗi thì rollback sạch. |
| **QA (Risk)** | Nếu 2 người bấm Convert cùng 1 Lead đồng thời, hoặc Sales A convert Lead của Sales B thì sao? | Chỉ `Owner` hoặc `Sales Manager` được phép Convert (`403`); dùng optimistic/atomic update `WHERE status = 'QUALIFIED'` trả `409 Conflict` ở lần bấm thứ 2! |

### Bước 2: Phân tầng kiểm thử (Chống *Cupcake Antipattern*)
- **Unit Test (Q1 — Dev):** Kiểm tra state machine `LeadStatus.convert()` và validate giá trị Deal `>= 0`.
- **Integration / DB Test (Q1/Q2 — Dev+QA):** Testcontainers xác minh rollback khi lỗi tạo Deal & concurrency 2 request convert cùng 1 Lead.
- **API Test (Q2 — QA):** REST Assured kiểm tra `403` (Sales khác Owner), `409` (Lead đã `CONVERTED`/`DISQUALIFIED`), JSON Schema.
- **Exploratory SBTM (Q3 — QA):** Khám phá trường hợp bấm đúp nút Convert, ký tự Unicode/dài 255, quay lại trình duyệt sau khi Convert.

---

## Bài 2: Thiết kế Mind Map & Chạy Phiên SBTM (90 phút) trên CRM Pipeline

1. **Vẽ Mind Map phạm vi (10 phút):** Mở rộng 4 nhánh chính của `Deal Pipeline`:
   - Tạo Deal (`Create` & gắn Contact/Customer)
   - Chuyển Stage (`PROSPECTING → QUALIFIED → PROPOSAL → NEGOTIATION → CLOSED_WON / CLOSED_LOST`)
   - Tính toán Doanh thu & Xác suất thắng (*Weighted Forecast = Amount × Probability%*)
   - Phân quyền & Thao tác đồng thời (`Owner` vs `Manager` vs `Other Sales`)
2. **Viết Charter:** *"Explore **CRM Deal Pipeline** with **tài khoản Sales khác nhau, Drag-and-Drop Kanban nhanh, và DevTools Network** to discover **lỗi tính sai Forecast, cho phép kéo Deal sang stage không hợp lệ, hoặc lộ Deal của đội khác**."*
3. **Chạy 90 phút & Chấm TBS:**
   - Tổng hợp thành Session Report chuẩn (xem mẫu chi tiết tại [08-deliverable-crm-exploratory-session-report.md](./08-deliverable-crm-exploratory-session-report.md)).

---

## Bài 3: Thực hành Jira Triage, Viết JQL & Phân tích RCA (5 Whys)

### A. Thực hành lọc & phân loại 5 ticket giả lập trên Jira

| Ticket | Mô tả tình huống | Quyết định Triage | Câu lệnh JQL đưa lên Dashboard |
| :--- | :--- | :--- | :--- |
| `CRM-201` | Sales A xem và sửa được Deal của Sales B qua API `PUT /deals/45` | **Blocker / P1 — Fix Now** | `priority in (Highest, High) AND status not in (Closed, Done)` |
| `CRM-202` | Đã fix xong trên Staging bởi Dev lúc 14h hôm nay | Kéo sang **`Ready for QA`** để retest | `status = "Ready for QA" ORDER BY updated ASC` |
| `CRM-203` | Nút "Export CSV" ở trang báo cáo năm lệch lề 3px trên Safari | **Low / P3 — Defer** sang Sprint sau | `priority = Low AND status = Deferred` |
| `CRM-204` | Khách hàng báo trên Production: Convert Lead tên có dấu `&` bị lỗi `500` | **Escaped Bug — P1 + Bắt buộc RCA!** | `labels = "escaped-prod" AND created >= startOfMonth()` |
| `CRM-205` | Yêu cầu thêm tính năng tự động gửi email chúc mừng khi `CLOSED_WON` | **Not a Bug → Convert to Story** | — |

### B. Viết RCA (5 Whys) cho `CRM-204` (Escaped Bug)
- **Why 1:** Tại sao Convert Lead tên công ty `"A & B Co."` bị lỗi? → Vì query tìm kiếm/tạo Customer không sanitize hoặc endpoint xuất XML/template báo cáo không escape ký tự `&`.
- **Why 2:** Tại sao không bắt được khi QA test? → Vì dữ liệu test mẫu chỉ dùng tên tiếng Anh đơn giản (`"TechStore Corp"`), không có ký tự đặc biệt `& < > "` và tiếng Việt.
- **Why 3:** Tại sao test data thiếu ký tự đặc biệt? → Vì team chưa có bộ **Test Data Heuristic (Naughty Strings)** dùng chung cho API và Exploratory Testing.
- **Action Item (vào Sprint Retrospective):** Bổ sung chuỗi `"A & B Co. — Công ty TNHH Thử Nghiệm <script>"` vào bộ seed chuẩn của Integration Test & Exploratory Heuristic Checklist!

---

## Bài 4: Thực hành Database Testing & Migration Audit cho CRM

Chạy kiểm toán trực tiếp trên CSDL PostgreSQL của **CRM** (hoặc trong Testcontainers) bằng 5 truy vấn sau:

```sql
-- 1. Kiểm tra Data Integrity: Tìm Deal có stage = 'CLOSED_WON' nhưng thiếu ngày đóng (closed_at) hoặc amount <= 0
SELECT id, title, stage, amount, closed_at
FROM deals
WHERE stage = 'CLOSED_WON'
  AND (closed_at IS NULL OR amount <= 0);

-- 2. Kiểm tra Orphan Records: Tìm Contact không gắn Customer hoặc gắn Customer đã bị xóa mềm (is_deleted = true)
SELECT ct.id AS contact_id, ct.email, ct.customer_id
FROM contacts ct
LEFT JOIN customers cu ON ct.customer_id = cu.id
WHERE cu.id IS NULL OR cu.is_deleted = TRUE;

-- 3. Kiểm tra Aggregation & Forecast: Đối chiếu tổng tiền Deal thắng theo từng Sales Owner trong tháng
SELECT u.full_name,
       COUNT(d.id) AS won_deals,
       COALESCE(SUM(d.amount), 0) AS total_won_revenue
FROM users u
LEFT JOIN deals d ON d.owner_id = u.id
    AND d.stage = 'CLOSED_WON'
    AND d.closed_at >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY u.id, u.full_name
ORDER BY total_won_revenue DESC;

-- 4. Kiểm tra trùng lặp Lead theo Email chuẩn hóa (Case-insensitive Duplicate)
SELECT LOWER(TRIM(email)) AS clean_email, COUNT(*) AS dup_count
FROM leads
GROUP BY LOWER(TRIM(email))
HAVING COUNT(*) > 1;

-- 5. Kiểm tra lịch sử Migration Flyway: Có script nào failed hoặc chưa chạy?
SELECT version, description, script, checksum, success, installed_on
FROM flyway_schema_history
WHERE success = FALSE
   OR installed_rank = (SELECT MAX(installed_rank) FROM flyway_schema_history);
```
