# Concepts Bổ sung — Giai đoạn 6: Tổng hợp Bẫy Thi ISTQB & Phỏng vấn từ Note Trường

Thư mục này đóng vai trò **bảng tra cứu nhanh (*Cheat-Sheet*)** trước khi thi **ISTQB CTFL v4.0.1** hoặc đi phỏng vấn từ các bài học chuyên sâu và các điểm dễ nhầm đã được giải thích trong `concepts/` của Giai đoạn 0, 1, 4 và 5.

## 8 Cặp Khái niệm "Hay Bị Hỏi Bẫy" Đã Chuẩn hóa từ Note Trên lớp

1. **Static Validation vs. Static Verification** → [00-nen-tang/concepts/01-static-validation.md](../../00-nen-tang/concepts/01-static-validation.md)  
   *Bẫy:* Nghĩ Static chỉ làm Verification. Thực tế cho khách bấm Wireframe / Figma Prototype là **Static Validation** (chống hội chứng *IKIWISI*).
2. **V&V vs. QA/QC** → [00-nen-tang/concepts/03-vv-standards-and-note-clarifications.md](../../00-nen-tang/concepts/03-vv-standards-and-note-clarifications.md)  
   *Bẫy:* Nhớ nhầm *"Verification = QA, Validation = QC"*. Chuẩn xác: Kiểm tra sản phẩm trực tiếp (cả V lẫn V) thuộc **QC**; chuẩn hóa quy trình/checklist thuộc **QA**.
3. **System Testing nằm ở Verification hay Validation?** → [00-nen-tang/concepts/02-v-model-and-vv.md](../../00-nen-tang/concepts/02-v-model-and-vv.md) & [07-system-testing.md](../../00-nen-tang/concepts/07-system-testing.md)  
   *Chuẩn:* System Testing là **Giao thoa V&V** (Verify kiến trúc/NFR + Validate hành trình E2E người dùng).
4. **4 cấp độ Review (Informal / Walkthrough / Technical Review / Formal Inspection)** → [04-informal-review.md](../../00-nen-tang/concepts/04-informal-review.md), [05-walkthrough.md](../../00-nen-tang/concepts/05-walkthrough.md), [06-technical-review.md](../../00-nen-tang/concepts/06-technical-review.md), [09-formal-inspection.md](../../00-nen-tang/concepts/09-formal-inspection.md) & [10-static-testing-reading-techniques-and-process.md](../../00-nen-tang/concepts/10-static-testing-reading-techniques-and-process.md)  
   *Điểm ăn tiền phỏng vấn:* **Technical Review** có chiều sâu công nghệ cao nhất và **khuyến khích tranh luận giải pháp**, còn **Formal Inspection (Fagan)** có tính hình thức quy trình cao nhất, có vai trò **Reader** diễn giải, đo tốc độ (`90–125 LOC/h`) và **cấm bàn giải pháp** ngay trong buổi soi lỗi; lưu ý **5 bước Review (ISTQB v4)** vs **6 bước (thêm Follow-up theo IEEE 1028)**.
5. **Unit vs. Integration Strategies & Stub vs. Driver** → [02-unit-integration-java/concepts/01-unit-and-integration-fundamentals.md](../../02-unit-integration-java/concepts/01-unit-and-integration-fundamentals.md)  
   *Phỏng vấn cổ điển:* **Top-Down** dùng **Stub** (thay tầng dưới chưa code); **Bottom-Up** dùng **Driver** (gọi tầng dưới khi tầng trên chưa code); **Big Bang** tích hợp một cục cực kỳ rủi ro khi khoanh vùng lỗi.
6. **2-Value vs. 3-Value BVA** → [01-test-design/concepts/01-school-notes-clarifications.md](../../01-test-design/concepts/01-school-notes-clarifications.md)  
   *Mẹo thi ISTQB:* Kiểm tra kỹ đề hỏi 2-value hay 3-value trước khi đếm mẫu số `% BVA Coverage`.
7. **Formal UAT vs. Alpha Testing vs. Beta Testing** → [04-quy-trinh-thuc-te/concepts/01-user-acceptance-testing-uat.md](../../04-quy-trinh-thuc-te/concepts/01-user-acceptance-testing-uat.md) & [02-alpha-and-beta-testing.md](../../04-quy-trinh-thuc-te/concepts/02-alpha-and-beta-testing.md)  
   *Điểm nhấn:* UAT dùng *Real Business Data*, QA tuyệt đối không test thay khách; Alpha lọc lỗi nội bộ; Beta thử thực địa ngoài môi trường không kiểm soát để phá *Curse of Knowledge*.
8. **OAT (RPO vs. RTO) & ISO/IEC 25010** → [05-non-functional-cicd/concepts/01-iso-25010-quality-characteristics.md](../../05-non-functional-cicd/concepts/01-iso-25010-quality-characteristics.md) & [02-operational-acceptance-testing-oat.md](../../05-non-functional-cicd/concepts/02-operational-acceptance-testing-oat.md)  
   *Phỏng vấn SRE/Senior QA:* Phân biệt **RPO** *(mất tối đa bao nhiêu phút dữ liệu)* và **RTO** *(mất tối đa bao lâu để hệ thống online lại)*.

> **Xem thêm bộ hướng dẫn chi tiết Giai đoạn 6:** [02-istqb-ctfl-v4-study-guide.md](../02-istqb-ctfl-v4-study-guide.md) và [04-interview-qa-masterbank.md](../04-interview-qa-masterbank.md).
