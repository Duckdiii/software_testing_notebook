# Thực hành Giai đoạn 2: TDD, Unit & Integration Test cho Module TechStore

> **Mục tiêu:** Chuyển hóa toàn bộ các Test Condition và Bug tiềm ẩn đã tìm ra ở **Giai đoạn 1** (`01-test-design/10-thuc-hanh-so-sanh-4-ky-thuat.md` & `11-deliverable-test-design-doc-rtm.md`) thành bộ **Unit Test + Slice Test + Testcontainers Integration Test** tự động bằng Java, JUnit 5, Mockito 5, AssertJ, JaCoCo và PITest.

---

## Bản đồ bài thực hành (4 phần liên hoàn)

| Bài tập | Công cụ / Kỹ năng | Mục tiêu cụ thể trên **TechStore** |
| :---: | :--- | :--- |
| **Bài 1** | **TDD (`Red → Green → Refactor`)** | Xây dựng `VoucherRuleEngine` xử lý `SALE20` & `FLAT50K`, trị tận gốc 2 bug từ GĐ 1 (thiếu check `startDate` & tính gộp Flash Sale). |
| **Bài 2** | **JUnit 5 + Mockito 5 + AssertJ** | Viết `CheckoutServiceTest` với `@Nested`, `@ParameterizedTest` (`@CsvSource`), `@Mock`, `@Captor`, `InOrder`, `SoftAssertions`. |
| **Bài 3** | **Spring Boot Test + Testcontainers** | Viết `@WebMvcTest` (`MockMvcTester` + `@MockitoBean`), `@DataJpaTest`, và `@SpringBootTest` + PostgreSQL container test **Concurrency / Race Condition** tranh `stock = 1` & `usageLimit = 1`. |
| **Bài 4** | **JaCoCo vs. PITest (Mutation Testing)** | Thực nghiệm so sánh "Test lấy số 100% coverage" vs. "Test có ý nghĩa diệt Mutant". |

---

## Bài 1: Thực hành TDD (5 vòng Red → Green → Refactor) cho `VoucherRuleEngine`

**Đặc tả nghiệp vụ (`FR-VCH-01`, `FR-VCH-02`):**
- Voucher có `active`, `[startDate, endDate]`, `minOrder`, `type` (`PERCENT` hoặc `FIXED`), `value`, `maxDiscount` (`> 0` nếu có trần).
- **Chỉ tính trên các sản phẩm KHÔNG phải Flash Sale (`isFlashSale == false`)**: cả ngưỡng `minOrder` lẫn số tiền giảm đều chỉ tính trên `eligibleSubtotal` (tổng tiền hàng thường).
- Số tiền giảm không bao giờ vượt quá `eligibleSubtotal`.

### Vòng 1 — Happy Path cho Voucher Phần trăm (`SALE20`)
1. **RED:** Tạo `VoucherRuleEngineTest.java` (lúc này chưa có class `VoucherRuleEngine` → compile fail = RED!).
   ```java
   class VoucherRuleEngineTest {
       private static final Instant NOW = Instant.parse("2026-11-25T10:00:00Z");
       private final VoucherRuleEngine engine = new VoucherRuleEngine(Clock.fixed(NOW, ZoneOffset.UTC));

       @Test
       void shouldApply20PercentWhenOrderMeetsMinSubtotal() {
           Voucher sale20 = Voucher.percent("SALE20", 20, 150_000, 500_000,
                   NOW.minus(1, ChronoUnit.DAYS), NOW.plus(5, ChronoUnit.DAYS));
           List<CartLine> lines = List.of(new CartLine("SP-01", 500_000, false));

           int discount = engine.calculateDiscount(sale20, lines);

           assertThat(discount).isEqualTo(100_000);
       }
   }
   ```
2. **GREEN:** Tạo `VoucherRuleEngine.java` viết tối giản nhất để pass:
   ```java
   public int calculateDiscount(Voucher v, List<CartLine> lines) {
       int subtotal = lines.stream().mapToInt(CartLine::lineTotal).sum();
       return (int) (subtotal * v.value() / 100.0);
   }
   ```

### Vòng 2 — Trị Bug GĐ 1 #1: Loại trừ sản phẩm Flash Sale (`FR-VCH-02`)
1. **RED:** Viết test giỏ hỗn hợp `[SP thường 300k + FlashSale 300k]` với `SALE20` (`minOrder = 500k`): kỳ vọng ném `MinOrderNotMetException` vì `eligibleSubtotal` chỉ có `300k < 500k`! Và giỏ `[SP thường 500k + FlashSale 300k]`: kỳ vọng chỉ giảm `20% * 500k = 100k` (không giảm trên 800k). Chạy → **FAIL**!
2. **GREEN:** Lọc `!line.flashSale()` khi tính `eligibleSubtotal` và kiểm tra `< v.minOrder()`.

### Vòng 3 — Trị Bug GĐ 1 #2: Kiểm tra đầy đủ `startDate` & `endDate`
1. **RED:** Thêm `@ParameterizedTest` kiểm tra cả 4 mốc biên thời gian (`startDate - 1s`, `startDate`, `endDate`, `endDate + 1s`):
   ```java
   @ParameterizedTest(name = "offsetFromStart={0}s, valid={1}")
   @CsvSource({
       "-1,    false", // Chưa mở -> phải từ chối! (Code GĐ 1 từng quên nhánh này!)
       "0,     true",  // Đúng giây bắt đầu -> hợp lệ
       "86400, true",  // Đúng giây kết thúc (sau 1 ngày) -> hợp lệ
       "86401, false"  // Hết hạn 1 giây -> từ chối
   })
   void shouldValidateVoucherTimeWindowBoundaries(long offsetSeconds, boolean expectedValid) {
       Instant start = NOW;
       Instant end = NOW.plusSeconds(86_400);
       Voucher v = Voucher.fixed("FLAT50K", 50_000, 200_000, start, end);
       VoucherRuleEngine customClockEngine = new VoucherRuleEngine(
               Clock.fixed(start.plusSeconds(offsetSeconds), ZoneOffset.UTC));
       List<CartLine> lines = List.of(new CartLine("SP-01", 300_000, false));

       if (expectedValid) {
           assertThat(customClockEngine.calculateDiscount(v, lines)).isEqualTo(50_000);
       } else {
           assertThatThrownBy(() -> customClockEngine.calculateDiscount(v, lines))
                   .isInstanceOf(InvalidVoucherException.class);
       }
   }
   ```
2. **GREEN:** Bổ sung `!now.isBefore(v.startDate())` và `!now.isAfter(v.endDate())`.

### Vòng 4 — Trần giảm tối đa (`maxDiscount`) & Voucher `FIXED` vượt tổng đơn
1. **RED:** Test đơn `1.000.000đ` dùng `SALE20` (`maxDiscount = 150.000đ`) → phải ra `150.000đ`. Test `FLAT50K` (`minOrder = 0`) trên đơn `30.000đ` → chỉ giảm tối đa `30.000đ` (không âm tiền!).
2. **GREEN:** Thêm `Math.min(discount, v.maxDiscount())` (khi `maxDiscount > 0`) và `Math.min(discount, eligibleSubtotal)`.

### Vòng 5 — REFACTOR
- Dùng `BigDecimal` với `RoundingMode.HALF_UP` (hoặc phép chia nguyên chính xác `subtotal * v.value() / 100`) thay cho `double` để tránh sai số dấu phẩy động kiểu `19999.9999999`.
- Tách hàm `validateActiveWindow(v, now)` và `computeEligibleSubtotal(lines)` cho rõ ràng, giữ toàn bộ test XANH 100%.

---

## Bài 2: Unit Test hoàn chỉnh với JUnit 5 (`@Nested` + `@ParameterizedTest`) + Mockito 5 + AssertJ

Áp dụng cho `CheckoutService.placeOrder(CheckoutCommand cmd)` — nơi điều phối `ProductRepository`, `VoucherRepository`, `VoucherRuleEngine`, `ShippingFeeCalculator`, `OrderRepository`.

### Các kịch bản bắt buộc tự viết & kiểm chứng
1. **`@ParameterizedTest` + `@CsvSource` cho Decision Table Freeship (`FR-CHK-02`) & Trần COD (`FR-CHK-03`):**
   - Kiểm tra lưu ý GĐ 1: Khách đặt hàng `490.000đ` (non-VIP, không có mã `FREESHIP`) → phí ship `30.000đ`, tổng thanh toán `520.000đ` (hệ thống **KHÔNG được lấy `520k` quay lại miễn ship!**).
   - Kiểm tra biên COD: `5.000.000đ` (cho phép) vs. `5.000.001đ` (chặn COD, ném `CodLimitExceededException`), trong khi `VNPAY` với `5.000.001đ` vẫn cho phép.
2. **`@Nested` cho Trường hợp Lỗi & Đảm bảo Side-Effect (`verify(..., never())`):**
   - Khi sản phẩm thứ 2 trong giỏ hết hàng (`quantity > stock`):
     - Assert ném `InsufficientStockException` có chứa SKU sản phẩm thiếu.
     - **`verify(orderRepository, never()).save(any())`** và **`verify(voucherRepository, never()).decrementUsageIfAvailable(any())`**.
3. **`@Captor` + `SoftAssertions` cho Happy Path:**
   - Bắt `Order` truyền vào `orderRepository.save(orderCaptor.capture())`.
   - Dùng `SoftAssertions.assertSoftly(...)` kiểm tra đồng thời `status == PENDING_PAYMENT`, `subtotal`, `discountAmount`, `shippingFee`, `finalTotal`, và `items` bằng `.extracting(...).containsExactlyInAnyOrder(...)`.

```java
@ExtendWith(MockitoExtension.class)
@DisplayName("CheckoutService — Unit Tests với Mockito 5 & AssertJ")
class CheckoutServiceTest {

    @Mock private ProductRepository productRepository;
    @Mock private VoucherRepository voucherRepository;
    @Mock private OrderRepository orderRepository;
    @Spy  private ShippingFeeCalculator shippingCalculator = new ShippingFeeCalculator();
    @Captor private ArgumentCaptor<Order> orderCaptor;

    private CheckoutService checkoutService;

    @BeforeEach
    void setUp() {
        VoucherRuleEngine ruleEngine = new VoucherRuleEngine(
                Clock.fixed(Instant.parse("2026-11-25T10:00:00Z"), ZoneOffset.UTC));
        checkoutService = new CheckoutService(
                productRepository, voucherRepository, orderRepository, ruleEngine, shippingCalculator);
    }

    @ParameterizedTest(name = "subtotal={0}, vip={1}, freeCode={2}, pay={3} => ship={4}, total={5}")
    @CsvSource({
        "490000, false, false, COD,   30000, 520000", // Bug GĐ 1 check: 490k + 30k = 520k không được freeship ngược!
        "500000, false, false, COD,       0, 500000", // Đúng biên 500k -> Freeship
        "150000, true,  false, COD,       0, 150000", // VIP -> Freeship
        "150000, false, true,  COD,       0, 150000", // Có mã FREESHIP -> Freeship
        "5000000,false, false, COD,       0, 5000000" // Trần tối đa COD
    })
    void shouldCalculateTotalAndAllowValidCodWithinLimit(
            int unitPrice, boolean isVip, boolean hasFreeship,
            PaymentMethod method, int expectedShip, int expectedTotal) {
        Product p = new Product(1L, "SKU-01", "Ban phim co", unitPrice, 10, false);
        given(productRepository.findAllById(List.of(1L))).willReturn(List.of(p));
        given(productRepository.decrementStockIfAvailable(1L, 1)).willReturn(1);
        given(orderRepository.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));

        CheckoutCommand cmd = new CheckoutCommand(100L, isVip, hasFreeship, null, method,
                List.of(new OrderItemRequest(1L, 1)));

        Order result = checkoutService.placeOrder(cmd);

        verify(orderRepository).save(orderCaptor.capture());
        Order saved = orderCaptor.getValue();
        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(result.getShippingFee()).isEqualTo(expectedShip);
            softly.assertThat(saved.getFinalTotal()).isEqualTo(expectedTotal);
            softly.assertThat(saved.getStatus()).isEqualTo(OrderStatus.PENDING_PAYMENT);
        });
    }
}
```

---

## Bài 3: Spring Boot Slice Tests & Testcontainers Concurrency Integration Test

### Bước 3A: `@WebMvcTest` với `MockMvcTester` & `@MockitoBean`
- Viết test kiểm tra:
  - `POST /api/v1/checkout` gửi `quantity = 0` hoặc `items = []` → `400 Bad Request` và `verifyNoInteractions(checkoutService)`.
  - `POST /api/v1/checkout` khi `checkoutService` ném `InsufficientStockException` → `409 Conflict` với JSON error message rõ ràng.
  - `POST /api/v1/checkout` hợp lệ → `201 Created` trả về `OrderResponse`.

### Bước 3B: `@DataJpaTest` với PostgreSQL Testcontainer
- Tắt thay thế H2 bằng `@AutoConfigureTestDatabase(replace = NONE)`.
- Kiểm tra câu query atomic:
  `UPDATE Product p SET p.stock = p.stock - :qty WHERE p.id = :id AND p.stock >= :qty`
  chạy qua `TestEntityManager` (`flush()` + `clear()`), đảm bảo `stock` không bao giờ âm (`stock = -1` như lỗi ở GĐ 1).

### Bước 3C: `@SpringBootTest(RANDOM_PORT)` + Testcontainers — Bắt Bug Race Condition Đồng Thời!
Đây là bài test khẳng định **đẳng cấp SDET** khi tái hiện trực tiếp `TC_VCH_006` & `BUG-CHK-01` từ GĐ 1 bằng đa luồng trên PostgreSQL thật:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(TestcontainersConfig.class) // Singleton PostgreSQLContainer + @ServiceConnection
class CheckoutConcurrencyIT {

    @Autowired private TestRestTemplate restTemplate;
    @Autowired private ProductRepository productRepository;
    @Autowired private VoucherRepository voucherRepository;
    @Autowired private OrderRepository orderRepository;

    private Long productId;

    @BeforeEach
    void seedCleanState() {
        orderRepository.deleteAllInBatch();
        voucherRepository.deleteAllInBatch();
        productRepository.deleteAllInBatch();

        // Sản phẩm chỉ còn ĐÚNG 1 cái cuối cùng trong kho!
        Product lastLaptop = productRepository.save(
                new Product("SKU-LAST-01", "Laptop Pro", 1_000_000, 1, false));
        productId = lastLaptop.getId();

        // Voucher chỉ còn ĐÚNG 1 lượt dùng!
        voucherRepository.save(Voucher.percent("SALE20", 20, 150_000, 500_000,
                Instant.now().minusSeconds(3600), Instant.now().plusSeconds(3600), 1));
    }

    @Test
    void shouldAllowOnlyOneOrderWhenTwoBuyersCheckoutLastItemConcurrently() throws Exception {
        int threads = 2;
        ExecutorService pool = Executors.newFixedThreadPool(threads);
        CountDownLatch readyLatch = new CountDownLatch(threads);
        CountDownLatch startGun = new CountDownLatch(1);
        List<Future<ResponseEntity<String>>> futures = new ArrayList<>();

        for (int i = 1; i <= threads; i++) {
            final long userId = i;
            futures.add(pool.submit(() -> {
                readyLatch.countDown();
                startGun.await(); // Ép cả 2 thread bắn request HTTP CÙNG MỘT THỜI ĐIỂM!
                CheckoutRequest req = new CheckoutRequest(
                        userId, false, false, "SALE20", PaymentMethod.VNPAY,
                        List.of(new OrderItemRequest(productId, 1)));
                return restTemplate.postForEntity("/api/v1/checkout", req, String.class);
            }));
        }

        readyLatch.await();
        startGun.countDown(); // Nổ súng xuất phát đồng thời!

        List<Integer> statusCodes = new ArrayList<>();
        for (Future<ResponseEntity<String>> f : futures) {
            statusCodes.add(f.get().getStatusCode().value());
        }
        pool.shutdown();

        // KỲ VỌNG: Đúng 1 request 201 Created, đúng 1 request 409 Conflict!
        assertThat(statusCodes).containsExactlyInAnyOrder(201, 409);

        // Kiểm chứng DB thật trong Postgres: stock = 0 (KHÔNG BAO GIỜ ÂM), chỉ có 1 Order!
        Product after = productRepository.findById(productId).orElseThrow();
        assertThat(after.getStock()).isZero();
        assertThat(orderRepository.count()).isEqualTo(1);
    }
}
```
> **Lưu ý kiến thức cực quan trọng từ bài này:** Nếu `CheckoutService.placeOrder()` trừ kho sai bằng kiểu `read-modify-write` trong Java (`if (p.getStock() >= qty) { p.setStock(p.getStock() - qty); save(p); }`) thay vì atomic SQL hoặc Pessimistic Lock, test tích hợp đa luồng với Testcontainers trên sẽ **bắt chết ngay** lỗi tạo 2 đơn và `stock = -1`! (Unit Test với Mockito KHÔNG bao giờ bắt được lỗi concurrency này!).

---

## Bài 4: Thực nghiệm JaCoCo vs. PITest (Chứng minh "Coverage cao ≠ Test tốt")

### Bước thực hiện
1. Chạy `mvn clean verify` → xem báo cáo JaCoCo tại `target/site/jacoco/index.html`.
2. Tạm thời thay thế các `assertThat(...)` trong `VoucherRuleEngineTest` bằng `assertThat(discount).isNotNegative()` (kiểu test lấy số), giữ nguyên 2 ca chạy để vẫn đạt **100% Line & Branch Coverage** trên JaCoCo.
3. Chạy `mvn test-compile pitest:mutationCoverage`:
   - **Kết quả với bộ Test lấy số:** JaCoCo báo `100%`, nhưng PITest báo hàng loạt mutant **`SURVIVED`** (đổi `>=` thành `>`, đổi `*` thành `/`, bỏ qua `Math.min(discount, maxDiscount)` mà test vẫn xanh!) → Mutation Score tụt xuống `< 30%`.
4. Khôi phục bộ test chuẩn (có BVA + AssertJ đầy đủ + `verify(..., never())`) và chạy lại `mvn test-compile pitest:mutationCoverage`:
   - **Kết quả mong đợi:** JaCoCo Branch Coverage `>= 85%` VÀ PITest Mutation Score `>= 85%`!

---

## Checklist tự kiểm tra sau khi hoàn thành Bài thực hành

- [ ] Toàn bộ Unit Test (`*Test.java`) chạy dưới `2 giây` và **không khởi động Spring hay Docker**.
- [ ] Mọi Integration Test (`*IT.java`) dùng **chung 1 Singleton PostgreSQL Container** qua `@ServiceConnection`.
- [ ] Không có `@DirtiesContext` hay `Thread.sleep(...)` trong bất kỳ file test nào.
- [ ] Bắt được lỗi biên thời gian `startDate`, lỗi `FlashSale`, lỗi `490k + 30k` ship, và lỗi concurrency `stock = 1` đã liệt kê từ GĐ 1.
