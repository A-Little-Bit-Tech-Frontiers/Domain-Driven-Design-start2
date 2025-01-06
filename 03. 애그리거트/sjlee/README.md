
## 1. 애그리거트

<img width="379" alt="image" src="https://github.com/user-attachments/assets/e0c10d6a-0bf3-4d05-8695-d817c76a3c73" />

도메인 객체 모델이 복잡해지면 개별 구성요소 위주로 모델을 이해하게 되고 전반적인 구조나 큰 수준에서 도메인 간의 관계를 파악하기 어려워진다. 
**주요 도메인 요소 간의 관계를 파악하기 어렵다는 것은 코드를 변경하고 확장하는 것이 어려워진다는 것을 의미한다.**   
그리고, **복잡한 도메인을 이해하고 관리하기 쉬운 단위로 만들려면 상위 수준에서 모델을 조망할 방법이 필요**하다. 이 방법이 **애그리거트**다.

도메인 규칙에 따라 최초 주문 시점에 일부 객체를 만들 필요가 없는 경우도 있지만 애그리거트에 속한 구성요소는 대부분 함께 생성하고 함께 제거한다.

위 다이어그램처럼 애그리거트는 경계를 갖는다. 독립된 객체군이며 각 애그리거트는 자기 자신을 관리할 뿐, 다른 애그리거트는 관리하지 않는다. **경계를 설정할 때 기본이 되는 것은 도메인 규칙과 요구사항**이다. 
**도메인 규칙에 따라 함께 생성되는 구성요소는 한 애그리거트에 속할 가능성이 높다.**

----

## 2. 애그리거트 루트

주문 애그리거트는 다음을 포함한다. 

- 총 금액인 totalAmounts를 갖고 있는 Order 엔티티
- 개별 구매 상품의 개수인 quantity와 금액인 price를 갖고 있는 OrderLine 밸류

이때 구매할 상품의 개수를 변경하면, 한 OrderLine의 quantity를 변경함과 동시에 Order의 totalAmounts도 변경해야 한다. 그렇지 않으면 아래 도메인 규칙을 어기고 데이터 일관성이 깨진다.

- 주문 총 금액은 (개별 구매 상품 개수*가격)의 합

**애그리거트는 여러 객체로 구성되기 때문에 한 객체만 상태가 정상이면 안된다. 도메인 규칙을 지키려면 애그리거트에 속한 모든 객체가 정상 상태를 가져야 한다.** 
애그리거트에 속한 모든 객체가 일관된 상태를 유지하려면 **애그리거트 전체를 관리할 주체**가 필요한데, 이 책임을 지는 것이 바로 **애그리거트의 루트 엔티티**이다.  
애그리거트에 속한 객체는 애그리거트 루트 엔티티에 직접 또는 간접적으로 속하게 된다.

<br>

### 2.1 도메인 규칙과 일관성

**애그리거트 루트의 핵심 역할은 애그리거트의 일관성이 깨지지 않도록 관리**하는 것이다. 
이를 위해 애그리거트가 제공해야 할 도메인 기능을 애그리거트가 구현한다. 예를 들어 주문 애그리거트는 배송지 변경, 상품 변경과 같은 기능을 제공하고, 애그리거트 루트인 Order가 이 기능을 구현한 메서드를 제공한다.

```java
public class Order{
	// 애그리거트 루트는 도메인 규칙을 구현한 기능을 제공한다.
	public void changeShippingInfo(ShippingInfo newShippingInfo) { 
		verifyNotYetShipped();
		setShippingInfo(newShippingInfo);
	}
	
	private void verifyNotYetShipped() {
		if (state != orderState.PAYMENT_WAITING && state != OrderState.PREPARING)
			throw new IllegalStateException("aleady shipped");
	}
}
```

애그리거트 외부에서 애그리거트에 속한 객체를 직접 변경하면 안된다. 이것은 애그리거트  루트가 강제하는 규칙을 적용할 수 없어 모델의 일관성을 깨는 원인이 된다. 다음 코드를 보자. 

```java
ShippingInfo si = order.getShippingInfo();
si.setAddress(newAddress);
```

위 코드는 애그리거트 루트에 속한 ShippingInfo를 직접 가져와 배송지 주소를 변경하고 있다. 이는 업무 규칙을 무시하고 DB 테이블을 직접 수정하는 것과 같은 결과를 만든다. **즉, 논리적인 데이터 일관성이 깨진다.** 

일관성을 지키기 위해 다음과 같이 상태 확인 로직을 응용 서비스에 구현할 수도 있다. 하지만 이렇게 되면 **동일한 검사 로직을 여러 응용 서비스에서 중복으로 구현할 가능성이 높아져 유지 보수에 도움이 되지 않는다.**

```java

ShippingInfo si = order.getShippingInfo();

// 주요 도메인 로직이 중복되는 문제
if (state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING) {
	throw new IUegalArgumentException();
}
si.setAddress(newAddress);
```


불필요한 중복을 피하고 애그리거트 루트를 통해서만 도메인 로직을 구현하게 만들려면 도메 인 모델에 대해 다음의 두 가지를 습관적으로 적용해야 한다.

- 단순 필드 변경 메서드를 public 범위로 만들지 않는다.
- 밸류 타입은 불변으로 구현한다.

```java
public class Order {
	private ShippingInfo shippingInfo;

	public void changeShippingInfo(ShippingInfo newShippingInfo) {
		verifyNotYetShipped();
		setShippingInfo(newShippingInfo);
	}

	// set 메서드의 접근 허용 범위는 private이다.
	private void setShippingInfo(ShippingInfo newShippingInfo) {
		// 밸류가 불변이면 새로운 객체를 할당해서 값을 변경해야 한다.
		// 불변이므로 this.shippingInfo.setAddress()와 같은 코드를 사용할 수 없다.
		this.shippingInfo = newShippingInfo;
}
```


<br>

### 2.2 트랜잭션 범위

한 트랜잭션에서는 한 개의 애그리거트만 수정해야 한다. 한 트랜잭션에서 두 개 이상의 애그리거트를 수정하면 트랜잭션 충돌이 발생할 가능성이 더 높아지기 때문에, 한 번에 수정하는 애그리거트 개수가 많아질수록 전체 처리량이 떨어지게 된다.

**한 트랜잭션에서 한 애그리거트만 수정한다는 것은 애그리거트에서 다른 애그리거트를 변경하지 않는다는 것을 의미**한다.
한 애그리거트에서 다른 애그리거트를 수정하면 결과적으로 두 애그리거트를 한 트랜잭션에서 수정하게 되므로, 애그리거트 내부에서 다른 애그리거트의 상태를 변경하는 기능을 실행하면 안된다.

예를 들어, 배송지 정보를 변경하면서 동시에 배송지 정보를 회원의 주소로 설정하는 기능이 있다고 해보자. 이때 주문 애그리거트는 다음과 같이 회원 애그리거트의 정보를 변경하면 안된다.

```java
public class Order {
	private Orderer orderer;

	public void shipTo(ShippingInfo newShippingInfo, boolean useNewShippingAddrAsMemberAddr) {
		verifyNotYetShipped();
		setShippingInfo(newShippingInfo);

		if (useNewShippingAddrAsMemberAddr) {
			// 다른 애그리거트의 상태를 변경하면 안됨
			orderer.getMember().changeAddress(newShippingInfo.getAddress());
		}
}
```

이것은 애그리거트가 자신의 책임 범위를 넘어 다른 애그리거트의 상태까지 관리하는 꼴이 된다. 애그리거트는 최대한 독립적이어야 하는데 한 애그리거트가 다른 애그리거트의 기능에 의존하게 되면 애그리거트간 결합도가 높아진다. 

만약 부득이하게 한 트랜잭션으로 두 개 이상의 애그리거트를 수정해야 한다면, 애그리거트에서 다른 애그리거트를 직접 수정하지 말고 **응용 서비스에서 두 애그리거트를 수정하도록 구현한다.**

```java
public class ChangeOrderService {
	
	// 두 개 이상의 애그리거트를 변경해야 하면,
	// 응용 서비스에서 각 애그리거트의 상태를 변경한다.
	@Transactional
	public void changeShippingInfo(
		OrderId id,
		ShippingInfo newShippingInfo,
		boolean useNewShippingAddrAsMemberAddr
	) {
		Order order = orderRepository.findbyId(id);
		if (order = = null) throw new OrderNotFoundException();
		
		order.shipTo(newShippingInfo);
		if (useNewshippingAddrAsMemberAddr) {
			Member member = findMember(order.getOrderer());
			member.changeAddress(newShippingInfo.getAddress());
		}
}
```

----

## 3. 리포지터리와 애그리거트

**애그리거트는 개념상 완전한 한 개의 도메인 모델을 표현하므로 객체의 영속성을 처리하는 리포지터리는 애그리거트 단위로 존재한다.** 
Order와 OrderLine을 물리적으로 다른 테이블에 저장한다고 해서 각 리포지터리를 만들지 않고, 애그리거트인 Order의 리포지터리만 존재한다.


----

## 4. ID를 이용한 애그리거트 참조

한 객체가 다른 객체를 참조하는 것처럼 애그리거트도 다른 애그리거트를 참조한다.

<img width="437" alt="image" src="https://github.com/user-attachments/assets/2ae6cd50-ad6f-4731-80c3-7eaffe404db3" />

필드를 이용한 애그리거트간 직접 참조는 편리함을 제공한다. 예를 들어, 주문 정보 조회 화면에서 회원 ID를 이용해 링크를 제공해야 할 경우 Order로부터 시작해서 회원 ID를 구할 수 있다.

하지만 직접 참조할 때, 가장 큰 문제는 편리함을 오용할 수 있다는 것이다. 한 애그리거트 내부에서 다른 애그리거트 객체에 접근할 수 있으면 다른 애그리거트의 상태를 쉽게 변경할 수 있게 된다. **트랜잭션 범위에서 언급한 것처럼 한 애그리거트가 관리하는 범위는 자기 자신으로 한정해야 한다. 그런데 애그리거트 내부에서 다른 애그리거트 객체에 접근할 수 있으면 다음 코드처럼 구현의 편리함 때문에 다른 애그리거트를 수정하고자 하는 유혹에 빠지기 쉽다.** 

두 번째 문제는 확장이다. 초기에는 단일 서버에 단일 DBMS로 서비스를 제공하는 것이 가능하다. 하지만, 사용자가 늘고 트래픽이 증가하면 자연스럽게 부하를 분산하기 위해 하위 도메인별로 시스템을 분리하기 시작한다. 이 과정에서 하위 도메인마다 서로 다른 DBMS를 사용할 때도 있다. 심지어 하위 도메인마다 다른 종류의 데이터 저장소를 사용하기도 한다. 이것은 더 이상 다른 애그리거트 루트를 참조하기 위해 JPA와 같은 단일 기술을 사용할 수 없음을 의미한다.

이러한 문제를 완화할 때, 사용할 수 있는 것이 ID를 이용해서 참조하는 것이다.

<img width="384" alt="image" src="https://github.com/user-attachments/assets/a669be1f-dcca-42b2-8424-79ff8f30df84" />

<br>

### 4.1 ID를 이용한 참조와 조회 성능

다른 애그리거트를 ID로 참조하면 참조하는 여러 애그리거트를 읽을 때 조회 속도가 문제 될 수 있다. 예를 들어 주문 목록을 보여주려면 상품 애그리거트와 회원 애그리거트를 함께 읽어야 하는데, 이를 처리할 때 다음과 같이 각 주문마다 상품과 회원 애그리거트를 읽어온다고 해보자. 

```java
Member member = memberRepository.findById(ordererId);
List<Order> orders = orderRepository.findByOrderer(ordererId);
List<OrderView> dtos = orders.stream()
	.map(order -> {
		// 각 주문마다 첫 번째 주문 상품 정보 로딩 위한 쿼리 실행
		ProductId prodId = order.getOrderLines().get(0).getProductId();
		Product product = productRepository.findByIcKprodId);
		return new OrderView(order, member, product);
}).toList();
```

위 코드는 주문 개수가 10개면, 주문을 읽어오기 위한 한 번의 쿼리와 주문별로 각 상품을 조회하기 위한 열 번의 쿼리를 실행한다. 
이는 조인을 사용해서 한 번에 가져오도록 변경되어야 한다. 때문에, 조회 전용 쿼리를 사용해 처리할 수 있다.  
예를 들어 데이터 조회를 위한 별도 DAO를 만들고 DAO의 조회 메서드에서 조인을 이용해 한 번의 쿼리로 필요한 데이터를 로딩하면 된다.

애그리거트마다 서로 다른 저장소를 사용하면, 한 번의 쿼리로 관련 애그리거트를 조회할 수 없다. 
이때는 조회 성능을 높이기 위해 캐시를 적용하거나 조회 전용 저장소를 따로 구성한다. 이 방법은 코드가 복잡해지는 단점이 있지만 시스템의 처리량을 높일 수 있다는 장점이 있다. 예를 들어, CQRS.
