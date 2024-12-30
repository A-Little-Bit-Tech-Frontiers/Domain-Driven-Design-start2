
## 1. 네 개의 영역

`표현`, `응용`, `도메인`, `인프라스트럭처`는 아키텍처를 설계할 때 출현하는 전형적인 네 가지 영역이다.

### 1.1 표현 영역

사용자의 요청을 받아 응용 영역에 전달하고, 응용 영역의 처리 결과를 다시 사용자에게 보여주는 역할을 한다. 웹 애플리케이션에서 표현 영역의 사용자는 웹 브라우저를 사용하는 사람일 수도 있고, REST API를 호출하는 외부 시스템일 수도 있다.

<img width="545" alt="image" src="https://github.com/user-attachments/assets/19ec87cc-a2ed-4170-b572-87af58c84ac9" />


### 1.2 응용 영역

(표현 영역을 통해 사용자의 요청을 전달받는)응용 영역은 시스템이 사용자에게 제공해야 할 기능을 구현한다.
주문등록, 주문취소, 상품 상세 조회와 같은 기능 구현을 예로 들 수 있다. 응용 영역은 기능을 구현하기 위해 도메인 영역의 도메인 모델을 사용한다.

주문 취소 기능을 제공하는 응용 서비스를 예로 살펴보면 다음과 같이 **주문 도메인 모델을 사용해서 기능을 구현**한다.

```java
public class CancelOrderService {
	
	@Transactional
	public void cancelOrder(String orderId) {
		Order order = findOrderById(orderId);
		if (order = = null) throw new OrderNotFoundException(orderId);
		order.cancel();
		}
		...
}
```

위 코드는 주문 취소 로직을 직접 구현하지 않고, Order 객체에 취소 처리를 위임하고 있다. 즉, 응용 서비스는 로직을 직접 수행하기보다는 도메인 모델에 로직 수행을 위임한다.

<img width="514" alt="image" src="https://github.com/user-attachments/assets/25b726ef-87aa-46f6-bac4-31adf1987d7a" />


### 1.3 도메인 영역

도메인 영역은 도메인 모델을 구현한다. 1장에서 봤던 Order, OrderLine, ShippingInfo와 같은 도메인 모델이 이 영역에 위치한다. 도메인 모델은 도메인의 핵심 로직을 구현한다.

### 1.4 인프라 영역

인프라 영역은 구현 기술에 대한 것을 다룬다. 예를 들어, RDBMS 연동을 처리하거나 메시징 큐에 메시지를 송/수신하는 기능을 구현하고, 각종 데이터베이스와의 데이터 연동을 처리한다. REST API 호출하는 것도 처리할 수 있다. 즉, 인프라스트럭처 영역은 논리적인 개념을 표현하기보다는 실제 구현을 다룬다.

<img width="444" alt="image" src="https://github.com/user-attachments/assets/31c48851-77c5-45f2-a436-e70703b90765" />


## 2. 계층 구조 아키텍처


<img width="505" alt="image" src="https://github.com/user-attachments/assets/6851296b-3c1c-4e0c-999a-96eb6b0bdd12" />

계층 구조는 그 특성상 상위 계층에서 하위 계층으로의 의존만 존재하고, 하위 계층은 상위 계층을 의존하지 않는다. 계층 구조를 엄격하게 적용한다면, 상위 계층은 바로 아래의 계층에만 의존을 가져야 하지만 구현의 편리함을 위해 유연하게 적용하기도 한다.

이 구조에서 구매한 가격이 얼마인지 계산하는 응용영역의 비즈니스 로직이 있다고 가정해보자. 그리고, 할인 금액을 계산하기 위해 Drools라는 룰 엔진을 사용해야 한다. 그리고, 이 룰 엔진은 인프라 계층에 구현되어 있다고 가정하자.

응용 영역은 아래와 같이 가격 계산을 위해 인프라 영역의 DroolsRuleEngine을 사용한다. 

```java
public class CalculateDiscountService {

	private DroolsRuleEngine ruleEngine;
	
	public CalculateDiscountService {
		ruleEngine = new DroolsRuleEngineO;
	}

	public Money calculateDiscount(List<OrderLine> orderLines, String customerId) {
		Customer customer = findCustomer(customerId);
		
		// Drools에 의존적인 코드다. Drools를 사용하기 위해서, Drools에 대한 지식이 필요하다.
		MutableMoney money = new MutableMoney(0);
		List<?> facts = Arrays.asList(customer, money);
		facts.addAll(orderLines);
		ruleEngine.evalute("discountCalculation", facts);
		//
		
		return money.toImmutableMoney();
	}
```

위 코드는 두 가지 문제점을 가지고 있다. 

- `CalculateDiscountService` 만 테스트하기가 어렵다.
    - 테스트하기 위해서 `RuleEngine` 클래스와 관련 설정 파일을 모두 만들어야 한다.
- 구현 방식을 변경하기 어렵다.
    - 주석에 적어놨지만, 구체적인 Drools에 의존성이 형성된다. 즉, Drools가 변경되면 CalculateDiscountService도 변경되어야 한다.
    - infra 모듈의 특정 기능을 사용하는 곳은 여러 모듈이 될 수도, 한 모듈에서 여러 도메인이 될 가능성이 높다. -> 그렇다는건 infra 모듈의 구현 방식이 변경되면, 사용하는 곳(다른 모듈)을 다 찾아다니면서 변경해줘야 한다.
    - 뿐만 아니라, 환경변수 등과 같은 값을 관리하는데도 중복이 많이 발생하게 된다. -> 예를 들어 redis 주소가 변경되면 상위 모듈에 찾아가 그 값을 다 변경해줘야 함 -> 생산성 저하
 
## 3. DIP

<img width="548" alt="image" src="https://github.com/user-attachments/assets/860f5055-0e99-4d90-9403-afbd808c99e1" />

- **고수준 모듈**
    - `CalculateDiscountService`는 ‘가격 할인 계산’이라는 의미 있는 단일 기능을 구현한다.
    고수준 모듈의 기능을 구현하려면 여러 하위 기능이 필요하다.
- **저수준 모듈**
    - 하위 기능을 실제로 구현한 것
    - 여기선, 고객 정보를 읽어오는 기능과 룰을 이용한 할인 금액을 구하는 기능이 하위 기능이다.
    이 기능들을 각각 JPA와 Drools를 사용해 구현하고 있다.

고수준 모듈을 동작하게 하기 위해선 저수준 모듈을 사용해야 한다. 그런데, 고수준 모듈이 저수준 모듈에 의존하게 되면 앞서 언급한 두 가지 문제가 발생한다.

DIP는 이 문제를 해결하기 위해 저수준 모듈이 고수준 모듈을 의존하도록 의존성을 역전시킨다. **추상화한 인터페이스를 통해서**.

- 추상화 인터페이스

```java
public interface RuleDiscounter {
	Money applyRules(Customer customer, List<OrderLine> orderLines);
}
```

위 인터페이스를 통해 CalculateDiscountService는 `RuleDiscounter.applyRules()` 만 사용하면 되고, 이는 “고객 정보와 구매 정보에 룰을 적용해서 할인 금액을 구한다”만 아는 것이다. CalculateDiscountService 입장에선 Drools로 구현되었는지 알 필요가 없는 것이다.

적용하면, 아래와 같다.

```java
public class DroolsRuleDiscounter implements RuleDiscounter {
 
	private KieContainer kContainer;

	public DroolsRuleEngine() {
		KieServices ks = KieServices.Factory.get();
		kContainer = ks.getKieClasspathContainer();
	}
		
	@Override
	public Money applyRules(Customer customer, List<OrderLine> orderLines) {
		KieSession kSession = kContainer.newKieSession("discountSession");
		
		try {
			... 코드 생략
			kSession .fireAllRules();
		} finally {
			kSession.dispose();
		}
		return money.toImmutableMoney();
	}
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

DIP를 적용하면서 아래와 같이 의존관계가 변경되었다.

<img width="388" alt="image" src="https://github.com/user-attachments/assets/c9723242-d724-4fed-81a4-e7ed5ddbacb5" />

`CalculateDiscountService`는 **더 이상 구현 기술인 Drools에 의존하지 않는다.** ‘룰을 이용한 할인 금액 계산’을 추상화한 RuleDiscounter 인터페이스에 의존할 뿐이다. **‘룰을 이용한 할인 금액 계산’은 고수준 모듈의 개념이므로 RuleDiscounter 인터페이스는 고수준 모듈에 속한다.** 
반면에, DroolsRuleDiscounter는 고수준의 **하위 기능을 구현**한 것이므로 **저수준 모듈에 속한다.**  

<img width="411" alt="image" src="https://github.com/user-attachments/assets/50cfcb5b-551f-43e8-ae72-84b0f6f37cb6" />


## 4. 도메인 영역의 주요 구성요소

<img width="594" alt="image" src="https://github.com/user-attachments/assets/be118142-031b-4e64-b4dd-2f7fe7c781ef" />



## 5. 모듈 구성

정답은 없다. 

### 5.1 영역별 별도 패키지 구성

<img width="362" alt="image" src="https://github.com/user-attachments/assets/78b2b6e8-89f0-45cf-9618-b0758412c553" />


### 5.2 하위 도메인별 모듈 구성

<img width="420" alt="image" src="https://github.com/user-attachments/assets/11a203ba-23fd-41e5-b06e-0acf1a821ff8" />


애그리거트, 모델, 리포지터리는 같은 패키지에 위치시킨다. 예를 들어 Order, OrderLine, Orderer, OrderRepository ^§- com.myshop.order.domain 패키지에 위치시킨다.  

도메인이 복잡하면 도메인 모델과 도메인 서비스를 다음과 같이 별도 패키지에 위치시킬 수도 있다.
• com.myshop.order.domain.order: 애그리거트 위치
• com.myshop.order.domain.service: 도메인 서비스 위치
