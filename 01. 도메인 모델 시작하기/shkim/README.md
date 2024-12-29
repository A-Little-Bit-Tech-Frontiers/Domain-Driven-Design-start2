# 도메인 모델 시작하기

## 도메인이란 ?

한 도메인은 다시 하위 도메인으로 나눌 수 있다. <br>
예를 들어 온라인 서점 도메인은 아래와 같이 몇 개의 하위 도메인으로 나눌 수 있다.

<img width="409" alt="스크린샷 2024-12-29 오후 2 56 18" src="https://github.com/user-attachments/assets/742aac64-475d-4612-a411-3c8a9e41980f" />

## 도메인 전문가와 개발자 간 지식 공유

개발자는 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다. <br>
이 과정에서 요구사항은 첫 단추와 같다. 첫 단추를 잘못 끼우면 모든 단추가 잘못 끼워지듯이 요구사항을 올바르게 이해하지 못하면 요구하지 않은 엉뚱한 기능을 만들게 된다. <br>
도메인 전문가 만큼은 아니겠지만 이해관계자와 개발자도 도메인 지식을 갖춰야 한다. <br>
제품 개발과 관련된 도메인 전문가, 관계자, 개발자가 같은 지식을 공유하고 직접 소통할수록 도메인 전문가가 원하는 제품을 만들 가능성이 높아진다.

## 도메인 모델

주문 도메인을 생각해 보자. <br>
온라인 쇼핑몰에서 주문을 하려면 상품을 몇 개 살지 선택하고 배송지를 입력한다. 선택한 상품 가격을 이용해서 총 지불 금액을 계산하고, 금액 지불을 위한 결제 수단을 선택한다. <br>
주문한 뒤에도 배송 전이면 배송지 주소를 변경하거나 주문을 취소할 수 있다. 이를 위한 주문 모델을 객체 모델로 구성하면 아래와 같이 만들 수 있다.

<img width="633" alt="스크린샷 2024-12-29 오후 2 59 25" src="https://github.com/user-attachments/assets/a06304c8-09e3-4b83-8df7-894e4b1c14de" />

## 도메인 모델 패턴

일반적인 애플리케이션의 아키텍처는 네 개의 영역으로 구성된다.

| 영역      | 설명                                                           |
|---------|--------------------------------------------------------------|
| 표현      | 사용자의 요청을 처리하고 사용자에게 정보를 보여준다.                                |
| 응용      | 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다. |
| 도메인     | 시스템이 제공할 도메인 규칙을 구현한다.  |
| 인프라스트럭처 |데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.|

도메인 계층은 도메인의 핵심 규칙을 구현한다. <br>
주문 도메인의 경우 ‘출고 전에 배송지를 변경 할 수 있다’라는 규칙과 ‘주문 취소는 배송 전에만 할 수 있다’라는 규칙을 구현한 코드가 도메인 계층에 위치하게 된다. <br>
이런 도메인 규칙을 객체 지향 기법으로 구현하는 패턴이 도메인 모델 패턴이다.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeableO) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippingInfo = newShippingInfo;
    }
}
```

```java
public enum OrderState {
    PAYMENT_WAITING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    PREPARING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    SHIPPED, DELIVERING, DELIVERY_COMPLETED;
    
    public boolean isShippingChangeable() {
        return false;
    }
}
```

OrderState는 주문 대기 중이거나 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙을 구현하고 있다. <br>
실제 배송지 정보를 변경하는 Order 클래스의 changeShippingInfo() 메서드는 OrderState isShippingChangeable() 메서드를 이용해서 변경 가능한 경우에만 배송지를 변경한다.

## 도메인 모델 도출

도메인을 모델링할 때 기본이 되는 작업은 모델을 구성하는 핵심 구성요소, 규칙, 기능을 찾는 것이다. <br>
이 과정은 요구사항에서 출발한다.

- 최소 한 종류 이상의 상품을 주문해야 한다.
- 한 상품을 한 개 이상 주문할 수 있다.
- 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다. 
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다. 
- 주문할 때 배송지 정보를 반드시 지정해야 한다. 
- 배송지 정보는 받는 사람 이름, 전화번호 주소로 구성된다.
- 출고를 하면 배송자를 변경할 수 없다.
- 출고 전에 주문을 취소할 수 있다.
- 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

<br>

다음 요구사항에 따르면 주문 항목을 표현하는 OrderLine은 적어도 주문할 상품, 상품의 가격, 구매 개수를 포함해야 한다. <br>
추가로 각 구매 항목의 구매 가격도 제공해야 한다. 이를 구현한 OrderLine 다음과 같다.

- 한 상품을 한 개 이상 주문할 수 있다.
- 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.

```java
public class OrderLine {
    private Product product;
    private int price;
    private int quantity;
    private int amounts;
    
    public OrderLine(Product product, int price, int quantity) {
        this.product = product;
        this.price = price;
        this.quantity = quantity;
        this.amounts = calculateAmounts();
    }
    
    private int calculateAmounts() {
        return price * quantity;
    }
    
    public int getAmounts() { ... }
}
```


한 종류 이상의 상품을 주문할 수 있으므로 Order는 최소 한 개 이상의 OrderLine을 포함해야 한다. <br>
또한 총 주문 금액은 OrderLine에서 구할 수 있다.

```java
public class Order {
    private List<OrderLine> orderLines;
    private Monty totalAmounts;
    
    public Order(List<OrderLine> orderLines) {
        setOrderLines(orderLines);
    }
    
    private void setOrderLines(List<OrderLine> orderLines) {
        verifyAtLeastOneOrMoreOrderLines(orderLines);
        this.orderLines = orderLines;
        calculateTotalAmounts();
    }
    
    private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
        if (orderLines == null || orderLines.isEmpty()) {
            throw new IUegalArgumentException("no OrderLine");
        }
    }
    
    private void calculateTotalAmounts() {
        int sum = orderLines.stream()
                .mapToInt(x -> x.getAmounts())
                .sum();
        this.totalAmounts = new Money(sum);
    }
}
```

## 엔티티와 밸류

도출한 모델은 엔티티와 밸류로 구분할 수 있다.

### 엔티티

엔티티의 가장 큰 특징은 식별자를 가진다는 것이다. 식별자는 엔티티 객체마다 고유해서 각 엔티티는 서로 다른 식별자를 갖는다. <br>
주문에 해당하는 클래스가 Order이므로 Order가 엔티티가 되며 주문번호를 속성으로 갖게 된다.

### 밸류 타입

ShippingInfo는 받는 사람과 주소에 대한 데이터를 갖고 있다.

```java
public class ShippingInfo {
    // 받는 사람
    private String receiverName;
    private String receiverPhoneNumber;
    
    // 주소
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipcode;
} 
```

밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다. <br>
예를 들어 받는 사람을 위한 밸류 타입인 Receiver를 다음과 같이 작성할 수 있다.


```java
public class Receiver {
    private String name;
    private String phoneNumber;

    public Receiver(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber;
    }
    
    public String getName() {
        return name;
    }
    
    public String getPhoneNumber() {
        return phoneNumber;
    }
} 
```


<img width="624" alt="스크린샷 2024-12-29 오후 3 14 28" src="https://github.com/user-attachments/assets/2d55237d-5113-4be9-b904-104bc94838ec" />

## 도메인 용어와 유비쿼터스 언어

도메인에서 사용하는 용어를 코드에 반영하지 않으면 그 코드는 개발자에게 코드의 의미를 해석해야 하는 부담을 준다.

```java
public OrderState {
    STEP1, STEP2, STEP3, STEP4, STEP5, STEP6
}
```

STEP1, STEP2, STEP3이 아닌 PAYMENT_WAITING, PREPARING, SHIPPED 같이 도메인에서 사용하는 용어를 최대한 코드에 반영하면 바로 이해할 수 있다. <br>
코드를 도메인 용어로 해석하거나 도메인 용어를 코드로 해석하는 과정이 줄어든다. <br>
이는 코드의 가독성을 높여서 코드를 분석하고 이해하는 시간을 줄여준다.






