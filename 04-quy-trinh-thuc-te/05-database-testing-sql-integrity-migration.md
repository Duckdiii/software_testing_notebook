# Database Testing: SQL Nâng cao, Data Integrity, Transaction/Rollback & Migration Testing

> **Nguồn tham khảo:** PostgreSQL / ANSI SQL Standards, Flyway & Liquibase Documentation, *Full Stack Testing* (Chương 2 — Data & Integration Testing), *Agile Testing* (Chương 20 — Data Conversion & Database Updates).

Nhiều lỗi nghiêm trọng không hề hiển thị ngay trên giao diện UI: đơn hàng báo "Thành công" trên màn hình nhưng trong Database lại lưu sai tổng tiền, thiếu dòng chi tiết, tạo bản ghi mồ côi (*orphan record*), hoặc bị nhân đôi sau khi chạy script migration. **Database Testing** giúp QA xác minh trực tiếp **tầng dữ liệu (System of Record)** — nơi không bao giờ nói dối!

---

## 1. SQL Nâng cao cho QA (`JOIN`, `GROUP BY + HAVING`, CTE, Window Functions)

### A. Tìm lỗi bằng `LEFT JOIN ... WHERE ... IS NULL` (Kiểm tra bản ghi thiếu / mồ côi)
```sql
-- 1. Tìm Deal trong CRM không liên kết với Customer hợp lệ nào (Orphan Record!)
SELECT d.id AS deal_id, d.title, d.customer_id
FROM deals d
LEFT JOIN customers c ON d.customer_id = c.id
WHERE c.id IS NULL;

-- 2. Tìm Lead đã chuyển sang CONVERTED nhưng lại KHÔNG sinh ra Deal tương ứng (Lỗi nửa vời!)
SELECT l.id AS lead_id, l.email, l.converted_at
FROM leads l
LEFT JOIN deals d ON d.source_lead_id = l.id
WHERE l.status = 'CONVERTED'
  AND d.id IS NULL;
```

### B. Đối chiếu tổng hợp với `GROUP BY` & `HAVING` (Kiểm tra tính nhất quán số liệu)
```sql
-- 3. Tìm các đơn hàng có tổng tiền lưu ở bảng `orders` KHÔNG KHỚP với tổng `order_items`!
SELECT o.id AS order_id,
       o.subtotal AS stored_subtotal,
       COALESCE(SUM(oi.unit_price * oi.quantity), 0) AS computed_subtotal
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.subtotal
HAVING o.subtotal <> COALESCE(SUM(oi.unit_price * oi.quantity), 0);
```

### C. Tìm bản ghi trùng lặp (Race Condition / Double-Submit) bằng CTE & `ROW_NUMBER()`
```sql
-- 4. Tìm các Customer bị tạo trùng email (không phân biệt hoa/thường!) do bấm đúp hoặc thiếu UNIQUE index
WITH ranked_customers AS (
    SELECT id,
           LOWER(TRIM(email)) AS normalized_email,
           created_at,
           ROW_NUMBER() OVER (PARTITION BY LOWER(TRIM(email)) ORDER BY created_at ASC) AS rn
    FROM customers
)
SELECT id, normalized_email, created_at
FROM ranked_customers
WHERE rn > 1;
```

---

## 2. Kiểm tra Data Integrity (Toàn vẹn Dữ liệu) trên 4 Trục

| Trục kiểm tra | Mục đích & Lỗi hay gặp | Câu lệnh SQL kiểm chứng mẫu |
| :--- | :--- | :--- |
| **1. Entity Integrity** *(Khóa chính & Định danh)* | Bản ghi có `id` duy nhất, không bị trùng lặp do thiếu `UNIQUE` constraint (VD: `email`, `voucher.code`). | `SELECT LOWER(code), COUNT(*) FROM vouchers GROUP BY LOWER(code) HAVING COUNT(*) > 1;` |
| **2. Referential Integrity** *(Khóa ngoại)* | Xóa `Customer` nhưng quên `ON DELETE RESTRICT` hoặc xóa tay làm `deals`/`orders` trỏ vào khoảng không (**Orphan Records**). | `LEFT JOIN ... WHERE parent.id IS NULL` |
| **3. Domain Integrity** *(Kiểu, miền giá trị, NOT NULL)* | Tồn kho âm (`stock < 0`), giá trị Deal âm (`amount < 0`), trạng thái lạ ngoài Enum (`status NOT IN (...)`), chuỗi rỗng `''` lọt vào cột `email`. | `SELECT * FROM products WHERE stock < 0;` <br> `SELECT * FROM deals WHERE stage NOT IN ('PROSPECTING','PROPOSAL','WON','LOST');` |
| **4. Business Rule Integrity** *(Ràng buộc chéo bảng)* | Ngày kết thúc nhỏ hơn ngày bắt đầu (`end_date < start_date`), tổng số lượt đã dùng vượt `usage_limit`. | `SELECT * FROM vouchers WHERE end_date < start_date;` |

---

## 3. Kiểm tra ACID Transaction, Rollback & Isolation

Một giao dịch nghiệp vụ phức tạp (như **Convert Lead → Customer + Contact + Deal** trên CRM hoặc **Place Order** trên TechStore) ghi vào 3–4 bảng liên tiếp. QA phải kiểm tra **Atomicity (Tính nguyên tử)**:

### Kịch bản kiểm thử Rollback thực tế
1. **Cách test tầng Integration (Java/Testcontainers — xem GĐ 2):** Giả lập bước 3 (tạo Deal hoặc trừ kho sản phẩm thứ 2) bị ném Exception hoặc vi phạm Constraint → kiểm tra bảng `customers`, `contacts`, `leads` ở bước 1 & 2 **phải quay về trạng thái ban đầu** (không được có Customer "rác" được tạo mà không có Deal!).
2. **Cách test nhanh trực tiếp trên DB console (`BEGIN ... ROLLBACK`):**
   ```sql
   BEGIN;
   UPDATE leads SET status = 'CONVERTED' WHERE id = 10;
   -- Giả lập câu INSERT bước sau fail do vi phạm NOT NULL / FK
   INSERT INTO deals (title, customer_id) VALUES ('Big Deal', 999999);
   -- Sau khi thấy lỗi FK -> kiểm tra lại trong cùng phiên hoặc ROLLBACK:
   ROLLBACK;
   SELECT id, status FROM leads WHERE id = 10; -- Phải vẫn giữ trạng thái cũ!
   ```
3. **Bẫy hay gặp trong Spring Boot:**
   - Hàm ghi nhiều bảng nhưng **quên `@Transactional`**.
   - Hàm có `@Transactional` nhưng ném **Checked Exception** (như `IOException`) mà quên `rollbackFor = Exception.class` → mặc định Spring chỉ rollback với `RuntimeException`!
   - Dùng `try { ... } catch (Exception e) { log.error(...); }` nuốt mất Exception nên Spring tưởng thành công và **Commit một nửa dữ liệu**!

---

## 4. Database Migration Testing (Flyway & Liquibase)

Trong dự án thực tế (như *Agile Testing* Chương 20 tr. 459 đã cảnh báo về rủi ro *Data Conversion and Database Updates* trước khi Go-Live), thay đổi cấu trúc bảng được quản lý bằng **Flyway** (`V1__init.sql`, `V2__add_lead_status.sql`) hoặc **Liquibase** (`changelog.xml`/`yaml`).

### 5 Kịch bản QA Bắt buộc Kiểm tra với Migration
1. **Clean-Slate Migration (Chạy từ đầu trên DB rỗng):** Chạy tuần tự toàn bộ `V1 → Vn` trên database trắng (Testcontainers) → kiểm tra mọi bảng, index, FK, constraint khớp thiết kế và **không có lỗi cú pháp**.
2. **Incremental Upgrade trên Dữ liệu Giống Production (Quan trọng nhất!):**
   - Nạp bản sao dữ liệu hiện có (đã ẩn danh PII) ở trạng thái `V1`, sau đó nâng cấp lên `V2`!
   - *Bug kinh điển:* Dev thêm cột `NOT NULL` mới (VD: `phone_verified BOOLEAN NOT NULL`) mà **không có giá trị `DEFAULT` hoặc script backfill dữ liệu cũ** → Chạy trên DB rỗng thì XANH 100%, nhưng deploy lên Production có 100.000 dòng cũ thì **NỔ SẬP DEPLOY!**
3. **Data Preservation & Transformation:** Đổi tên/tách cột (VD: tách `full_name` thành `first_name` + `last_name`) → kiểm tra có bị mất dữ liệu, cắt xén ký tự tiếng Việt có dấu, hoặc sai `NULL` không.
4. **Checksum & Lịch sử Migration (`flyway_schema_history` / `DATABASECHANGELOG`):**
   ```sql
   -- Kiểm tra mọi migration đều success = true
   SELECT installed_rank, version, description, script, checksum, installed_on, success
   FROM flyway_schema_history
   ORDER BY installed_rank DESC;
   ```
   - Xác minh Dev **không sửa thẳng vào file `V1__...sql` đã áp dụng** (sẽ gây lỗi lệch `checksum` ở môi trường Staging/Production!). Phải luôn tạo file version mới `V3__...sql`.
5. **Backward Compatibility & Rollback Plan:** Trong hệ thống Zero-Downtime (Blue-Green / Rolling Deploy), code cũ `v1.0` có thể vẫn đang chạy song song 1–2 phút trong lúc DB vừa migrate lên `V2` → tuyệt đối không `DROP COLUMN` ngay trong cùng 1 đợt release mà phải tách làm 2 giai đoạn (*Expand and Contract pattern*)!
