# 도메인 서비스

## 여러 애그리거트가 필요한 기능

도메인 영역의 코드를 작성하다 보면, 한 애그리거트로 기능을 구현할 수 없을 때가 있다. <br>
대표적인 예가 결제 금액 계산 로직이다. 실제 결제 금액을 계산할 때는 다음과 같은 내용이 필요하다.

- 상품 애그리거트: 구매하는 상품의 가격이 필요하다. 또한 상품에 따라 배송비가 추가되기도 한다.
- 주문 애그리거트: 상품별로 구매 개수가 필요하다.
- 할인 쿠폰 애그리거트: 쿠폰별로 지정한 할인 금액이나 비율에 따라 주문 총 금액을 할인한다. 쿠폰을 조건
  에 따라 중복 사용할 수 있다거나 지정한 카테고리의 상품에만 적용할 수 있다는 제약 조건이 있다면 할인 계산
  이 복잡해진다.
- 회원 애그리거트: 회원 등급에 따라 추가 할인이 가능하다.

이 상황에서 실제 결제 금액을 계산해야 하는 주체는 어떤 애그리거트일까? <br>
생각해 볼 수 있는 방법은 주문 애그리거트가 필요한 데이터를 모두 가지도록 한 뒤 할인 금액 계산 책임을 주문 애그리거트에 할당하는 것이다. <br>
여기서 결제 금액 계산 로직이 주문 애그리거트의 책임이 맞을까? <br>
**이렇게 한 애그리거트에 넣기 애매한 도메인 기능을 억지로 특정 애그리거트에 구현하면 안 된다.** <br>
억지로 구현하면 애그리거트는 자신의 책임 범위를 넘어서는 기능을 구현하기 때문에 코드가 길어지고 외부에 대한 의존이 높아지게 되며 코드를 복잡하게 만들어 수정을 어렵게 만드는 요인이 된다. <br>
이런 문제를 해소하는 가장 쉬운 방법이 하나 있는데 그것이 바로 도메인 기능을 별도 서비스로 구현하는 것이다.

## 도메인 서비스

도메인 서비스는 도메인 영역에 위치한 도메인 로직을 표현할 때 사용한다. 주로 다음 상황에서 도메인 서비스를 사용한다.

- 계산 로직: 여러 애그리거트가 필요한 계산 로직이나, 한 애그리거트에 넣기에는 다소 복잡한 계산 로직
- 외부 시스템 연동이 필요한 도메인 로직: 구현하기 위해 타 시스템을 사용해야 하는 도메인 로직

<img width="669" alt="Image" src="https://github.com/user-attachments/assets/bfc9db06-59fb-412b-b7b4-327d5f255703" />

```java
public class DiscountCalculationService {
    
    public Money calculateDiscountAmounts(
            List<OrderLine> orderLines,
            List<Coupon> coupons,
            MemberGrade grade) {
        Money couponDiscount =
                coupons.streamO
                        .map(coupon -> calculateDiscount(coupon))
                        .reduce(Money(0), (vl, v2) -> vl.add(v2));
        Money membershipDiscount =
                calculateDiscount(orderer.getMember().getGrade());
        return couponDiscount.add(membershipDiscount);
    }

    private Money calculateDiscount(Coupon coupon) {
        
    }

    private Money calculateDiscount(MemberGrade grade) {
        
    }
}
```

```java
// 할인 계산 서비스를 사용하는 주체는 애그리거트가 될 수도 있고 응용 서비스가 될 수도 있다.
// DiscountCalculationService< 다음과 같이 애그리거트의 결제 금액 계산 기능에 전달하면 사용 주체는 애그리거트가 된다.
public class Order {
    public void calculateAmounts(
            DiscountCalculationService disCalSvc, MemberGrade grade) {
        Money totalAmounts = getTotalAmounts();
        Money discountAmounts =
                disCalSvc.calculateDiscountAmounts(this .orderLines, this .coupons, grade);
        this.paymentAmounts = totalAmounts.minus(discountAmounts);
    }
}
```

```java
// 애그리거트 객체에 도메인 서비스를 전달하는 것은 응용 서비스 책임이다.
public class OrderService {
    private DiscountCalculationService discountCalculationService;

    @Transactional
    public OrderNo placeOrder(OrderRequest orderRequest) {
        OrderNo orderNo = orderRepository.nextId();
        Order order = createOrder(orderNo, orderRequest);
        orderRepository.save(order);
        // 응용 서비스 실행 후 표현 영역에서 필요한 값 리턴
        return orderNo;
    }
    
    private Order createOrder(OrderNo orderNo, OrderRequest orderReq) {
        Member member = findMember(orderReq.getOrdererld());
        Order order = new Order(orderNo, orderReq.getOrderLines(),
                orderReq.getCoupons(), createOrderer(member),
                orderReq.getShippingInfo());
        order.calculateAmounts(this.discountCalculationService,
                member.getGrade());
        return order;
    }
}
```

### 외부 시스템 연동과 도메인 서비스

외부 시스템이나 타 도메인과의 연동 기능도 도메인 서비스가 될 수 있다.

```java
public interface SurveyPermissionChecker {
    boolean hasUserCreationPermission(String userId);
}
```

```java
public class CreateSurveyService {
    // SurveyPermissionChecker 인터페이스를 구현한 클래스는 인프라스트럭처 영역에 위치해 연동을 포함한 권한 검사 기능을 구현한다.
    private SurveyPermissionChecker permissionChecker;

    public Long createSurvey(CreateSurveyRequest req) {
        validate(req);
        // 도메인 서비스를 이용해서 외부 시스템 연동을 표현
        if (!permissionChecker.hasUserCreationPermission(req.getRequestorIdO)) {
            throw new NoPermissionExceptionO;
        }
    }
}
```










