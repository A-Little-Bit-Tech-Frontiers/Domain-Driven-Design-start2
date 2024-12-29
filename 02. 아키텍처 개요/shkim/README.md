# 아키텍처 개요

## 네 개의 영역

표현, 응용, 도메인, 인프라스트럭처는 아키텍처를 설계할 때 출현하는 전형적인 네 가지 영역이다. <br>
웹 애플리케이션의 표현 영역은 HTTP 요청을 응용 영역이 필요로 하는 형식으로 변환해서 응용 영역에 전달하고 응용 영역의 응답을 HTTP 응답으로 변환하여 전송한다. <br>
표현 영역을 통해 사용자의 요청을 전달받는 응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현한다. <br>
응용 서비스는 로직을 직접 수행하기보다는 도메인 모델에 로직 수행을 위임한다. <br>
도메인 영역은 도메인 모델을 구현한다. <br>
도메인 모델은 도메인의 핵심 로직을 구현한다. 예를 들어 주문 도메인은 ‘배송지 변경’, ‘결제 완료’, ‘주문 총액 계산’과 같은 핵심 로직을 도메인 모델에서 구현한다. <br>
인프라스트럭처 영역은 구현 기술에 대한 것을 다룬다. 이 영역은 RDBMS 연동을 처리하고, 메시징 큐에 메시지를 전송하거나 수신하는 기능을 구현한다.

## 계층 구조 아키텍처

<img width="280" alt="스크린샷 2024-12-29 오후 4 18 24" src="https://github.com/user-attachments/assets/9ac3cabd-96f5-41cb-8d73-0147dc7c68b1" />

할인 금액을 계산하기 위해 Drools라는 룰 엔진을 사용해서 계산 로직을 수행하는 인프라스트럭처 영역의 코드를 예로 보자. <br>
응용 영역은 가격 계산을 위해 인프라스트럭처 영역의 Dro이sR니leEngine을 사용한다. 

<img width="618" alt="스크린샷 2024-12-29 오후 4 21 17" src="https://github.com/user-attachments/assets/aef7a38c-4cfa-44a5-b1bf-1bd610a972e0" />

인프라스트럭처에 의존하면 ‘테스트 어려움’과 ‘기능 확장의 어려움’이라는 두 가지 문제가 발생한다.

## DIP

고수준 모듈이 제대로 동작하려면 저수준 모듈을 사용해야 한다. <br>
그런데 고수준 모듈이 저수준 모듈을 사용하면 앞서 계층 구조 아키텍처에서 언급했던 두 가지 문제, 즉 구현 변경과 테스트가 어렵다는 문제가 발생한다. <br>
D止는 이 문제를 해결하기 위해 저수준 모듈이 고수준 모듈에 의존하도록 바꾼다.

```java
public interface RuleDiscounter {
    Money applyRules(Customer customer, List<OrderLine> orderLines);
}
```

```java
public class CalculateDiscountService {
    private RuleDiscounter ruleDiscounter;
    
    public CalculateDiscountService(RuleDiscounter ruleDiscounter) {
        this.ruleDiscounter = ruleDiscounter;
    }
    
    public Money calculateDiscount(List<OrderLine> orderLines, String customerId) {
        Customer customer = findCustomer(customerId);
        return ruleDiscounter.applyRules(customer, orderLines);
    }
}
```

CalculateDiscountService에는 Drools에 의존하는 코드가 없다. 단지 RuleDiscounter룰을 적용한다는 사실만 알뿐이다. <br>
룰 적용을 구현한 클래스는 RuleDiscounter 인터페이스를 상속받아 구현한다. <br>
DIP를 적용하면 저수준 모듈이 고수존 모듈에 의존하게 된다. 

<img width="475" alt="스크린샷 2024-12-29 오후 4 24 54" src="https://github.com/user-attachments/assets/b89cf69a-02d5-4e71-9894-297cd16e9fa5" />

## 도메인 영역의 주요 구성 요소

| 요소      | 설명                                                                                        |
|---------|-------------------------------------------------------------------------------------------|
| 엔티티     | 고유의 식별자를 갖는 객체로 자신의 라이프 사이클을 갖는다. 주문, 회원, 상품과 같이 도메인의 고유한 개념을 표현한다.                       |
| 밸류      | 고유의 식별자를 갖지 않는 객체로 주로 개념적으로 하나의 값을 표현할 떄 사용된다. 배송지 주소를 표현하기 위한 주소나 구매 금액을 위한 금액과 같은 타입이다. |
| 애그리거트   | 연관된 엔티티와 밸류 객체를 개념적으로 하나로 묶은 것이다.                                                         |
| 리포지터리   | 도메인 모델의 영속성을 처리한다.                                                                        |
| 도메인 서비스 | 특정 엔티티에 속하지 않은 도메인 로직을 제공한다. 여러 엔티티와 밸류를 필요로 하면 도메인 서비스에서 로직을 구현한다.                       |

### 엔티티와 밸류

도메인 모델의 엔티티는 단순히 데이터를 담고 있는 데이터 구조라기보다는 데이터와 함께 기능을 제공하는 객체이다. <br>
도메인 관점에서 기능을 구현하고 기능 구현을 캡슐화해서 데이터가 임의로 변경되는 것을 막는다. <br>
밸류는 불변으로 구현할 것을 권장하며, 이는 엔티티의 밸류 타입 데이터를 변경할 때는 객체 자체를 완전히 교체한다는 것을 의미한다.

### 애그리거트

도메인 모델은 개별 객체뿐만 아니라 상위 수준에서 모델을 볼 수 있어야 전체 모델의 관계와 개별 모델을 이해하는 데 도움이 된다. <br>
도메인 모델에서 전체 구조를 이해하는 데 도움이 되는 것이 바로 애그리거트다. <br>
애그리거트는 관련 객체를 하나로 묶은 군집이다. 애그리거트의 대표적인 예가 주문이다. 주문이라는 도메인 개념은 ‘주문’, ‘배송지 정보’, ‘주문자’, ‘주문 목록’, ‘총 결제 금액’의 하위 모델로 구성된다.

애그리거트는 군집에 속한 객체를 관리하는 루트 엔티티를 갖는다. <br>
루트 엔티티는 애그리거트에 속해 있는 엔티티와 밸류 객체를 이용해서 애그리거트가 구현해야 할 기능을 제공한다.

<img width="544" alt="스크린샷 2024-12-29 오후 4 34 00" src="https://github.com/user-attachments/assets/7a3fdd1b-e6cd-4b6c-b44d-ef66eaad1467" />

## 모듈 구성

<img width="531" alt="스크린샷 2024-12-29 오후 4 35 45" src="https://github.com/user-attachments/assets/da106036-f1fe-4867-b6db-2c88799fdd9a" />















