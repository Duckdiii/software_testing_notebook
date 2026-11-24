# Mockito 5 & AssertJ: Test Doubles, Verify & Fluent Assertions

> **Nguồn tham khảo:** Mockito 5 Official Documentation, AssertJ Core Documentation, *Unit Testing Principles, Practices, and Patterns* (Vladimir Khorikov), Martin Fowler (*Mocks Aren't Stubs*), `Unit_Testing__Integration_Testing...pdf`.

Nếu JUnit 5 là "khung sườn chạy test", thì **Mockito** và **AssertJ** là bộ đôi vũ khí biến Unit Test thành tài sản bảo vệ mã nguồn: **Mockito** cô lập hoàn toàn class đang kiểm thử (*System Under Test — SUT*) khỏi database/API bên ngoài, còn **AssertJ** giúp câu lệnh kiểm chứng (*Assertion*) đọc trôi chảy như tiếng Anh tự nhiên và thông báo lỗi cực kỳ rõ ràng khi test fail.

---

## 1. Phân biệt 5 loại Test Doubles (Câu hỏi phỏng vấn cực hay gặp)

Theo Martin Fowler & Gerard Meszaros (*xUnit Test Patterns*), từ "Mock" thường bị dùng chung chung, nhưng thực tế có **5 loại Test Double** khác nhau hoàn toàn về bản chất:

| Loại | Bản chất & Mục đích | Ví dụ thực tế trên **TechStore** |
| :--- | :--- | :--- |
| **1. Dummy** | Object truyền vào chỉ để **lấp đầy tham số** cho hàm biên dịch được; không bao giờ được dùng đến giá trị bên trong. | Truyền `new AuditContext()` rỗng hoặc `null` khi test một nhánh không ghi log audit. |
| **2. Stub** | Object giả **trả về dữ liệu cứng định sẵn** (*canned answers*) khi được gọi, giúp SUT đi qua nhánh logic mong muốn. *(Kiểm thử trạng thái — State verification)*. | `when(productRepo.findById(1L)).thenReturn(Optional.of(laptop));` |
| **3. Fake** | Phiên bản cài đặt **hoạt động thật nhưng đơn giản hóa** (không dùng cho production). | `InMemoryOrderRepository` dùng `ConcurrentHashMap` thay cho JPA/PostgreSQL, hoặc H2 trong bộ nhớ. |
| **4. Spy** | Bọc quanh **object thật**: mặc định **chạy code thật** của các phương thức, trừ những phương thức bạn chủ đích `stub` lại, đồng thời **ghi lại lịch sử lời gọi** để `verify`. | `OrderPricingService` thật nhưng `doReturn(0).when(spy).fetchExternalTax(...)` — **hãy dùng rất hạn chế** vì thường là dấu hiệu thiết kế class đang ôm quá nhiều trách nhiệm! |
| **5. Mock** | Object giả được cấu hình kỳ vọng hành vi **từ trước** và dùng để **xác minh tương tác (`verify`)**: hàm X có được gọi đúng 1 lần với đúng tham số đó không? *(Kiểm thử hành vi — Behavior verification)*. | `verify(emailService, times(1)).sendOrderConfirmation(eq("buyer@techstore.vn"), any());` |

> **Tổng kết ngắn gọn (như trong tài liệu `Unit_Testing__Integration_Testing...pdf`):**
> - **Stub** = giả lập **dữ liệu trả về** (`when(...).thenReturn(...)`).
> - **Mock** = giả lập & **kiểm tra hành vi gọi hàm** (`verify(...)`).

---

## 2. Mockito 5: Kiến trúc & Cách khởi tạo chuẩn trong JUnit 5

### Điểm mới quan trọng của Mockito 5
- Yêu cầu tối thiểu **Java 11+**.
- **`mockito-inline` được bật làm mặc định**: Bạn có thể mock trực tiếp **`final` class, `final` method, `enum`, và phương thức `static`** mà không cần thêm dependency `mockito-inline` riêng như thời Mockito 3/4.
- Tích hợp với JUnit 5 **luôn dùng `@ExtendWith(MockitoExtension.class)`** (tự khởi tạo `@Mock`, `@Spy`, `@Captor`, `@InjectMocks`, tự chạy Strict Stubbing và tự dọn dẹp sau mỗi test).

```java
@ExtendWith(MockitoExtension.class)
class CheckoutServiceTest {

    @Mock
    private ProductRepository productRepository; // Mock 100%

    @Mock
    private VoucherService voucherService;

    @Mock
    private OrderRepository orderRepository;

    @Spy
    private ShippingFeeCalculator shippingCalculator = new ShippingFeeCalculator(); // Object thật được theo dõi

    @Captor
    private ArgumentCaptor<Order> orderCaptor; // Bắt object truyền vào mock để kiểm tra sâu

    @InjectMocks
    private CheckoutService checkoutService; // Mockito tự inject các @Mock/@Spy qua Constructor!
}
```

> **Cảnh báo quan trọng với `@InjectMocks`:**
> - `@InjectMocks` ưu tiên **Constructor Injection**. Nếu class có 2 dependency cùng interface hoặc dùng field injection kết hợp tham số nguyên thủy (`@Value("${app.taxRate}") double taxRate`), `@InjectMocks` sẽ không tự truyền được hoặc inject nhầm `0.0`.
> - **Best Practice:** Nếu Constructor phức tạp, hãy tự `new CheckoutService(...)` trong `@BeforeEach` cho tường minh và dễ kiểm soát!

---

## 3. Stubbing: Cổ điển vs. BDDMockito, Non-void vs. `void` / `@Spy`

### A. Hai phong cách Stubbing tương đương nhau
1. **Phong cách chuẩn (`Mockito`):**
   ```java
   when(productRepo.findById(10L)).thenReturn(Optional.of(keyboard));
   when(productRepo.findById(99L)).thenThrow(new EntityNotFoundException("Not found"));
   ```
2. **Phong cách BDD (`BDDMockito`) — khớp cấu trúc Given / When / Then:**
   ```java
   // Given
   given(productRepo.findById(10L)).willReturn(Optional.of(keyboard));
   // When
   Order result = checkoutService.placeOrder(request);
   // Then
   then(orderRepo).should(times(1)).save(any(Order.class));
   ```

### B. `thenReturn` vs. `thenAnswer` (Stub động theo tham số đầu vào)
```java
// Trả lại chính object Order được truyền vào save() — mô phỏng hành vi JPA gán ID tự tăng
when(orderRepository.save(any(Order.class))).thenAnswer(invocation -> {
    Order saved = invocation.getArgument(0);
    saved.setId(1001L);
    return saved;
});

// Chuỗi trả về qua nhiều lần gọi liên tiếp (ví dụ: lần 1 lỗi mạng, lần 2 retry thành công!)
when(paymentGateway.charge(any()))
    .thenThrow(new GatewayTimeoutException("Timeout"))
    .thenReturn(PaymentReceipt.success("TXN-999"));
```

### C. Bẫy chết người với phương thức `void` và `@Spy` (`doThrow` / `doReturn`)
- Bạn **không thể** viết `when(emailService.sendEmail(...)).thenThrow(...)` vì hàm `void` không có kiểu trả về để nhét vào `when(...)`!
  - **Cách đúng cho `void`:**
    ```java
    doThrow(new MailSendException("SMTP down"))
        .when(emailService).sendOrderConfirmation(anyString(), any());
    // Hoặc explicit doNothing() (mặc định mock void đã không làm gì, chỉ cần khi ghi đè stub hoặc @Spy)
    doNothing().when(auditLogger).log(any());
    ```
- **Với `@Spy`:** Nếu bạn viết `when(spyList.get(0)).thenReturn("x")`, Java sẽ gọi **hàm `get(0)` thật TRƯỚC** khi đưa vào `when(...)` → nổ ngay `IndexOutOfBoundsException`!
  - **Cách đúng cho `@Spy`:** Luôn dùng họ **`doReturn(...).when(spy)...`**:
    ```java
    doReturn("Laptop Gaming").when(spyList).get(0); // KHÔNG gọi method thật khi stubbing!
    ```

### D. Quy tắc vàng về `ArgumentMatchers`
- Nếu một lời gọi stub hoặc verify có **ít nhất 1 matcher** (`any()`, `eq()`, `argThat()`), thì **TẤT CẢ tham số còn lại đều phải dùng matcher**!
  ```java
  // SAI -> Ném InvalidUseOfMatchersException!
  when(voucherRepo.findValidCode("SALE20", any())).thenReturn(Optional.of(v));

  // ĐÚNG -> Bọc "SALE20" bằng eq(...)
  when(voucherRepo.findValidCode(eq("SALE20"), any(Instant.class))).thenReturn(Optional.of(v));
  ```

---

## 4. Strict Stubbing (`STRICT_STUBS`) — Tránh "Test Rác" & Stub Thừa

Từ Mockito 3+ (và mặc định khi dùng `@ExtendWith(MockitoExtension.class)` trong Mockito 5), Mockito bật chế độ **`Strictness.STRICT_STUBS`**:
1. **Phát hiện stub dư thừa (`UnnecessaryStubbingException`):** Nếu bạn viết `when(repo.findById(1L))...` trong `@BeforeEach` hoặc trong `@Test` nhưng luồng chạy thực tế **không bao giờ gọi tới nó**, test sẽ **FAIL**! Điều này giữ cho test sạch, không chứa giả định sai lệch.
2. **Phát hiện gọi sai tham số (`PotentialStubbingProblem`):** Nếu bạn stub `findById(1L)` nhưng code lại gọi `findById(2L)`, Mockito báo lỗi rõ ràng ngay tại điểm gọi thay vì âm thầm trả về `null` rồi gây `NullPointerException` khó hiểu ở 10 dòng sau.
3. **Cách xử lý hợp lệ khi 1 stub dùng chung ở `@BeforeEach` chỉ áp dụng cho vài test:**
   ```java
   @BeforeEach
   void setUp() {
       // Chỉ đánh dấu lenient() cho đúng stub tùy chọn đó — tuyệt đối không tắt strict cho cả class!
       lenient().when(featureFlagService.isVoucherEnabled()).thenReturn(true);
   }
   ```

---

## 5. Verification, `ArgumentCaptor` & `MockedStatic`

### A. Kiểm tra hành vi (`verify`) chính xác
```java
// 1. Kiểm tra số lần gọi
verify(orderRepository, times(1)).save(any(Order.class)); // times(1) là mặc định
verify(emailService, never()).sendOrderConfirmation(anyString(), any());
verify(paymentService, atLeastOnce()).refund(anyLong());

// 2. Kiểm tra thứ tự gọi giữa nhiều dependency (InOrder)
InOrder inOrder = inOrder(productRepository, orderRepository, emailService);
inOrder.verify(productRepository).saveAll(anyList());   // Phải trừ kho TRƯỚC
inOrder.verify(orderRepository).save(any(Order.class)); // Rồi mới lưu đơn
inOrder.verify(emailService).sendOrderConfirmation(anyString(), any()); // Sau cùng mới gửi mail

// 3. Cẩn trọng với verifyNoMoreInteractions()!
// Chỉ dùng khi thực sự cần khóa chặt giao tiếp của 1 mock nhạy cảm (VD: cổng trừ tiền),
// vì lạm dụng sẽ khiến test cực kỳ giòn (brittle) mỗi khi refactor thêm hàm log/metric.
verifyNoInteractions(paymentGateway); // Tốt cho các case lỗi đầu vào: chắc chắn KHÔNG đụng tới cổng thanh toán!
```

### B. `ArgumentCaptor` — Soi tận bên trong object tạo mới bởi SUT
Khi hàm `checkoutService.placeOrder(req)` tự tạo `new Order(...)` bên trong rồi gọi `orderRepository.save(order)`, bạn không cầm sẵn tham chiếu `order` đó ở bên ngoài. Dùng `@Captor` để bắt lấy object mà SUT vừa truyền vào `save()`:

```java
@Captor
private ArgumentCaptor<Order> orderCaptor;

@Test
void shouldSaveOrderWithCalculatedDiscountAndPendingStatus() {
    // ... Given & When ...
    checkoutService.placeOrder(request);

    // Then: bắt chính xác object Order đã đưa xuống repository
    verify(orderRepository).save(orderCaptor.capture());
    Order capturedOrder = orderCaptor.getValue();

    assertThat(capturedOrder.getStatus()).isEqualTo(OrderStatus.PENDING_PAYMENT);
    assertThat(capturedOrder.getDiscountAmount()).isEqualTo(100_000);
    assertThat(capturedOrder.getFinalAmount()).isEqualTo(400_000);
}
```
> **Lưu ý:** Nếu giá trị truyền vào chỉ là kiểu đơn giản hoặc Value Object có `equals()` chuẩn, hãy ưu tiên `verify(repo).findByCode(eq("SALE20"))` thay vì lạm dụng `ArgumentCaptor`.

### C. `MockedStatic` — Mock hàm `static` an toàn tuyệt đối
Khi code legacy gọi `LocalDateTime.now()` hoặc `UUID.randomUUID()` trực tiếp *(dù thiết kế tốt nhất là inject `Clock`!)*, Mockito 5 cho phép mock static **bắt buộc phải đặt trong `try-with-resources`**:

```java
@Test
void shouldRejectExpiredVoucherUsingFixedTime() {
    Instant fakeNow = Instant.parse("2026-12-05T00:00:00Z");

    try (MockedStatic<Instant> mockedInstant = mockStatic(Instant.class)) {
        mockedInstant.when(Instant::now).thenReturn(fakeNow);

        assertThatThrownBy(() -> voucherService.validate(expiredVoucher))
                .isInstanceOf(VoucherExpiredException.class);
    } // Rời khỏi khối try -> Mockito tự giải phóng static mock, KHÔNG bao giờ rò rỉ sang test sau!
}
```

---

## 6. AssertJ: Fluent Assertions giúp Test sắc bén & dễ đọc

Import chuẩn: `import static org.assertj.core.api.Assertions.*;` (và `assertThatThrownBy`, `tuple`).

### Tại sao AssertJ vượt trội hoàn toàn so với JUnit `Assertions`?
1. **Đúng ngữ pháp tự nhiên & không bao giờ ngược `expected`/`actual`:** `assertThat(actual).isEqualTo(expected)`.
2. **IDE auto-complete theo đúng kiểu dữ liệu:** `assertThat(list)` chỉ gợi ý hàm cho `List`, `assertThat(str)` gợi ý hàm chuỗi, `assertThat(map)` gợi ý `containsEntry`.
3. **Thông báo lỗi khi Fail cực chi tiết:** Hiển thị rõ phần tử nào thiếu, phần tử nào dư trong Collection hoặc field nào lệch trong Object.

### Các chiêu AssertJ dùng hàng ngày trong Java/Spring Boot

```java
// 1. String & Number assertions
assertThat(order.getCode())
    .isNotBlank()
    .startsWith("ORD-2026-")
    .hasSize(15);

assertThat(discountAmount)
    .isPositive()
    .isBetween(1_000, 150_000);

// 2. Collection assertions với extracting() & tuple() — KHÔNG cần viết vòng lặp for!
assertThat(order.getItems())
    .hasSize(2)
    .extracting(OrderItem::getProductSku, OrderItem::getQuantity, OrderItem::isFlashSale)
    .containsExactlyInAnyOrder(
        tuple("SKU-LAPTOP-01", 1, false),
        tuple("SKU-MOUSE-02", 2, true)
    );

// 3. Exception testing — kiểm tra cả kiểu Exception, message, và cause
assertThatThrownBy(() -> checkoutService.placeOrder(invalidRequest))
    .isInstanceOf(InsufficientStockException.class)
    .hasMessageContaining("SKU-LAPTOP-01")
    .hasNoCause();

// Khẳng định KHÔNG ném ngoại lệ (thay thế try-catch trống)
assertThatCode(() -> voucherService.validate(validVoucher))
    .doesNotThrowAnyException();

// 4. SoftAssertions — kiểm tra mọi trường của OrderDTO và báo TẤT CẢ sai lệch cùng lúc
SoftAssertions.assertSoftly(softly -> {
    softly.assertThat(response.status()).isEqualTo(OrderStatus.PENDING_PAYMENT);
    softly.assertThat(response.shippingFee()).isZero();
    softly.assertThat(response.finalTotal()).isEqualTo(400_000);
});

// 5. So sánh Object lồng nhau mà không phụ thuộc equals() (bỏ qua ID/createdAt tự sinh!)
assertThat(savedOrder)
    .usingRecursiveComparison()
    .ignoringFields("id", "createdAt", "updatedAt")
    .isEqualTo(expectedOrder);
```

---

## 7. Anti-patterns cần tránh khi dùng Mockito & AssertJ

| Anti-pattern | Hậu quả | Cách làm đúng |
| :--- | :--- | :--- |
| **Mock chính class đang test (`SUT`)** hoặc mock Value Object (`OrderItem`, `Money`, `DTO`). | Bạn không test code thật mà đang test mock trả về thứ bạn vừa cài đặt! | Chỉ mock **Collaborator / Dependency** bên ngoài biên (Repository, External API, Clock, Gateway). |
| **Mock quá nhiều lớp nội bộ (Over-mocking)** — test 1 hàm mà phải setup 6 `@Mock` và 15 `when(...)`. | Dấu hiệu class đang vi phạm **Single Responsibility Principle (SRP)**; mỗi lần refactor code là test vỡ hàng loạt. | Tách Domain Logic thuần (không phụ thuộc DB) ra khỏi Orchestration Service, hoặc dùng object thật cho collaborator đơn giản. |
| **Viết `any()` tràn lan trong `verify(...)`** khi cần verify tham số quan trọng. | Nếu code truyền nhầm `productId` với `orderId` (cùng kiểu `Long`), test vẫn pass! | Dùng `eq(expectedId)` hoặc `ArgumentCaptor` cho dữ liệu nghiệp vụ quan trọng. |
| **Lạm dụng `assertThat(result).isNotNull()` là xong** | Đạt coverage 100% nhưng hàm trả về sai giá trị vẫn pass! | Assert giá trị cụ thể, trạng thái, danh sách con và cả negative behavior (`verify(..., never())`). |
