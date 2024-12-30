
## 1 도메인

<img width="306" alt="image" src="https://github.com/user-attachments/assets/3dd31f17-a74e-4bc5-bedd-72e0aee40f16" />

**도메인은 여러 하위 도메인으로 구성**된다. 위 예시를 바탕으로 설명하면, “온라인 서점”이란 도메인을 위와 같이 표현했다.  
`카탈로그`라는 하위 도메인은 고객에게 구매할 수 있는 상품 목록을 제공하고, `주문` 이란 하위 도메인은 고객의 주문을 처리하는 도메인이다.   
완전한 기능을 제공하기 위해서는 여러 하위 도메인간에 연동이 필요하다. 예를 들어, 고객이 물건을 구매하면 `주문`, `결제`, `배송` 도메인이 엮여야 한다.


## 2. 도메인 전문가와 개발자 간 지식 공유

도메인 전문가 만큼은 아니겠지만 이해관계자와 개발자도 도메인 지식을 갖춰야 한다. 제품 개발과 관련된 도메인 전문가, 관계자, 개발자가 같은 지식을 공유하고 **직접 소통할수록 도메인 전문가가 원하는 제품을 만들 가능성이 높아진다.**   
개발자는 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다. 이 과정에서 **요구사항은 첫 단추와 같다.** 첫 단추를 잘못 끼우면 모든 단추가 잘못 끼워지듯이 요구사항을 올바르게 이해하지 못하면 요구하지 않은 엉뚱한 기능을 만들게 된다. 
단추는 풀어서 다시 끼우기 쉽지만 작성한 코드는 그렇지 않다. **잘못 개발한 코드를 수정해서 올바르게 고치려면 많은 노력이 든다.**  


## 3. 도메인 모델

기본적인 도메인 모델은 특정 도메인을 개념적으로 표현한 것을 의미한다.

<img width="451" alt="image" src="https://github.com/user-attachments/assets/6f5d79bd-028d-4ab4-9851-317bb52ab939" />

위 이미지는 객체를 이용한 도메인 모델이다. 도메인을 이해하려면 도메인이 제공하는 기능과 도메인의 주요 데이터 구성을 파악해야 하는데, 이런 면에서 기능과 데이터를 함께 보여주는 객체 모델은 도메인을 모델링하기에 적합하다.  
도메인은 다수의 하위 도메인으로 구성된다. 각 하위 도메인이 다루는 영역은 서로 다르기 때문에 같은 용어라도 하위 도메인마다 의미가 달라질 수 있다.  
도메인에 따라 용어의 의미가 결정되므로, 여러 하위 도메인을 하나의 다이어그램에 모델링하면 안된다.  
(-> 주문 도메인이면, 주문 모델만 모델링하고, 카탈로그 도메인이면, 카탈로그 모델만 모델링)  


## 4. 도메인 모델 패턴

<img width="299" alt="image" src="https://github.com/user-attachments/assets/1dd2a2d6-b11c-4df3-97fc-a69033e07a7f" />

일반적인 애플리케이션의 아키텍처는 위와 같이 네 개의 영역으로 구성된다. 

- 표현 계층: 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여기서 사용자는 소프트웨어를 사용하는 사람뿐만 아니라 외부 시스템일 수도 있다.
- 응용 계층: 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 **도메인 계층을 조합해서 기능을 실행**한다.
- 도메인 계층: 시스템이 제공할 도메인 규칙을 구현한다.
- 인프라 계층: 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.

도메인 모델은 아키텍처 상의 **도메인 계층을 객체 지향 기법으로 구현하는 패턴**을 말한다. 

도메인 계층은 도메인의 핵심 규칙을 구현한다. 주문 도메인의 경우 아래 규칙을 구현한 코드가 도메인 계층에 위치하게 된다.

- 출고 전에 배송지를 변경 할 수 있다
- 주문 취소는 배송 전에만 할 수 있다

이런 도메인 규칙을 객체 지향 기법으로 구현하는 패턴이 도메인 모델 패턴이다.  

```java
public class Order {
	private OrderState state;
	private ShippingInfo shippingInfo;

	public void changeShippingInfo(ShippingInfo newShippingInfo) {
		if (!isShippingChangeableO) {
			throw new IUegalStateException("can't change shipping in " + state);
		}
		this.shippingInfo = newShippingInfo;
	}
	
	
	private boolean isShippingChangeableO {
		return (state == OrderState.PAYMENT_WAITING || state == OrderState.PREPARING);
	}
}
```

배송지 변경 가능 여부를 판단하는 기능이 Order에 있든 OrderState에 있든 중요한 점은  주문과 관련된 중요 업무 규칙을 주문 도메인 모델인 Order나 OrderState에서 구현한다는 점이다. 
핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확 장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.


## 5. 엔티티와 밸류

<img width="464" alt="image" src="https://github.com/user-attachments/assets/80e79214-7fd6-435a-b5db-2007ebfecbb7" />

### 5.1 엔티티

엔티티의 가장 큰 특징은 식별자를 가진다는 것이다. 예를 들어 주문 도메인에서 각 주문은 주문번호를 가지고 있는데, 이 주문번호는 각 주문마다 서로 다르다. 식별자는 아래와 같은 특징이 있다.

- 식별자는 불변이다.
- 식별자는 고유하다.

### 5.2 밸류 타입

#### 5.2.1 밸류 타입은 하나의 개념을 묶을 수 있다.

<img width="321" alt="image" src="https://github.com/user-attachments/assets/896793a5-efdf-4f7c-b6b7-3712ffd12333" />

ShippingInfo 클래스는 배송정보를 의미한다. 내부적으로 받는 사람과 주소 데이터를 갖고 있다. 
`receiverName` 필드와 `receiverPhoneNmnber`필드는 서로 다른 두 데이터를 담고 있지만, **개념적으로는 `받는 사람`이라는 하나의 개념을 표현**하고 있다. (아래 주소라는 하나의 개념적 표현도 마찬가지)  

밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다. 예를 들어 `받는 사람`을 위한 밸류 타입인 `Receiver`를 다음과 같이 작성할 수 있다.

```java
public class Receiver {
	private String name;
	private String phoneNumber;
}
```

주소도 마찬가지로 아래와 같이 `Address` 라는 밸류 타입으로 명확하게 정의할 수 있다. 

```java
public class Address {
	private String address1;
	private String address2;
	private String zipcode;
}
```

위 두 밸류 타입을 사용해서, 다시 `ShippingInfo`를 구현하면 아래와 같이 변경된다.

<img width="270" alt="image" src="https://github.com/user-attachments/assets/509e4269-dab1-4047-8663-d114156e55b0" />


----

#### 5.2.2 밸류 타입은 명확한 의미로 표현이 가능해진다.

밸류 타입은 의미를 명확하게 표현하기 위해 사용되는 경우도 있다. 

아래 `OrderLine` 클래스를 보자.

```java
public class OrderLine {
	private Product product;
	private int price;
	private int quantity;
	private int amounts;
}
```

`price`와 `amounts`는 int 타입을 사용하고 있지만, 개념적으로는 “돈”을 의미하는 값이다. 따라서 `Money` 타입을 만들어 사용하면 아래와 같이 의미가 명확해진다. 

- Money 밸류 타입 정의

```java
public class Money { 
	private int value;
	
	public Money(int value) {
		this.money = money;
	}
}
```

- int → Money 타입으로 변경

```java
public class OrderLine {
	private Product product;
	private Money price;
	private int quantity;
	private Money amounts;
}
```


----

#### 5.2.3 밸류 타입은 개념적인 의미에 대한 기능을 추가할 수 있다.

<img width="535" alt="image" src="https://github.com/user-attachments/assets/df3abe0a-b6a6-4e92-8861-fef88d5d953e" />

