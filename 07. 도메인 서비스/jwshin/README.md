# 💡 7.1 여러 애그리거트가 필요한 기능

- 도메인 영역의 코드를 작성하다 보면, 한 애그리거트로 기능을 구현할 수 없을 때가 있다.
- 예를 들어, 결제 금액 계산 로직을 구현해야 한다고 하면 주문, 상품, 할인, 회원 등 여러 애그리거트를 조합해서 계산해야 한다.
- 만약, 특정 애그리거트에 필요한 모든 데이터를 가지게 한 뒤 모든 책임을 할당하면 어떻게 될까?
  - 다른 애그리거트의 변경사항에 엉뚱한 애그리거트를 수정해야 하는 상황이 발생할 수 있다.
  - 한 애그리거트에 넣기 애매한 도메인 기능을 억지로 특정 애그리거트에 구현하면 안 된다!
  - 억지로 구현하면 애그리거트는 자신의 책임 범위를 넘어서는 기능을 구현하기 때문에 코드가 길어지고 외부에 대한 의존이 높아지게 되며 코드를 복잡하게 만들어 수정을 어렵게 만드는 요인이 된다.
- 이런 문제를 해소하려면 도메인 기능을 별도의 서비스로 구현해야 한다. -> 도메인 서비스

# 💡 7.2 도메인 서비스

- 도메인 서비스는 도메인 영역에 위치한 도메인 로직을 표현할 때 사용한다. 주로 다음 상황에서 도메인 서비스를 사용한다.
  - 계산 로직: 여러 애그리거트가 필요한 계산 로직이나, 한 애그리거트에 넣기에는 다소 복잡한 계산 로직
  - 외부 시스템 연동이 필요한 도메인 로직: 구현하기 위해 타 시스템을 사용해야 하는 도메인 로직
 
### 7.2.1 계산 로직과 도메인 서비스

- 응용 영역의 서비스가 응용 로직을 다룬다면 도메인 서비스는 도메인 로직을 다룬다.
- 도메인 영역의 애그리거트나 밸류와 같은 구성요소와 도메인 서비스를 비교할 때 다른 점은 도메인 서비스는 **상태 없이 로직만 구현**한다는 점이다.
- 도메인 서비스를 구현하는 데 필요한 상태는 다른 방법으로 전달받는다.

```java
public class DiscountCalculationService {
    // 할인 금액 계산 로직을 위한 도메인 서비스는 다음과 같이 도메인의 의미가 드러나는 용어를 타입과 메서드 이름으로 갖는다.
    // 파라미터로는 애그리거트를 받는거 같음!!
    public Money calculateDiscountAmounts(
            List<OrderLine> orderLines,
            List<Coupon> coupons,
            MemberGrade grade) {
        Money couponDiscount = coupons.streamO
                                        .map(coupon -> calculateDiscount(coupon))
                                        .reduce(Money(0), (vl, v2) -> vl.add(v2));
        Money membershipDiscount = calculateDiscount(orderer.getMember().getGrade());
        return couponDiscount.add(membershipDiscount);
    }

    private Money calculateDiscount(Coupon coupon) {
        
    }

    private Money calculateDiscount(MemberGrade grade) {
        
    }
}
```

- 할인 계산 서비스를 사용하는 주체는 애그리거트가 될 수도 있고 응용 서비스가 될 수도 있다.
- DiscountCalculationService를 다음과 같이 애그리거트의 결제 금액 계산 기능에 전달하면 사용 주체는 애그리거트가 된다.

```java
public class Order {
    // 애그리거트 객체에 도메인 서비스를 전달하는 것은 응용 서비스 책임이다.
    // 첫 번째 파라미터에 도메인 서비스를 전달하고, Order 애그리거트에서 사용한다.
    public void calculateAmounts(DiscountCalculationService disCalSvc, MemberGrade grade) {
        Money totalAmounts = getTotalAmounts();
        Money discountAmounts = disCalSvc.calculateDiscountAmounts(this.orderLines, this.coupons, grade);
        this.paymentAmounts = totalAmounts.minus(discountAmounts);
    }
}
```

```java
public class OrderService {
    private DiscountCalculationService discountCalculationService; // 응용 서비스에서 도메인 서비스를 필드로 가지고 있는다.

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
        // 응용 서비스에서 도메인 서비스를 전달한다!
        order.calculateAmounts(this.discountCalculationService, member.getGrade());
        return order;
    }
}
```

### 반대의 경우

- 애그리거트 메서드를 실행할 때 도메인 서비스를 인자로 전달하지 않고 반대로 도메인 서비스의 기능을 실행할 때 애그리거트를 전달하기도 한다.
- 대표적인 예로 계좌 이체가 있다.
- 계좌 이체에는 두 계좌 애그리거트가 관여하는데 한 애그리거트는 금액을 출금하고 한 애그리거트는 금액을 입금한다.

```java
public class TransferService {
    public void transfer(Account fromAcc, Account toAcc, Money anunints) {
        fromAcc.withdraw(amounts);
        toAcc.credit(amounts);
    }
}
```

- 도메인 서비스는 도메인 로직을 수행하지 응용 로직을 수행하진 않는다. 트랜잭션 처리와 같은 로직은 응용 로직이므로 도메인 서비스가 아닌 응용 서비스에서 처리해야 한다.

### 📌 기준 잡기, 응용 서비스 vs 도메인 서비스

- 특정 기능이 응용 서비스인지, 도메인 서비스인지 기준을 잡기 애매할 때는 다음 두 가지를 확인해보자!
  - 해당 로직이 애그리거트의 상태를 변경하는지?
  - 애그리거트 상태 값을 계산하는지?
- 예를 들어 계좌 이체 로직은 계좌 애그리거트의 상태를 변경한다. -> 도메인 서비스
- 결제 금액 로직은 주문 애그리거트의 주문 금액을 계산한다. -> 도메인 서비스
- 위 두 로직은 도메인 로직이면서 한 애그리거트에 넣기에 적합하지 않으므로 도메인 서비스로 구현한다.

### 7.2.2 외부 시스템 연동과 도메인 서비스

- 외부 시스템이나 타 도메인과의 연동 기능도 도메인 서비스가 될 수 있다.
- 예를 들어, 다른 시스템의 정보를 가져와서 validation해야 하는 상황을 생각해보자.
- 시스템 간 연동은 HTTP API 호출로 이루어질 수 있지만 도메인 입장에서는 권한을 가졌는지 확인하는 도메인 로직으로 볼 수 있다.

```java
// 도메인 로직 관점에서 인터페이스를 작성한다! (타 시스템과 연동한다는 관점으로 작성하지 않았다.)
// SurveyPermissionChecker 인터페이스를 구현한 클래스는 인프라스트럭처 영역에 위치해 연동을 포함한 권한 검사 기능을 구현한다. (DIP)
public interface SurveyPermissionChecker {
    boolean hasUserCreationPermission(String userId);
}
```

```java
public class CreateSurveyService {
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

### 7.2.3 도메인 서비스의 패키지 위치

- 도메인 서비스는 도메인 로직을 표현하므로 도메인 서비스의 위치는 다른 도메인 구성요소와 동일한 패키지에 위치한다.

![image](https://github.com/user-attachments/assets/7bd7c2f8-9dad-4fb7-8dcc-7565fb78da86)

### 7.2.4 도메인 서비스의 인터페이스와 클래스

- 도메인 서비스의 로직이 고정되어 있지 않은 경우 도메인 서비스 자체를 인터페이스로 구현하고 이를 구현한 클래스를 둘 수도 있다.
- 특히 도메인 로직을 외부 시스템이나 별도 엔진을 이용해서 구현할 때 인터페이스와 클래스를 분리하게 된다.

![image](https://github.com/user-attachments/assets/0d690829-1018-4647-b2c4-9c579417e1e1)

- 위 그림과 같이 도메인 서비스의 구현이 특정 구현 기술에 의존하거나 외부 시스템의 API를 실행한다면 도메인 영역의 도메인 서비스는 인터페이스로 추상화해야 한다. (DIP)
- 이를 통해 도메인 영역이 특정 구현에 종속되는 것을 방지할 수 있고 도메인 영역에 대한 테스트가 쉬워진다.

