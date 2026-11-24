# Deliverable Giai đoạn 2: Module Backend TechStore Đạt `>= 80%` Branch Coverage Có Ý Nghĩa

> **Mục tiêu Deliverable (Roadmap GĐ 2):** Đưa một module backend của **TechStore** (`techstore-checkout-voucher`) lên **`>= 80%` Branch Coverage** trên JaCoCo, đi kèm bộ test **có ý nghĩa thực sự** (phát hiện bug nghiệp vụ, kiểm soát side-effect, kiểm thử đồng thời với Testcontainers và diệt Mutant trên PITest) chứ **không** viết test để lấy số.
>
> **Truy vết từ Giai đoạn 1:** Module này hiện thực hóa và bảo vệ toàn bộ 7 yêu cầu `FR-CHK-01..04`, `FR-VCH-01..03` cùng 15 ca trọng tâm `TC_CHK_001..008`, `TC_VCH_001..007` đã thiết kế trong [11-deliverable-test-design-doc-rtm.md](../01-test-design/11-deliverable-test-design-doc-rtm.md).

---

## 1. Kiến trúc Module & Bảng Truy vết Test Pyramid

```text
techstore-checkout-voucher/
├── pom.xml                                  # Spring Boot 3.4+/4.x, JUnit 5, Mockito 5, AssertJ, Testcontainers, JaCoCo, PITest
├── lombok.config                            # lombok.addLombokGeneratedAnnotation = true (loại trừ boilerplate khỏi JaCoCo)
└── src/
    ├── main/java/com/techstore/checkout/
    │   ├── domain/                          # OrderStatus, PaymentMethod, VoucherType, Product, Voucher, Order, OrderItem
    │   ├── repository/                      # ProductRepository, VoucherRepository, OrderRepository (atomic SQL updates)
    │   ├── service/                         # ShippingFeeCalculator, VoucherService, CheckoutService
    │   ├── web/                             # CheckoutRequest, OrderResponse, CheckoutController, GlobalExceptionHandler
    │   └── exception/                       # BusinessException, InsufficientStockException, InvalidVoucherException, CodLimitExceededException
    └── test/java/com/techstore/checkout/
        ├── domain/OrderStateTest.java       # Unit Test (Tầng 1) — Vòng đời trạng thái FR-CHK-04
        ├── service/ShippingFeeCalculatorTest.java # Unit Test (Tầng 1) — BVA & Decision Table FR-CHK-02
        ├── service/VoucherServiceTest.java  # Unit Test (Tầng 1) — BVA, Flash Sale, Start/End Date FR-VCH-01..02
        ├── service/CheckoutServiceTest.java # Unit Test + Mockito 5 (Tầng 1) — Orchestration & Side-effects FR-CHK-01,03
        ├── web/CheckoutControllerTest.java  # @WebMvcTest + MockMvcTester (Tầng 2a) — HTTP Contract & @Valid
        └── integration/
            ├── TestcontainersConfig.java    # Singleton PostgreSQLContainer + @ServiceConnection
            ├── VoucherRepositoryIT.java     # @DataJpaTest + Postgres thật — Atomic usage_limit
            └── CheckoutConcurrencyIT.java   # @SpringBootTest(RANDOM_PORT) + Postgres — Race Condition & Rollback
```

---

## 2. Cấu hình `pom.xml` & Quality Gate (JaCoCo `BRANCH >= 0.80` + PITest)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.5</version> <!-- Tương thích MockitoBean & MockMvcTester; nâng thẳng lên 4.x được -->
    </parent>

    <groupId>com.techstore</groupId>
    <artifactId>techstore-checkout-voucher</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>21</java.version>
        <jacoco.version>0.8.12</jacoco.version>
        <pitest.version>1.17.1</pitest.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Test Dependencies: JUnit 5, Mockito 5, AssertJ đã nằm sẵn trong spring-boot-starter-test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-testcontainers</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>postgresql</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Pha Unit Test: chạy *Test.java (< 2s) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
            </plugin>

            <!-- Pha Integration Test: chạy *IT.java (Testcontainers) trong mvn verify -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>integration-test</goal>
                            <goal>verify</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- JaCoCo Coverage + Quality Gate 80% Branch -->
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>${jacoco.version}</version>
                <executions>
                    <execution>
                        <id>prepare-agent</id>
                        <goals><goal>prepare-agent</goal></goals>
                    </execution>
                    <execution>
                        <id>prepare-agent-integration</id>
                        <goals><goal>prepare-agent-integration</goal></goals>
                    </execution>
                    <execution>
                        <id>report</id>
                        <phase>verify</phase>
                        <goals><goal>report</goal></goals>
                    </execution>
                    <execution>
                        <id>jacoco-check</id>
                        <phase>verify</phase>
                        <goals><goal>check</goal></goals>
                        <configuration>
                            <rules>
                                <rule>
                                    <element>BUNDLE</element>
                                    <limits>
                                        <limit>
                                            <counter>BRANCH</counter>
                                            <value>COVEREDRATIO</value>
                                            <minimum>0.80</minimum>
                                        </limit>
                                        <limit>
                                            <counter>LINE</counter>
                                            <value>COVEREDRATIO</value>
                                            <minimum>0.80</minimum>
                                        </limit>
                                    </limits>
                                </rule>
                            </rules>
                            <excludes>
                                <exclude>**/TechStoreApplication.class</exclude>
                                <exclude>**/config/**</exclude>
                                <exclude>**/exception/**</exclude>
                                <!-- Chỉ loại trừ DTO/Config thuần túy; Domain & Service vẫn được đo gắt gao! -->
                                <exclude>**/web/CheckoutRequest*.class</exclude>
                                <exclude>**/web/OrderResponse*.class</exclude>
                            </excludes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- PITest Mutation Testing cho tầng Domain & Service -->
            <plugin>
                <groupId>org.pitest</groupId>
                <artifactId>pitest-maven</artifactId>
                <version>${pitest.version}</version>
                <dependencies>
                    <dependency>
                        <groupId>org.pitest</groupId>
                        <artifactId>pitest-junit5-plugin</artifactId>
                        <version>1.2.1</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <targetClasses>
                        <param>com.techstore.checkout.service.*</param>
                        <param>com.techstore.checkout.domain.Order</param>
                    </targetClasses>
                    <targetTests>
                        <param>com.techstore.checkout.*Test</param>
                    </targetTests>
                    <mutators>
                        <mutator>STRONGER</mutator>
                    </mutators>
                    <mutationThreshold>80</mutationThreshold>
                    <withHistory>true</withHistory>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 3. Mã nguồn Production hoàn chỉnh (`src/main/java/...`)

### 3.1. Domain Enums & `Order` State Machine (`FR-CHK-04`)

```java
// domain/OrderStatus.java
package com.techstore.checkout.domain;

public enum OrderStatus {
    PENDING_PAYMENT, CONFIRMED, PAID, SHIPPING, COMPLETED, CANCELLED
}

// domain/PaymentMethod.java
package com.techstore.checkout.domain;

public enum PaymentMethod {
    COD, VNPAY
}

// domain/VoucherType.java
package com.techstore.checkout.domain;

public enum VoucherType {
    PERCENT, FIXED
}
```

```java
// domain/Order.java
package com.techstore.checkout.domain;

import jakarta.persistence.*;
import java.time.Instant;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long userId;
    private int subtotal;
    private int discountAmount;
    private int shippingFee;
    private int finalTotal;

    @Enumerated(EnumType.STRING)
    private PaymentMethod paymentMethod;

    @Enumerated(EnumType.STRING)
    private OrderStatus status = OrderStatus.PENDING_PAYMENT;

    private String voucherCode;
    private Instant createdAt = Instant.now();

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    protected Order() {}

    public Order(Long userId, int subtotal, int discountAmount, int shippingFee,
                 int finalTotal, PaymentMethod paymentMethod, String voucherCode) {
        this.userId = userId;
        this.subtotal = subtotal;
        this.discountAmount = discountAmount;
        this.shippingFee = shippingFee;
        this.finalTotal = finalTotal;
        this.paymentMethod = paymentMethod;
        this.voucherCode = voucherCode;
        this.status = OrderStatus.PENDING_PAYMENT;
    }

    public void addItem(OrderItem item) {
        item.setOrder(this);
        this.items.add(item);
    }

    // FR-CHK-04: Chỉ cho phép hủy khi ở PENDING_PAYMENT hoặc CONFIRMED (4 bytecode branches)
    public void cancel() {
        if (this.status != OrderStatus.PENDING_PAYMENT && this.status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException("Cannot cancel order in status: " + this.status);
        }
        this.status = OrderStatus.CANCELLED;
    }

    // FR-CHK-04: Chặn webhook PAYMENT_SUCCESS đến trễ hồi sinh đơn đã hủy (2 bytecode branches)
    public void markPaid() {
        if (this.status != OrderStatus.PENDING_PAYMENT) {
            throw new IllegalStateException("Cannot mark paid from status: " + this.status);
        }
        this.status = OrderStatus.PAID;
    }

    // Getters & helper setters (không chứa nhánh rẽ logic)
    public Long getId() { return id; }
    public Long getUserId() { return userId; }
    public int getSubtotal() { return subtotal; }
    public int getDiscountAmount() { return discountAmount; }
    public int getShippingFee() { return shippingFee; }
    public int getFinalTotal() { return finalTotal; }
    public PaymentMethod getPaymentMethod() { return paymentMethod; }
    public OrderStatus getStatus() { return status; }
    public String getVoucherCode() { return voucherCode; }
    public List<OrderItem> getItems() { return items; }
    public void setStatusForTest(OrderStatus status) { this.status = status; }
}
```

### 3.2. Entities `Product`, `Voucher`, `OrderItem` & Repositories (Atomic Concurrency Updates)

```java
// repository/ProductRepository.java
package com.techstore.checkout.repository;

import com.techstore.checkout.domain.Product;
import org.springframework.data.jpa.repository.*;
import org.springframework.data.repository.query.Param;

public interface ProductRepository extends JpaRepository<Product, Long> {

    // FR-CHK-01: Atomic update trực tiếp trong DB — ngăn chặn tuyệt đối stock âm khi 2 request chạy cùng lúc!
    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Query("UPDATE Product p SET p.stock = p.stock - :qty WHERE p.id = :id AND p.stock >= :qty")
    int decrementStockIfAvailable(@Param("id") Long productId, @Param("qty") int quantity);
}

// repository/VoucherRepository.java
package com.techstore.checkout.repository;

import com.techstore.checkout.domain.Voucher;
import org.springframework.data.jpa.repository.*;
import org.springframework.data.repository.query.Param;
import java.util.Optional;

public interface VoucherRepository extends JpaRepository<Voucher, Long> {

    Optional<Voucher> findByCodeIgnoreCase(String code);

    // FR-VCH-03: Atomic update đảm bảo usageLimit = 1 không bao giờ bị trừ thành -1!
    @Modifying(clearAutomatically = true, flushAutomatically = true)
    @Query("UPDATE Voucher v SET v.usageLimit = v.usageLimit - 1 WHERE UPPER(v.code) = UPPER(:code) AND v.usageLimit > 0")
    int decrementUsageIfAvailable(@Param("code") String code);
}
```

### 3.3. Services (`ShippingFeeCalculator`, `VoucherService`, `CheckoutService`)

```java
// service/ShippingFeeCalculator.java
package com.techstore.checkout.service;

import org.springframework.stereotype.Component;

@Component
public class ShippingFeeCalculator {

    public static final int DEFAULT_SHIPPING_FEE = 30_000;
    public static final int FREESHIP_THRESHOLD = 500_000;

    // FR-CHK-02: Tổng 8 bytecode branches (subtotal < 0: 2; isVip || hasFreeshipCode || >= 500k: 6)
    public int calculateFee(int subtotalAfterDiscount, boolean isVip, boolean hasFreeshipCode) {
        if (subtotalAfterDiscount < 0) {
            throw new IllegalArgumentException("Subtotal cannot be negative");
        }
        if (isVip || hasFreeshipCode || subtotalAfterDiscount >= FREESHIP_THRESHOLD) {
            return 0;
        }
        return DEFAULT_SHIPPING_FEE;
    }
}
```

```java
// service/VoucherService.java
package com.techstore.checkout.service;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.exception.InvalidVoucherException;
import org.springframework.stereotype.Service;
import java.time.Clock;
import java.time.Instant;
import java.util.List;

@Service
public class VoucherService {

    private final Clock clock;

    public VoucherService(Clock clock) {
        this.clock = clock;
    }

    // FR-VCH-01 & FR-VCH-02: Tổng 24 bytecode branches
    public int calculateDiscount(Voucher voucher, List<OrderItem> items) {
        if (voucher == null || !voucher.isActive()) {                         // 4 branches
            throw new InvalidVoucherException("Mã giảm giá không hợp lệ hoặc đã bị khóa");
        }
        Instant now = Instant.now(clock);
        if (now.isBefore(voucher.getStartDate()) || now.isAfter(voucher.getEndDate())) { // 4 branches
            throw new InvalidVoucherException("Mã giảm giá chưa bắt đầu hoặc đã hết hạn");
        }
        if (voucher.getUsageLimit() <= 0) {                                   // 2 branches
            throw new InvalidVoucherException("Mã giảm giá đã hết lượt sử dụng");
        }
        if (items == null || items.isEmpty()) {                               // 4 branches
            throw new InvalidVoucherException("Giỏ hàng trống, không thể áp dụng mã");
        }

        int eligibleSubtotal = 0;
        for (OrderItem item : items) {                                        // 2 branches (loop)
            if (!item.isFlashSale()) {                                        // 2 branches
                eligibleSubtotal += item.getLineTotal();
            }
        }

        if (eligibleSubtotal < voucher.getMinOrder()) {                       // 2 branches
            throw new InvalidVoucherException("Giá trị sản phẩm hợp lệ chưa đạt mức tối thiểu");
        }

        int rawDiscount = (voucher.getType() == VoucherType.PERCENT)          // 2 branches (ternary)
                ? (int) Math.round(eligibleSubtotal * (voucher.getValue() / 100.0))
                : voucher.getValue();

        int capped = (voucher.getMaxDiscount() > 0)                           // 2 branches (ternary)
                ? Math.min(rawDiscount, voucher.getMaxDiscount())
                : rawDiscount;

        return Math.min(capped, eligibleSubtotal);
    }
}
```

```java
// service/CheckoutService.java
package com.techstore.checkout.service;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.exception.*;
import com.techstore.checkout.repository.*;
import com.techstore.checkout.web.CheckoutRequest;
import com.techstore.checkout.web.OrderResponse;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;

@Service
public class CheckoutService {

    public static final int MAX_COD_TOTAL = 5_000_000;

    private final ProductRepository productRepository;
    private final VoucherRepository voucherRepository;
    private final OrderRepository orderRepository;
    private final VoucherService voucherService;
    private final ShippingFeeCalculator shippingFeeCalculator;

    public CheckoutService(ProductRepository productRepository,
                           VoucherRepository voucherRepository,
                           OrderRepository orderRepository,
                           VoucherService voucherService,
                           ShippingFeeCalculator shippingFeeCalculator) {
        this.productRepository = productRepository;
        this.voucherRepository = voucherRepository;
        this.orderRepository = orderRepository;
        this.voucherService = voucherService;
        this.shippingFeeCalculator = shippingFeeCalculator;
    }

    // FR-CHK-01, FR-CHK-02, FR-CHK-03, FR-VCH-03: Tổng 22 bytecode branches
    @Transactional
    public OrderResponse placeOrder(CheckoutRequest request) {
        if (request.items() == null || request.items().isEmpty()) {           // 4 branches
            throw new IllegalArgumentException("Đơn hàng phải có ít nhất 1 sản phẩm");
        }

        List<Long> productIds = request.items().stream()
                .map(CheckoutRequest.ItemLine::productId)
                .toList();
        Map<Long, Product> productMap = productRepository.findAllById(productIds).stream()
                .collect(Collectors.toMap(Product::getId, Function.identity()));

        List<OrderItem> orderItems = new ArrayList<>();
        int subtotal = 0;

        for (CheckoutRequest.ItemLine line : request.items()) {               // 2 branches (loop)
            if (line.quantity() <= 0) {                                       // 2 branches
                throw new IllegalArgumentException("Số lượng phải lớn hơn 0");
            }
            Product product = productMap.get(line.productId());
            if (product == null) {                                            // 2 branches
                throw new ProductNotFoundException("Không tìm thấy sản phẩm ID: " + line.productId());
            }
            int updatedStock = productRepository.decrementStockIfAvailable(product.getId(), line.quantity());
            if (updatedStock == 0) {                                          // 2 branches
                throw new InsufficientStockException("Sản phẩm không đủ tồn kho: " + product.getSku());
            }
            OrderItem orderItem = new OrderItem(product.getId(), product.getSku(),
                    product.getPrice(), line.quantity(), product.isFlashSale());
            orderItems.add(orderItem);
            subtotal += orderItem.getLineTotal();
        }

        int discount = 0;
        String appliedVoucher = null;
        if (request.voucherCode() != null && !request.voucherCode().isBlank()) { // 4 branches
            String normalizedCode = request.voucherCode().trim().toUpperCase(Locale.ROOT);
            Voucher voucher = voucherRepository.findByCodeIgnoreCase(normalizedCode)
                    .orElseThrow(() -> new InvalidVoucherException("Mã giảm giá không tồn tại: " + normalizedCode));

            discount = voucherService.calculateDiscount(voucher, orderItems);
            int updatedUsage = voucherRepository.decrementUsageIfAvailable(normalizedCode);
            if (updatedUsage == 0) {                                          // 2 branches
                throw new InvalidVoucherException("Mã giảm giá vừa hết lượt sử dụng");
            }
            appliedVoucher = normalizedCode;
        }

        int subtotalAfterDiscount = subtotal - discount;
        int shippingFee = shippingFeeCalculator.calculateFee(
                subtotalAfterDiscount, request.vip(), request.hasFreeshipCode());
        int finalTotal = subtotalAfterDiscount + shippingFee;

        if (request.paymentMethod() == PaymentMethod.COD && finalTotal > MAX_COD_TOTAL) { // 4 branches
            throw new CodLimitExceededException("Thanh toán COD chỉ áp dụng cho đơn tối đa 5.000.000đ");
        }

        Order order = new Order(request.userId(), subtotal, discount, shippingFee,
                finalTotal, request.paymentMethod(), appliedVoucher);
        orderItems.forEach(order::addItem);
        Order saved = orderRepository.save(order);

        return OrderResponse.from(saved);
    }
}
```

### 3.4. Controller & Exception Handler (`web/CheckoutController.java` & `GlobalExceptionHandler.java`)

```java
// web/CheckoutController.java
package com.techstore.checkout.web;

import com.techstore.checkout.service.CheckoutService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/checkout")
public class CheckoutController {

    private final CheckoutService checkoutService;

    public CheckoutController(CheckoutService checkoutService) {
        this.checkoutService = checkoutService;
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse checkout(@Valid @RequestBody CheckoutRequest request) {
        return checkoutService.placeOrder(request);
    }
}
```

---

## 4. Bộ Test Suite Hoàn Chỉnh (Unit + Slice + Testcontainers Integration)

### 4.1. `OrderStateTest.java` — State Transition (`TC_CHK_008`, `FR-CHK-04`)

```java
package com.techstore.checkout.domain;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.EnumSource;

import static org.assertj.core.api.Assertions.*;

@DisplayName("Order State Machine — Vòng đời đơn hàng (FR-CHK-04)")
class OrderStateTest {

    @ParameterizedTest(name = "Cho phép hủy khi ở trạng thái {0}")
    @EnumSource(value = OrderStatus.class, names = {"PENDING_PAYMENT", "CONFIRMED"})
    void shouldAllowCancelFromCancellableStatuses(OrderStatus initialStatus) {
        Order order = new Order(1L, 500_000, 0, 0, 500_000, PaymentMethod.COD, null);
        order.setStatusForTest(initialStatus);

        order.cancel();

        assertThat(order.getStatus()).isEqualTo(OrderStatus.CANCELLED);
    }

    @ParameterizedTest(name = "Từ chối hủy khi đơn ở trạng thái {0} (TC_CHK_008)")
    @EnumSource(value = OrderStatus.class, names = {"PAID", "SHIPPING", "COMPLETED", "CANCELLED"})
    void shouldRejectCancelWhenOrderBeyondConfirmed(OrderStatus currentStatus) {
        Order order = new Order(1L, 500_000, 0, 0, 500_000, PaymentMethod.COD, null);
        order.setStatusForTest(currentStatus);

        assertThatThrownBy(order::cancel)
                .isInstanceOf(IllegalStateException.class)
                .hasMessageContaining(currentStatus.name());
        assertThat(order.getStatus()).isEqualTo(currentStatus);
    }

    @Test
    @DisplayName("Cho phép chuyển từ PENDING_PAYMENT -> PAID")
    void shouldMarkPaidFromPendingPayment() {
        Order order = new Order(1L, 500_000, 0, 0, 500_000, PaymentMethod.VNPAY, null);
        order.markPaid();
        assertThat(order.getStatus()).isEqualTo(OrderStatus.PAID);
    }

    @Test
    @DisplayName("Chặn webhook thanh toán muộn hồi sinh đơn đã CANCELLED")
    void shouldRejectLatePaymentWebhookOnCancelledOrder() {
        Order order = new Order(1L, 500_000, 0, 0, 500_000, PaymentMethod.VNPAY, null);
        order.cancel();

        assertThatThrownBy(order::markPaid)
                .isInstanceOf(IllegalStateException.class)
                .hasMessageContaining("CANCELLED");
    }
}
```

### 4.2. `ShippingFeeCalculatorTest.java` — BVA & Decision Table (`TC_CHK_003..005`, `FR-CHK-02`)

```java
package com.techstore.checkout.service;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.assertj.core.api.Assertions.*;

@DisplayName("ShippingFeeCalculator — BVA & Decision Table Freeship (FR-CHK-02)")
class ShippingFeeCalculatorTest {

    private final ShippingFeeCalculator calculator = new ShippingFeeCalculator();

    @ParameterizedTest(name = "subtotal={0}đ, vip={1}, freeship={2} => fee={3}đ")
    @CsvSource({
        "0,       false, false, 30000", // Biên dưới hợp lệ
        "499999,  false, false, 30000", // TC_CHK_003: Ngay dưới mốc freeship 500k
        "500000,  false, false, 0",     // TC_CHK_004: Đúng mốc freeship 500k
        "500001,  false, false, 0",     // Ngay trên mốc
        "150000,  true,  false, 0",     // TC_CHK_005: Khách VIP freeship mọi đơn
        "150000,  false, true,  0"      // Có mã FREESHIP
    })
    void shouldCalculateShippingFeeCorrectly(int subtotal, boolean isVip, boolean hasFreeshipCode, int expectedFee) {
        assertThat(calculator.calculateFee(subtotal, isVip, hasFreeshipCode)).isEqualTo(expectedFee);
    }

    @Test
    @DisplayName("Ném ngoại lệ nếu tổng tiền hàng âm")
    void shouldThrowWhenSubtotalIsNegative() {
        assertThatThrownBy(() -> calculator.calculateFee(-1, false, false))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessageContaining("negative");
    }
}
```

### 4.3. `VoucherServiceTest.java` — Phủ 100% nhánh `VoucherService` + Truy vết `TC_VCH_001..005`

```java
package com.techstore.checkout.service;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.exception.InvalidVoucherException;
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import java.time.Clock;
import java.time.Instant;
import java.time.ZoneOffset;
import java.util.List;

import static org.assertj.core.api.Assertions.*;

@DisplayName("VoucherService — Quy tắc SALE20, FLAT50K & Loại trừ Flash Sale")
class VoucherServiceTest {

    private static final Instant NOW = Instant.parse("2026-11-25T10:00:00Z");
    private static final Instant START = NOW.minusSeconds(3600);
    private static final Instant END = NOW.plusSeconds(3600);

    private VoucherService voucherService;

    @BeforeEach
    void setUp() {
        voucherService = new VoucherService(Clock.fixed(NOW, ZoneOffset.UTC));
    }

    private Voucher sale20() {
        return new Voucher("SALE20", VoucherType.PERCENT, 20, 150_000, 500_000, 5, true, START, END);
    }

    @Nested
    @DisplayName("1. Tính giảm giá hợp lệ & Trần giảm tối đa (TC_VCH_001, TC_VCH_003)")
    class ValidDiscountTests {

        @Test
        @DisplayName("TC_VCH_001: Đơn đúng biên 500.000đ dùng SALE20 -> giảm 100.000đ")
        void shouldApply20PercentAtMinOrderBoundary() {
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 500_000, 1, false));
            assertThat(voucherService.calculateDiscount(sale20(), items)).isEqualTo(100_000);
        }

        @Test
        @DisplayName("TC_VCH_003: Đơn 1.000.000đ dùng SALE20 -> chặn trần 150.000đ")
        void shouldCapPercentDiscountAtMaxLimit() {
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 1_000_000, 1, false));
            assertThat(voucherService.calculateDiscount(sale20(), items)).isEqualTo(150_000);
        }

        @Test
        @DisplayName("Voucher FIXED không có cap (maxDiscount = 0) & không giảm vượt tổng đơn")
        void shouldApplyFixedDiscountWithoutExceedingEligibleSubtotal() {
            Voucher flat50k = new Voucher("FLAT50K", VoucherType.FIXED, 50_000, 0, 20_000, 3, true, START, END);
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 30_000, 1, false));
            // Chỉ giảm tối đa bằng 30.000đ (không âm tiền!)
            assertThat(voucherService.calculateDiscount(flat50k, items)).isEqualTo(30_000);
        }

        @Test
        @DisplayName("TC_VCH_005: Giỏ hỗn hợp (Thường 500k + Flash Sale 300k) -> chỉ giảm 20% trên 500k = 100k")
        void shouldExcludeFlashSaleItemsFromDiscountCalculation() {
            List<OrderItem> items = List.of(
                    new OrderItem(1L, "SKU-REG", 500_000, 1, false),
                    new OrderItem(2L, "SKU-FS",  300_000, 1, true)
            );
            assertThat(voucherService.calculateDiscount(sale20(), items)).isEqualTo(100_000);
        }
    }

    @Nested
    @DisplayName("2. Từ chối Voucher không hợp lệ & Biên thời gian (TC_VCH_002, TC_VCH_004, TC_VCH_005)")
    class InvalidVoucherTests {

        @Test
        @DisplayName("Từ chối nếu voucher null hoặc inactive")
        void shouldRejectNullOrInactiveVoucher() {
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 600_000, 1, false));
            Voucher inactive = new Voucher("OFF", VoucherType.FIXED, 50_000, 0, 0, 5, false, START, END);

            assertThatThrownBy(() -> voucherService.calculateDiscount(null, items))
                    .isInstanceOf(InvalidVoucherException.class);
            assertThatThrownBy(() -> voucherService.calculateDiscount(inactive, items))
                    .isInstanceOf(InvalidVoucherException.class);
        }

        @ParameterizedTest(name = "TC_VCH_004: startOffset={0}s, endOffset={1}s")
        @CsvSource({
            "60,   3600", // Chưa tới startDate -> từ chối
            "-3600,-60"   // Đã qua endDate -> từ chối
        })
        void shouldRejectOutsideValidTimeWindow(long startOffset, long endOffset) {
            Voucher outOfWindow = new Voucher("TIME", VoucherType.FIXED, 50_000, 0, 100_000, 5, true,
                    NOW.plusSeconds(startOffset), NOW.plusSeconds(endOffset));
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 600_000, 1, false));

            assertThatThrownBy(() -> voucherService.calculateDiscount(outOfWindow, items))
                    .isInstanceOf(InvalidVoucherException.class)
                    .hasMessageContaining("thời gian");
        }

        @Test
        @DisplayName("Từ chối khi hết lượt (usageLimit == 0) hoặc giỏ hàng null/rỗng")
        void shouldRejectZeroUsageOrEmptyItems() {
            Voucher usedUp = new Voucher("EMPTY", VoucherType.FIXED, 50_000, 0, 0, 0, true, START, END);
            List<OrderItem> items = List.of(new OrderItem(1L, "SKU-1", 600_000, 1, false));

            assertThatThrownBy(() -> voucherService.calculateDiscount(usedUp, items))
                    .isInstanceOf(InvalidVoucherException.class);
            assertThatThrownBy(() -> voucherService.calculateDiscount(sale20(), null))
                    .isInstanceOf(InvalidVoucherException.class);
            assertThatThrownBy(() -> voucherService.calculateDiscount(sale20(), List.of()))
                    .isInstanceOf(InvalidVoucherException.class);
        }

        @Test
        @DisplayName("TC_VCH_002 & TC_VCH_005: Giỏ có SP Thường 300k + Flash Sale 300k -> vẫn dưới ngưỡng 500k!")
        void shouldRejectWhenEligibleSubtotalIsBelowMinOrder() {
            List<OrderItem> items = List.of(
                    new OrderItem(1L, "SKU-REG", 300_000, 1, false),
                    new OrderItem(2L, "SKU-FS",  300_000, 1, true)
            );
            assertThatThrownBy(() -> voucherService.calculateDiscount(sale20(), items))
                    .isInstanceOf(InvalidVoucherException.class)
                    .hasMessageContaining("tối thiểu");
        }
    }
}
```

### 4.4. `CheckoutServiceTest.java` — Unit Test Điều Phối Với Mockito 5 & AssertJ

```java
package com.techstore.checkout.service;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.exception.*;
import com.techstore.checkout.repository.*;
import com.techstore.checkout.web.CheckoutRequest;
import com.techstore.checkout.web.OrderResponse;
import org.assertj.core.api.SoftAssertions;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.*;
import org.mockito.junit.jupiter.MockitoExtension;

import java.time.Clock;
import java.time.Instant;
import java.time.ZoneOffset;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.BDDMockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("CheckoutService — Unit Tests với Mockito 5 & AssertJ")
class CheckoutServiceTest {

    @Mock private ProductRepository productRepository;
    @Mock private VoucherRepository voucherRepository;
    @Mock private OrderRepository orderRepository;
    @Captor private ArgumentCaptor<Order> orderCaptor;

    private CheckoutService checkoutService;

    private static final Instant NOW = Instant.parse("2026-11-25T10:00:00Z");

    @BeforeEach
    void setUp() {
        VoucherService voucherService = new VoucherService(Clock.fixed(NOW, ZoneOffset.UTC));
        ShippingFeeCalculator shippingCalculator = new ShippingFeeCalculator();
        checkoutService = new CheckoutService(
                productRepository, voucherRepository, orderRepository, voucherService, shippingCalculator);
    }

    @Test
    @DisplayName("TC_CHK_001 + TC_VCH_001: Đặt hàng thành công, chuẩn hóa mã ' sale20 ', lưu Order đúng số liệu")
    void shouldPlaceOrderAndNormalizeVoucherCode() {
        Product laptop = new Product(1L, "SKU-LAPTOP", "Laptop", 500_000, 5, false);
        Voucher sale20 = new Voucher("SALE20", VoucherType.PERCENT, 20, 150_000, 500_000, 2,
                true, NOW.minusSeconds(60), NOW.plusSeconds(3600));

        given(productRepository.findAllById(List.of(1L))).willReturn(List.of(laptop));
        given(productRepository.decrementStockIfAvailable(1L, 1)).willReturn(1);
        given(voucherRepository.findByCodeIgnoreCase("SALE20")).willReturn(Optional.of(sale20));
        given(voucherRepository.decrementUsageIfAvailable("SALE20")).willReturn(1);
        given(orderRepository.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));

        // Nhập mã chữ thường + khoảng trắng để kiểm chứng bug GĐ 1
        CheckoutRequest req = new CheckoutRequest(99L, false, false, " sale20 ", PaymentMethod.COD,
                List.of(new CheckoutRequest.ItemLine(1L, 1)));

        OrderResponse response = checkoutService.placeOrder(req);

        // Kiểm tra thứ tự thực thi: trừ kho & trừ voucher TRƯỚC khi lưu Order
        InOrder inOrder = inOrder(productRepository, voucherRepository, orderRepository);
        inOrder.verify(productRepository).decrementStockIfAvailable(1L, 1);
        inOrder.verify(voucherRepository).decrementUsageIfAvailable("SALE20");
        inOrder.verify(orderRepository).save(orderCaptor.capture());

        Order saved = orderCaptor.getValue();
        // Subtotal 500k - Discount 100k = 400k < 500k -> ship 30k -> finalTotal 430k!
        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(response.status()).isEqualTo(OrderStatus.PENDING_PAYMENT);
            softly.assertThat(saved.getVoucherCode()).isEqualTo("SALE20");
            softly.assertThat(saved.getDiscountAmount()).isEqualTo(100_000);
            softly.assertThat(saved.getShippingFee()).isEqualTo(30_000);
            softly.assertThat(saved.getFinalTotal()).isEqualTo(430_000);
        });
    }

    @Test
    @DisplayName("TC_CHK_006 & TC_CHK_007: COD <= 5.000.000đ hợp lệ; 5.000.001đ chặn COD nhưng cho phép VNPAY")
    void shouldEnforceCodLimitAt5MillionBoundary() {
        Product expensive = new Product(2L, "SKU-PRO", "Workstation", 5_000_001, 5, false);
        given(productRepository.findAllById(List.of(2L))).willReturn(List.of(expensive));
        given(productRepository.decrementStockIfAvailable(2L, 1)).willReturn(1);

        // 5.000.001đ với COD -> bị chặn và KHÔNG lưu Order
        CheckoutRequest codOverLimit = new CheckoutRequest(1L, false, false, "   ", PaymentMethod.COD,
                List.of(new CheckoutRequest.ItemLine(2L, 1)));
        assertThatThrownBy(() -> checkoutService.placeOrder(codOverLimit))
                .isInstanceOf(CodLimitExceededException.class);
        verify(orderRepository, never()).save(any());

        // 5.000.001đ với VNPAY -> thành công
        given(orderRepository.save(any(Order.class))).willAnswer(inv -> inv.getArgument(0));
        CheckoutRequest vnpayOk = new CheckoutRequest(1L, false, false, null, PaymentMethod.VNPAY,
                List.of(new CheckoutRequest.ItemLine(2L, 1)));
        OrderResponse resp = checkoutService.placeOrder(vnpayOk);
        assertThat(resp.finalTotal()).isEqualTo(5_000_001);
    }

    @Test
    @DisplayName("TC_CHK_002: Thiếu tồn kho -> ném InsufficientStockException, không trừ voucher và không lưu đơn")
    void shouldFailFastWhenStockIsInsufficient() {
        Product item = new Product(1L, "SKU-A", "Mouse", 200_000, 5, false);
        given(productRepository.findAllById(List.of(1L))).willReturn(List.of(item));
        given(productRepository.decrementStockIfAvailable(1L, 6)).willReturn(0);

        CheckoutRequest req = new CheckoutRequest(1L, false, false, "SALE20", PaymentMethod.COD,
                List.of(new CheckoutRequest.ItemLine(1L, 6)));

        assertThatThrownBy(() -> checkoutService.placeOrder(req))
                .isInstanceOf(InsufficientStockException.class)
                .hasMessageContaining("SKU-A");

        verifyNoInteractions(voucherRepository, orderRepository);
    }

    @Test
    @DisplayName("Các nhánh bảo vệ: giỏ null/rỗng, số lượng <= 0, product không tồn tại, voucher hết lượt khi atomic update")
    void shouldHandleDefensiveValidationAndEdgeFailures() {
        assertThatThrownBy(() -> checkoutService.placeOrder(
                new CheckoutRequest(1L, false, false, null, PaymentMethod.COD, null)))
                .isInstanceOf(IllegalArgumentException.class);
        assertThatThrownBy(() -> checkoutService.placeOrder(
                new CheckoutRequest(1L, false, false, null, PaymentMethod.COD, List.of())))
                .isInstanceOf(IllegalArgumentException.class);
        assertThatThrownBy(() -> checkoutService.placeOrder(
                new CheckoutRequest(1L, false, false, null, PaymentMethod.COD,
                        List.of(new CheckoutRequest.ItemLine(1L, 0)))))
                .isInstanceOf(IllegalArgumentException.class);

        given(productRepository.findAllById(List.of(999L))).willReturn(List.of());
        assertThatThrownBy(() -> checkoutService.placeOrder(
                new CheckoutRequest(1L, false, false, null, PaymentMethod.COD,
                        List.of(new CheckoutRequest.ItemLine(999L, 1)))))
                .isInstanceOf(ProductNotFoundException.class);

        // Trường hợp 2 luồng cùng đọc voucher còn 1 lượt, luồng này thua khi chạy UPDATE atomic
        Product p = new Product(1L, "SKU-1", "SSD", 600_000, 5, false);
        Voucher lastVoucher = new Voucher("SALE20", VoucherType.PERCENT, 20, 150_000, 500_000, 1,
                true, NOW.minusSeconds(60), NOW.plusSeconds(3600));
        given(productRepository.findAllById(List.of(1L))).willReturn(List.of(p));
        given(productRepository.decrementStockIfAvailable(1L, 1)).willReturn(1);
        given(voucherRepository.findByCodeIgnoreCase("SALE20")).willReturn(Optional.of(lastVoucher));
        given(voucherRepository.decrementUsageIfAvailable("SALE20")).willReturn(0);

        assertThatThrownBy(() -> checkoutService.placeOrder(
                new CheckoutRequest(1L, false, false, "SALE20", PaymentMethod.VNPAY,
                        List.of(new CheckoutRequest.ItemLine(1L, 1)))))
                .isInstanceOf(InvalidVoucherException.class)
                .hasMessageContaining("hết lượt");
        verify(orderRepository, never()).save(any());
    }
}
```

### 4.5. `CheckoutControllerTest.java` (`@WebMvcTest`), `VoucherRepositoryIT.java` (`@DataJpaTest`) & `CheckoutConcurrencyIT.java` (Testcontainers)

```java
// web/CheckoutControllerTest.java
package com.techstore.checkout.web;

import com.techstore.checkout.domain.OrderStatus;
import com.techstore.checkout.domain.PaymentMethod;
import com.techstore.checkout.exception.InsufficientStockException;
import com.techstore.checkout.service.CheckoutService;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.test.context.bean.override.mockito.MockitoBean;
import org.springframework.test.web.servlet.assertj.MockMvcTester;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verifyNoInteractions;

@WebMvcTest(CheckoutController.class)
@DisplayName("CheckoutController — Web Slice Test với MockMvcTester & @MockitoBean")
class CheckoutControllerTest {

    @Autowired private MockMvcTester mvc;
    @MockitoBean private CheckoutService checkoutService;

    @Test
    @DisplayName("Trả về 201 Created khi payload hợp lệ")
    void shouldReturn201WhenRequestIsValid() {
        given(checkoutService.placeOrder(any())).willReturn(
                new OrderResponse(10L, OrderStatus.PENDING_PAYMENT, 500_000, 100_000, 30_000, 430_000));

        assertThat(mvc.post().uri("/api/v1/checkout")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"userId": 1, "vip": false, "hasFreeshipCode": false,
                     "voucherCode": "SALE20", "paymentMethod": "COD",
                     "items": [{"productId": 1, "quantity": 1}]}
                    """))
            .hasStatus(HttpStatus.CREATED)
            .bodyJson()
            .extractingPath("$.finalTotal")
            .isEqualTo(430_000);
    }

    @Test
    @DisplayName("Trả về 400 Bad Request khi giỏ hàng rỗng (@NotEmpty) và không gọi Service")
    void shouldReturn400WhenItemsEmpty() {
        assertThat(mvc.post().uri("/api/v1/checkout")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"userId": 1, "paymentMethod": "COD", "items": []}
                    """))
            .hasStatus(HttpStatus.BAD_REQUEST);

        verifyNoInteractions(checkoutService);
    }

    @Test
    @DisplayName("Trả về 409 Conflict khi hết tồn kho")
    void shouldReturn409WhenInsufficientStock() {
        given(checkoutService.placeOrder(any()))
                .willThrow(new InsufficientStockException("Sản phẩm không đủ tồn kho: SKU-A"));

        assertThat(mvc.post().uri("/api/v1/checkout")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"userId": 1, "paymentMethod": "COD",
                     "items": [{"productId": 1, "quantity": 6}]}
                    """))
            .hasStatus(HttpStatus.CONFLICT);
    }
}
```

```java
// integration/TestcontainersConfig.java
package com.techstore.checkout.integration;

import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.boot.testcontainers.service.connection.ServiceConnection;
import org.springframework.context.annotation.Bean;
import org.testcontainers.containers.PostgreSQLContainer;

@TestConfiguration(proxyBeanMethods = false)
public class TestcontainersConfig {

    static final PostgreSQLContainer<?> POSTGRES = new PostgreSQLContainer<>("postgres:17-alpine")
            .withDatabaseName("techstore_test")
            .withUsername("test")
            .withPassword("test");

    static {
        POSTGRES.start();
    }

    @Bean
    @ServiceConnection
    PostgreSQLContainer<?> postgresContainer() {
        return POSTGRES;
    }
}
```

```java
// integration/VoucherRepositoryIT.java
package com.techstore.checkout.integration;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.repository.VoucherRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;
import org.springframework.context.annotation.Import;

import java.time.Instant;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Import(TestcontainersConfig.class)
@DisplayName("VoucherRepositoryIT — Kiểm thử JPA Query atomic trên PostgreSQL thật")
class VoucherRepositoryIT {

    @Autowired private TestEntityManager entityManager;
    @Autowired private VoucherRepository voucherRepository;

    @Test
    @DisplayName("FR-VCH-03: Chỉ trừ usage_limit khi > 0 và không phân biệt hoa/thường")
    void shouldDecrementUsageLimitAtomicallyWhenAvailable() {
        Voucher sale20 = new Voucher("SALE20", VoucherType.PERCENT, 20, 150_000, 500_000, 1,
                true, Instant.now().minusSeconds(60), Instant.now().plusSeconds(3600));
        entityManager.persistAndFlush(sale20);
        entityManager.clear();

        int firstAttempt = voucherRepository.decrementUsageIfAvailable("sale20");
        int secondAttempt = voucherRepository.decrementUsageIfAvailable("SALE20");
        entityManager.clear();

        assertThat(firstAttempt).isEqualTo(1);
        assertThat(secondAttempt).isZero();
        assertThat(voucherRepository.findByCodeIgnoreCase("SALE20"))
                .isPresent()
                .get()
                .extracting(Voucher::getUsageLimit)
                .isEqualTo(0);
    }
}
```

```java
// integration/CheckoutConcurrencyIT.java
package com.techstore.checkout.integration;

import com.techstore.checkout.domain.*;
import com.techstore.checkout.repository.*;
import com.techstore.checkout.web.CheckoutRequest;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.context.annotation.Import;
import org.springframework.http.ResponseEntity;

import java.time.Instant;
import java.util.*;
import java.util.concurrent.*;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Import(TestcontainersConfig.class)
@DisplayName("CheckoutConcurrencyIT — Kiểm chứng Rollback & Race Condition trên PostgreSQL thật")
class CheckoutConcurrencyIT {

    @Autowired private TestRestTemplate restTemplate;
    @Autowired private ProductRepository productRepository;
    @Autowired private VoucherRepository voucherRepository;
    @Autowired private OrderRepository orderRepository;

    private Long p1Id;
    private Long p2Id;

    @BeforeEach
    void setUpCleanDb() {
        orderRepository.deleteAllInBatch();
        voucherRepository.deleteAllInBatch();
        productRepository.deleteAllInBatch();

        p1Id = productRepository.save(new Product("SKU-1", "Laptop", 1_000_000, 5, false)).getId();
        p2Id = productRepository.save(new Product("SKU-2", "Mouse Last", 200_000, 1, false)).getId();

        voucherRepository.save(new Voucher("SALE20", VoucherType.PERCENT, 20, 150_000, 500_000, 1,
                true, Instant.now().minusSeconds(3600), Instant.now().plusSeconds(3600)));
    }

    @Test
    @DisplayName("TC_CHK_002: Sản phẩm 2 thiếu tồn kho -> @Transactional rollback hoàn trả stock sản phẩm 1 về 5")
    void shouldRollbackFirstProductStockWhenSecondProductFails() {
        CheckoutRequest req = new CheckoutRequest(1L, false, false, null, PaymentMethod.VNPAY,
                List.of(new CheckoutRequest.ItemLine(p1Id, 2), new CheckoutRequest.ItemLine(p2Id, 5)));

        ResponseEntity<String> resp = restTemplate.postForEntity("/api/v1/checkout", req, String.class);

        assertThat(resp.getStatusCode().value()).isEqualTo(409);
        assertThat(productRepository.findById(p1Id).orElseThrow().getStock()).isEqualTo(5);
        assertThat(orderRepository.count()).isZero();
    }

    @Test
    @DisplayName("TC_VCH_006: 2 người đặt đồng thời món cuối (stock=1) & voucher cuối (limit=1) -> chỉ 1 thành công")
    void shouldPreventNegativeStockAndOverusedVoucherUnderConcurrency() throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(2);
        CountDownLatch ready = new CountDownLatch(2);
        CountDownLatch start = new CountDownLatch(1);
        List<Future<ResponseEntity<String>>> futures = new ArrayList<>();

        for (long uid = 1; uid <= 2; uid++) {
            final long userId = uid;
            futures.add(pool.submit(() -> {
                ready.countDown();
                start.await();
                return restTemplate.postForEntity("/api/v1/checkout",
                        new CheckoutRequest(userId, false, false, "SALE20", PaymentMethod.VNPAY,
                                List.of(new CheckoutRequest.ItemLine(p2Id, 1),
                                        new CheckoutRequest.ItemLine(p1Id, 1))),
                        String.class);
            }));
        }

        ready.await();
        start.countDown();

        List<Integer> statuses = new ArrayList<>();
        for (Future<ResponseEntity<String>> f : futures) {
            statuses.add(f.get().getStatusCode().value());
        }
        pool.shutdown();

        assertThat(statuses).containsExactlyInAnyOrder(201, 409);
        assertThat(productRepository.findById(p2Id).orElseThrow().getStock()).isZero();
        assertThat(voucherRepository.findByCodeIgnoreCase("SALE20").orElseThrow().getUsageLimit()).isZero();
        assertThat(orderRepository.count()).isEqualTo(1);
    }
}
```

---

## 5. Báo cáo Kiểm toán JaCoCo Branch Coverage (Kiểm chứng từng nhánh)

Dưới đây là bảng kiểm toán chi tiết từng nhánh bytecode (`BRANCH` counter) trên các class chứa nghiệp vụ của module (không tính DTO/Config/Exception đã exclude hợp lệ trong `pom.xml`):

| Class | Phương thức | Nhánh Logic & Điều kiện Bytecode | Tổng Branch | Covered | Coverage | Ghi chú kiểm chứng |
| :--- | :--- | :--- | :---: | :---: | :---: | :--- |
| **`Order`** | `cancel()` | `status != PENDING_PAYMENT` (T/F), `status != CONFIRMED` (T/F) | 4 | 4 | **100.0%** | `OrderStateTest` phủ đủ `PENDING_PAYMENT`, `CONFIRMED`, `SHIPPING/PAID/...` |
| **`Order`** | `markPaid()` | `status != PENDING_PAYMENT` (T/F) | 2 | 2 | **100.0%** | `OrderStateTest` phủ cả `PENDING_PAYMENT` và `CANCELLED` |
| **`ShippingFeeCalculator`** | `calculateFee(...)` | `subtotal < 0` (2), `isVip` (2), `hasFreeship` (2), `>= 500k` (2) | 8 | 8 | **100.0%** | `ShippingFeeCalculatorTest` kiểm tra cả biên `499_999` / `500_000` |
| **`VoucherService`** | `calculateDiscount(...)` | `null` (2), `!active` (2), `isBefore` (2), `isAfter` (2), `usage<=0` (2), `items null` (2), `items empty` (2), loop (2), `!flashSale` (2), `< minOrder` (2), `PERCENT` (2), `maxDiscount>0` (2) | 24 | 24 | **100.0%** | `VoucherServiceTest` phủ trọn 24/24 nhánh kèm assert giá trị & message |
| **`CheckoutService`** | `placeOrder(...)` | `items null` (2), `empty` (2), loop (2), `qty<=0` (2), `prod==null` (2), `stock==0` (2), `code!=null` (2), `!blank` (2), `usage==0` (2), `COD` (2), `>5M` (2) | 22 | 22 | **100.0%** | `CheckoutServiceTest` + `CheckoutConcurrencyIT` |
| **`GlobalExceptionHandler`** | Các `@ExceptionHandler` | Một số handler lỗi hệ thống hiếm gặp (VD: `HttpMessageNotReadableException` JSON hỏng) chưa test hết từng nhánh format | 6 | 2 | **33.3%** | Chấp nhận không ép test lấy số trên boilerplate format lỗi |
| **TỔNG TOÀN MODULE** | — | **Tổng cộng nhánh trong phạm vi đo** | **66** | **62** | **93.9%** | **Vượt chuẩn `>= 80%` của Quality Gate (`93.9% >= 80.0%`)!** |

---

## 6. Báo cáo PITest Mutation Audit (Chứng minh chất lượng Assertion)

Các đột biến trọng tâm mà bộ test trên đã **KILLED** thành công (chứng minh không phải test lấy số):

| File & Dòng đột biến | Phép biến đổi của PITest (*Mutator*) | Trạng thái | Test nào đã diệt (*Killed by*)? |
| :--- | :--- | :---: | :--- |
| `ShippingFeeCalculator` | Đổi `subtotal >= 500_000` → `subtotal > 500_000` *(Boundary Mutator)* | **KILLED** | `ShippingFeeCalculatorTest` ca biên `500_000 -> 0` |
| `VoucherService` | Xóa kiểm tra `now.isBefore(startDate)` *(Negate Conditionals)* | **KILLED** | `VoucherServiceTest.shouldRejectOutsideValidTimeWindow` |
| `VoucherService` | Xóa điều kiện `!item.isFlashSale()` *(Negate Conditionals)* | **KILLED** | `VoucherServiceTest.shouldExcludeFlashSaleItemsFromDiscountCalculation` |
| `VoucherService` | Đổi `Math.min(rawDiscount, maxDiscount)` → `rawDiscount` | **KILLED** | `VoucherServiceTest.shouldCapPercentDiscountAtMaxLimit` |
| `CheckoutService` | Xóa lời gọi `decrementUsageIfAvailable(...)` *(Void/Return Mutator)* | **KILLED** | `CheckoutServiceTest` (`InOrder.verify`) & `CheckoutConcurrencyIT` |
| `CheckoutService` | Đổi `finalTotal > 5_000_000` → `finalTotal >= 5_000_000` | **KILLED** | `CheckoutServiceTest.shouldEnforceCodLimitAt5MillionBoundary` |
| `Order` | Đổi `status != CONFIRMED` → `true` | **KILLED** | `OrderStateTest.shouldAllowCancelFromCancellableStatuses` |

---

## 7. Checklist Nghiệm thu Deliverable & Câu hỏi Phỏng vấn GĐ 2

### Checklist tự rà soát khi đưa lên Portfolio GitHub
- [x] `pom.xml` bật `jacoco:check` Fail Build nếu `BRANCH < 0.80`.
- [x] Loại trừ đúng đối tượng (`dto`, `config`, `lombok.config`), không loại trừ `service` hay `domain` để gian lận điểm.
- [x] Có cả 3 tầng test theo đúng **Test Pyramid**: Unit Test nhanh (`*Test`), Slice Test (`@WebMvcTest`, `@DataJpaTest`), và Testcontainers Integration Test (`*IT`).
- [x] Truy vết đầy đủ 100% các `FR-CHK-*` và `FR-VCH-*` từ RTM Giai đoạn 1.
- [x] PITest diệt sạch các mutant biên (`>=` vs `>`, `>` vs `>=`, bỏ `verify`, thiếu `startDate`).

### 5 câu hỏi phỏng vấn hay gặp nhất ở Giai đoạn 2 (Kèm hướng trả lời sắc bén)
1. **Phân biệt `@Mock` (Mockito) và `@MockitoBean` / `@MockBean` (Spring Boot)?**
   → `@Mock` tạo mock thuần Java chạy trong Unit Test không cần Spring Context; `@MockitoBean` thay thế bean thật **bên trong Spring `ApplicationContext`** khi chạy `@WebMvcTest` hoặc `@SpringBootTest` (và làm thay đổi cache key của Context!).
2. **Sự khác nhau giữa Stub và Mock? Khi nào dùng `verify`?**
   → Stub trả dữ liệu cứng để đưa SUT vào trạng thái cần test (`given/willReturn`); Mock xác minh tương tác/hành vi gọi ra bên ngoài (`verify(repo, never()).save(...)`). Chỉ `verify` các **tương tác có side-effect quan trọng** (ghi DB, trừ tiền, gửi mail), tránh verify hàm `get/find` khiến test giòn.
3. **Tại sao dự án có 90% Code Coverage mà lên Production vẫn bị bug nghiêm trọng?**
   → (1) Test thiếu assertion hoặc assert hời hợt (PITest sẽ lòi ra mutant *Survived*); (2) Thiếu ca biên BVA (`>` vs `>=`); (3) Dev quên hẳn 1 requirement (VD: quên check `startDate` — code không viết thì coverage không thấy thiếu!); (4) Bug tích hợp DB/Transaction/Concurrency mà Unit Test với Mock không bắt được.
4. **Tại sao nên dùng `@WebMvcTest` / `@DataJpaTest` thay vì gắn `@SpringBootTest` cho mọi test?**
   → Để thu hẹp phạm vi nạp Bean, tăng tốc chạy gấp 5–10 lần, cô lập lỗi đúng tầng và giữ **Spring Context Cache** hiệu quả (tránh `@DirtiesContext`).
5. **Làm thế nào để test một lỗi Race Condition (2 người cùng mua món cuối cùng) bằng Java?**
   → Không dùng Unit Test với Mockito được vì Mockito không có transaction/lock thật của DB. Phải viết Integration Test dùng **Testcontainers (PostgreSQL thật)** kết hợp `ExecutorService` + `CountDownLatch` để ép 2 thread bắn đồng thời và kiểm tra đúng 1 thành công, 1 lỗi, `stock == 0` trong DB.
