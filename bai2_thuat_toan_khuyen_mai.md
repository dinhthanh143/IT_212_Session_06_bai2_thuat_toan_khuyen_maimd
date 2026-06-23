# BÀI 2: THỰC HÀNH XÂY DỰNG THUẬT TOÁN KHUYẾN MÃI PHỨC TẠP (CHAIN-OF-THOUGHT - CoT)

## 1. Giải thích tầm quan trọng của thứ tự ưu tiên tính toán trong các nghiệp vụ kế toán/tài chính
Trong các hệ thống phần mềm thương mại điện tử, kế toán và tài chính, thứ tự ưu tiên áp dụng các quy tắc giảm giá, chiết khấu, thuế và phí vận chuyển là cực kỳ quan trọng vì những lý do sau:
1. **Tránh thất thoát tài chính doanh nghiệp:** Nếu áp dụng chiết khấu phần trăm (như giảm giá thành viên 10%) sau khi cộng các chi phí phụ thu (như phí vận chuyển hoặc thuế), doanh nghiệp vô tình giảm giá luôn cả phần phí dịch vụ và thuế, dẫn đến thất thoát tiền.
2. **Tính hợp pháp về mặt Thuế (Tax Compliance):** Thuế giá trị gia tăng (VAT) phải được tính trên giá trị thực tế của hàng hóa sau khi đã trừ đi các khoản giảm giá, chiết khấu hợp lệ được đăng ký với Bộ Công Thương. Nếu tính thuế VAT trên giá gốc trước khi giảm giá, khách hàng sẽ phải chịu mức thuế cao hơn quy định của pháp luật, hoặc ngược lại, doanh nghiệp có thể bị khép vào tội trốn thuế hoặc báo cáo tài chính sai lệch.
3. **Độ chính xác của tiền tệ (Precision & Rounding Errors):** Sử dụng sai kiểu dữ liệu (như `float` hoặc `double`) thay vì `BigDecimal` sẽ gây ra các sai số làm tròn nhị phân (IEEE 754). Khi tích lũy hàng triệu giao dịch, những sai số vài đồng lẻ này sẽ tích tụ thành những khoản chênh lệch tài chính khổng lồ, gây khó khăn cho việc đối soát kế toán cuối tháng.
4. **Logic nghiệp vụ nhất quán:** Thứ tự áp dụng rõ ràng giúp bộ phận vận hành và marketing tính toán được chính xác biên lợi nhuận của từng chiến dịch khuyến mãi.

---

## 2. Nội dung Prompt CoT thiết kế gửi cho AI
Dưới đây là prompt áp dụng kỹ thuật Chain-of-thought (CoT) để hướng dẫn AI suy luận từng bước:

```text
[Role]
Bạn là một Chuyên gia phân tích nghiệp vụ (Business Analyst) kiêm Senior Java Developer trong lĩnh vực E-Commerce FinTech.

[Context]
Hệ thống SpeedyCart cần tính tổng số tiền thanh toán cuối cùng của đơn hàng dựa trên một loạt quy tắc khuyến mãi, thuế và phí vận chuyển phức tạp.

[Task & Chain-of-Thought Instructions]
Hãy phân tích và giải quyết bài toán theo phương pháp suy luận từng bước (Chain-of-thought) như sau:
1. **Phân tích logic nghiệp vụ & Xác định thứ tự thực hiện:** Xác định thứ tự ưu tiên tính toán đúng đắn nhất giữa: Tổng tiền gốc, Giảm giá sản phẩm trực tiếp, Mã giảm giá (Coupon), Chiết khấu thành viên (Loyalty), Phí vận chuyển, và Thuế VAT. Hãy giải thích tại sao thứ tự đó là hợp lệ về mặt kế toán/tài chính.
2. **Xây dựng công thức tính chi tiết:** Liệt kê công thức toán học rõ ràng cho từng bước.
3. **Chạy thử bằng tay (Dry-run):** Thực hiện tính toán từng bước bằng văn bản cho ca kiểm thử cụ thể sau:
   - Giỏ hàng có:
     + 2 sản phẩm A (giá gốc 400.000 VND/sản phẩm, giảm giá trực tiếp 10%).
     + 1 sản phẩm B (giá gốc 300.000 VND/sản phẩm, không giảm giá).
   - Khách hàng áp dụng mã coupon: Giảm 50.000 VND (yêu cầu đơn hàng tối thiểu sau giảm giá sản phẩm đạt 500.000 VND).
   - Khách hàng hạng Gold: Giảm 5% trên tổng tiền đã trừ coupon.
   - Phí vận chuyển: Dưới 10km tính 30.000 VND, mỗi km tiếp theo cộng 5.000 VND. Địa chỉ nhận hàng cách kho 12km. Miễn phí vận chuyển nếu tổng tiền đơn hàng sau chiết khấu và trước thuế lớn hơn 1.000.000 VND.
   - Thuế VAT: 10% tính trên tổng tiền sau khi đã trừ tất cả các chiết khấu trên.
4. **Sinh mã nguồn Java:** Viết class `OrderCalculator` chứa phương thức `calculateTotal(Order order)`. Yêu cầu:
   - Sử dụng kiểu dữ liệu `BigDecimal` với chế độ làm tròn `RoundingMode.HALF_UP` cho toàn bộ các phép tính tiền tệ để đảm bảo độ chính xác tuyệt đối.
   - Code rõ ràng, sạch sẽ, có comment giải thích rõ tương ứng với từng bước trong logic CoT đã phân tích.

[Output Format]
- Phần 1: Phân tích logic và thứ tự ưu tiên tính toán hợp lệ.
- Phần 2: Công thức tính toán chi tiết từng bước.
- Phần 3: Bảng dry-run chi tiết với số liệu ca kiểm thử.
- Phần 4: Mã nguồn Java Class `OrderCalculator` hoàn chỉnh.
```

---

## 3. Minh chứng chạy thực tế (Text log phản hồi của AI)

### PHẦN 1: PHÂN TÍCH LOGIC VÀ THỨ TỰ ƯU TIÊN TÍNH TOÁN HỢP LỆ

Theo nguyên tắc kế toán tài chính và luật thuế thương mại điện tử, thứ tự áp dụng các khoản giảm giá, thuế và phí vận chuyển phải tuân theo quy trình từ "gần sản phẩm nhất" đến "toàn đơn hàng" và cuối cùng là các khoản phí dịch vụ, thuế phát sinh.
Thứ tự ưu tiên hợp lệ được xác định như sau:

1. **Bước 1: Tính tổng tiền gốc (Original Subtotal)**
2. **Bước 2: Áp dụng Giảm giá trực tiếp trên từng sản phẩm (Product Discount)**
   - *Lý do:* Đây là giảm giá tại cấp độ mặt hàng (Item-level), quyết định giá bán thực tế (Selling Price) của sản phẩm trước khi áp dụng các chương trình quy mô đơn hàng.
3. **Bước 3: Áp dụng Mã giảm giá đơn hàng (Coupon Discount)**
   - *Lý do:* Coupon áp dụng trên tổng giá trị hàng hóa thực tế bán ra (sau khi đã trừ giảm giá sản phẩm).
4. **Bước 4: Áp dụng Chiết khấu thành viên (Loyalty Discount)**
   - *Lý do:* Chiết khấu thành viên được tính trên số tiền khách hàng thực tế phải trả cho hàng hóa sau khi đã áp dụng coupon chung.
5. **Bước 5: Áp dụng Phí vận chuyển (Shipping Fee)**
   - *Lý do:* Phí vận chuyển cần được xác định tại đây vì điều kiện miễn phí vận chuyển phụ thuộc vào "tổng tiền đơn hàng sau chiết khấu và trước thuế".
6. **Bước 6: Tính Thuế VAT (10%)**
   - *Lý do:* Thuế VAT đánh trên giá trị gia tăng của hàng hóa thực tế tiêu dùng (sau chiết khấu). Thuế VAT không đánh lên phí vận chuyển (phí vận chuyển do đơn vị vận chuyển xuất hóa đơn riêng hoặc đã bao gồm VAT dịch vụ riêng).
7. **Bước 7: Tính Tổng tiền thanh toán cuối cùng (Final Total)**
   - *Công thức:* `Tổng tiền cuối cùng = Tổng tiền sau chiết khấu + Thuế VAT + Phí vận chuyển`.

---

### PHẦN 2: CÔNG THỨC TÍNH CHI TIẾT TỪNG BƯỚC

Đặt các biến như sau:
- $P_i$: Giá gốc sản phẩm $i$, $Q_i$: Số lượng sản phẩm $i$, $D_i$: Tỷ lệ giảm giá sản phẩm $i$.
- $Subtotal_{Origin}$: Tổng tiền gốc.
- $Subtotal_{ProductDiscount}$: Tổng tiền sau giảm giá sản phẩm.
- $Discount_{Coupon}$: Giá trị giảm của coupon.
- $Subtotal_{Coupon}$: Tổng tiền sau coupon.
- $Discount_{Loyalty}$: Giá trị giảm loyalty.
- $Subtotal_{Loyalty}$: Tổng tiền sau chiết khấu thành viên (đây là mốc xét duyệt miễn phí vận chuyển).
- $Fee_{Ship}$: Phí vận chuyển thực tế.
- $Tax_{VAT}$: Thuế VAT.
- $Total_{Final}$: Tổng tiền thanh toán cuối cùng.

**Các công thức:**
1. $Subtotal_{Origin} = \sum (P_i \times Q_i)$
2. $Subtotal_{ProductDiscount} = \sum [P_i \times (1 - D_i) \times Q_i]$
3. Nếu $Subtotal_{ProductDiscount} \ge 500,000$: $Subtotal_{Coupon} = Subtotal_{ProductDiscount} - Discount_{Coupon}$. Ngược lại: $Subtotal_{Coupon} = Subtotal_{ProductDiscount}$.
4. $Subtotal_{Loyalty} = Subtotal_{Coupon} \times (1 - Tỷ\_lệ\_Loyalty)$
5. Xác định Phí vận chuyển:
   - Nếu Khoảng cách $D \le 10$ km: $Fee_{Ship\_Base} = 30,000$.
   - Nếu $D > 10$ km: $Fee_{Ship\_Base} = 30,000 + (D - 10) \times 5,000$.
   - Nếu $Subtotal_{Loyalty} > 1,000,000$ VND: $Fee_{Ship} = 0$. Ngược lại: $Fee_{Ship} = Fee_{Ship\_Base}$.
6. $Tax_{VAT} = Subtotal_{Loyalty} \times 10\%$
7. $Total_{Final} = Subtotal_{Loyalty} + Tax_{VAT} + Fee_{Ship}$

---

### PHẦN 3: DRY-RUN CHI TIẾT VỚI CA KIỂM THỬ CỤ THỂ

**Thông tin ca kiểm thử:**
- Sản phẩm A: Giá gốc 400,000 VND, số lượng = 2, giảm giá trực tiếp = 10%.
- Sản phẩm B: Giá gốc 300,000 VND, số lượng = 1, giảm giá trực tiếp = 0%.
- Mã coupon: Giảm 50,000 VND (yêu cầu đơn hàng tối thiểu sau giảm giá sản phẩm đạt 500,000 VND).
- Khách hàng hạng Gold: Giảm 5%.
- Địa chỉ nhận hàng cách kho: 12 km.

#### Các bước tính toán cụ thể:

| Bước | Phép tính | Kết quả (VND) | Giải thích |
| :--- | :--- | :--- | :--- |
| **1. Tổng tiền gốc** | $(400,000 \times 2) + (300,000 \times 1)$ | **1,100,000** | Tổng tiền gốc của 2 sản phẩm A và 1 sản phẩm B. |
| **2. Giảm giá sản phẩm** | Sản phẩm A: $400,000 \times 2 \times 90\% = 720,000$<br>Sản phẩm B: $300,000 \times 1 \times 100\% = 300,000$<br>Tổng: $720,000 + 300,000$ | **1,020,000** | Áp dụng giảm giá 10% trực tiếp cho sản phẩm A. |
| **3. Giảm giá Coupon** | Kiểm tra điều kiện: $1,020,000 \ge 500,000$ (Thỏa mãn)<br>Tổng sau coupon: $1,020,000 - 50,000$ | **970,000** | Áp dụng trừ trực tiếp 50,000 VND từ coupon. |
| **4. Chiết khấu Loyalty** | Giảm giá Gold (5%): $970,000 \times 5\% = 48,500$<br>Tổng sau Loyalty: $970,000 - 48,500$ | **921,500** | Giảm 5% tính trên số tiền 970,000 VND sau coupon. |
| **5. Phí vận chuyển** | Khoảng cách 12km > 10km.<br>Phí cơ bản: $30,000 + (12 - 10) \times 5,000 = 40,000$.<br>Kiểm tra điều kiện miễn phí vận chuyển:<br>Tổng sau chiết khấu trước thuế: $921,500 \le 1,000,000$ (Không được miễn phí). | **40,000** | Phí vận chuyển thực tế là 40,000 VND. |
| **6. Thuế VAT (10%)** | $921,500 \times 10\%$ | **92,150** | Thuế VAT 10% tính trên tổng tiền sau chiết khấu. |
| **7. Tổng thanh toán** | $921,500 + 92,150 + 40,000$ | **1,053,650** | Tổng số tiền khách hàng phải trả cuối cùng. |

---

### PHẦN 4: MÃ NGUỒN JAVA CLASS `ORDERCALCULATOR`

Dưới đây là mã nguồn Java Spring Boot sử dụng `BigDecimal` để đảm bảo độ chính xác tuyệt đối.

#### 1. Định nghĩa các Model phụ trợ
```java
package com.speedycart.model;

import lombok.Builder;
import lombok.Data;
import java.math.BigDecimal;
import java.util.List;

@Data
@Builder
public class Order {
    private List<OrderItem> items;
    private Coupon coupon;
    private CustomerTier customerTier;
    private double shippingDistanceKm;
}

@Data
@Builder
public class OrderItem {
    private String productId;
    private BigDecimal originalPrice;
    private BigDecimal discountRate; // Ví dụ: 0.10 cho 10%
    private int quantity;
}

@Data
@Builder
public class Coupon {
    private String code;
    private BigDecimal discountAmount;
    private BigDecimal minOrderAmount;
}

public enum CustomerTier {
    NORMAL, GOLD, PLATINUM
}
```

#### 2. Class thực hiện tính toán `OrderCalculator`
```java
package com.speedycart.service;

import com.speedycart.model.Order;
import com.speedycart.model.OrderItem;
import com.speedycart.model.CustomerTier;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;
import java.math.RoundingMode;

@Service
public class OrderCalculator {

    private static final BigDecimal HUNDRED = new BigDecimal("100");
    private static final BigDecimal VAT_RATE = new BigDecimal("0.10"); // 10% VAT
    private static final BigDecimal GOLD_DISCOUNT_RATE = new BigDecimal("0.05"); // 5%
    private static final BigDecimal PLATINUM_DISCOUNT_RATE = new BigDecimal("0.10"); // 10%
    
    private static final BigDecimal SHIPPING_BASE_DISTANCE = new BigDecimal("10");
    private static final BigDecimal SHIPPING_BASE_FEE = new BigDecimal("30000");
    private static final BigDecimal SHIPPING_EXTRA_FEE_PER_KM = new BigDecimal("5000");
    private static final BigDecimal FREE_SHIPPING_THRESHOLD = new BigDecimal("1000000");

    public BigDecimal calculateTotal(Order order) {
        if (order == null || order.getItems() == null || order.getItems().isEmpty()) {
            return BigDecimal.ZERO;
        }

        // Bước 1 & 2: Tính tổng tiền hàng sau khi đã áp dụng giảm giá trực tiếp từng sản phẩm
        BigDecimal subtotalAfterProductDiscount = BigDecimal.ZERO;

        for (OrderItem item : order.getItems()) {
            BigDecimal originalPrice = item.getOriginalPrice();
            BigDecimal discountRate = item.getDiscountRate() != null ? item.getDiscountRate() : BigDecimal.ZERO;
            BigDecimal quantity = new BigDecimal(item.getQuantity());

            // Giá bán thực tế của 1 sản phẩm = Giá gốc * (1 - Tỷ lệ giảm giá)
            BigDecimal itemSellingPrice = originalPrice.multiply(BigDecimal.ONE.subtract(discountRate));
            // Tổng tiền của sản phẩm này = Giá bán thực tế * Số lượng
            BigDecimal itemTotal = itemSellingPrice.multiply(quantity);

            subtotalAfterProductDiscount = subtotalAfterProductDiscount.add(itemTotal);
        }
        subtotalAfterProductDiscount = subtotalAfterProductDiscount.setScale(2, RoundingMode.HALF_UP);

        // Bước 3: Áp dụng Mã giảm giá (Coupon Code)
        BigDecimal subtotalAfterCoupon = subtotalAfterProductDiscount;
        if (order.getCoupon() != null) {
            BigDecimal minOrderAmount = order.getCoupon().getMinOrderAmount();
            // Điều kiện áp dụng coupon
            if (subtotalAfterProductDiscount.compareTo(minOrderAmount) >= 0) {
                BigDecimal couponDiscount = order.getCoupon().getDiscountAmount();
                subtotalAfterCoupon = subtotalAfterProductDiscount.subtract(couponDiscount);
                if (subtotalAfterCoupon.compareTo(BigDecimal.ZERO) < 0) {
                    subtotalAfterCoupon = BigDecimal.ZERO;
                }
            }
        }
        subtotalAfterCoupon = subtotalAfterCoupon.setScale(2, RoundingMode.HALF_UP);

        // Bước 4: Áp dụng Chiết khấu thành viên (Loyalty Discount)
        BigDecimal loyaltyDiscountRate = BigDecimal.ZERO;
        if (order.getCustomerTier() == CustomerTier.GOLD) {
            loyaltyDiscountRate = GOLD_DISCOUNT_RATE;
        } else if (order.getCustomerTier() == CustomerTier.PLATINUM) {
            loyaltyDiscountRate = PLATINUM_DISCOUNT_RATE;
        }

        BigDecimal loyaltyDiscountAmount = subtotalAfterCoupon.multiply(loyaltyDiscountRate);
        BigDecimal subtotalAfterLoyalty = subtotalAfterCoupon.subtract(loyaltyDiscountAmount)
                .setScale(2, RoundingMode.HALF_UP);

        // Bước 5: Tính Phí vận chuyển (Shipping Fee)
        BigDecimal shippingFee = BigDecimal.ZERO;
        if (subtotalAfterLoyalty.compareTo(FREE_SHIPPING_THRESHOLD) <= 0) {
            BigDecimal distance = new BigDecimal(order.getShippingDistanceKm());
            if (distance.compareTo(SHIPPING_BASE_DISTANCE) <= 0) {
                shippingFee = SHIPPING_BASE_FEE;
            } else {
                BigDecimal extraDistance = distance.subtract(SHIPPING_BASE_DISTANCE)
                        .setScale(0, RoundingMode.CEILING); // Làm tròn lên phần km lẻ
                BigDecimal extraFee = extraDistance.multiply(SHIPPING_EXTRA_FEE_PER_KM);
                shippingFee = SHIPPING_BASE_FEE.add(extraFee);
            }
        }
        shippingFee = shippingFee.setScale(2, RoundingMode.HALF_UP);

        // Bước 6: Tính Thuế VAT (10% tính trên tổng tiền sau khi đã trừ tất cả các chiết khấu trên)
        BigDecimal vatAmount = subtotalAfterLoyalty.multiply(VAT_RATE)
                .setScale(2, RoundingMode.HALF_UP);

        // Bước 7: Tính tổng thanh toán cuối cùng
        BigDecimal finalTotal = subtotalAfterLoyalty.add(vatAmount).add(shippingFee)
                .setScale(0, RoundingMode.HALF_UP); // Làm tròn thành số nguyên đồng tiền Việt Nam

        return finalTotal;
    }
}
```
