# 스프링 데이터 JPA를 이용한 조회 기능


## 1 검색을 위한 스펙

리스트 조회 등과 같이 다양한 검색 조건을 조합해야 하는 경우에는 스펙(specification)을 사용할 수 있다.  스펙은 애그리거트가 특정 조건을 충족하는지를 검사할 때 사용하는 인터페이스다. 

```java
public interface Speficiation<T> {
	public boolean isSatisfiedBy(T agg);
}
```

`isSatisfiedBy()` 메서드의 `agg` 파라미터가 검색 대상이 되는 객체다. 스펙을 리포지토리에 사용하면 agg는 애그리거트 루트가 되고, DAO에 사용하면 agg는 검색 결과로 리턴할 데이터 객체가된다. 

이 메서드는 검색 대상 객체가 조건을 만족하면 true를 리턴한다. 예를 들어, Order 애그리거트가 특정 고객의 주문이 맞는지 확인하는 스펙을 구현하면 아래와 같다.  

```java
public class OrdererSpec implements Specification<Drder> {
	
	private String ordererId;

	public OrdererSpec(String ordererId) {
		this.ordererId = ordererId;
	}

	public boolean isSatisfiedBy(Order agg) {
		return agg.getOrdererId().getMemberId().getIdO.equals(ordererId);
	}
}
```

만약, 리포지토리가 모든 애그리거트 객체를 보관하고 있다면 아래와 같이 스펙을 사용할 수 있다.  

```java
public class MemoryOrderRepository implements OrderRepository {
public List<Order> findAU(Specification<Order> spec) {
	List<Order> allOrders = findAll();
	return allOrders.stream()
									.filter(order -> spec .isSatisfiedBy(order))
									.toList();
}
```

---

스프링 데이터 JPA는 검색 조건을 표현하기 위한 `Specification` 인터페이스를 제공한다. 

```java
public interface Specification<T> extends Serializable {
	// not, where, and, or 메서드 생략
	@Nullable
	Predicate toPredicate(
		Root<T> root,
		CriteriaQuery<?> query,
		CriteriaBuilder cb
		);
}
```

예를 들어 OrederId 맞는지 확인하기 위한 검색 조건을 주고 생성하고 싶다면, `Specification<OrderEntity>`를 구현한 `OrderIdSpec`이란 클래스를 정의하고, `toPredicate()`를 구현하면 된다.

아니면 아래와 같이 스펙 구현 클래스를 개별로 만들지 않고, 별도 클래스에 스펙 생성 기능을 모아두는 것도 방법이다. 

```java
public class OrderSummarySpecs {

	public static Specification<OrderSummary> ordererId(String ordererId) {
		return (Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuilder cb) ->
			cb.equal(root.<String〉get("ordererIdH), ordererId);
	}
	
	public static Specification<OrderSummary> orderDateBetween(LocalDateTime from, LocalDateTime to) {
		return (Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuilder cb) ->
			cb.between(root.get(OrderSummary_.orderDate), from, to);
	}
}
```


<br>

## 2. 페이징 처리하기
목록을 보여줄 때 전체 데이터 중 일부만 보여주는 페이징 처리는 기본이다. JPA는 페이징 처리를 위해 Pageable 타입을 이용한다. Sort 타입과 마찬가지로 find 메서드에 Pageable 타입 파라미터를 사용하면 페이징을 자동으로 처리해 준다.

```java
import org.springframework.data.domain.Pageable;

public interface MemberDataDao extends Repository<MemberData, String> {
		List<MemberData> findByNameLike(String name, Pageable pageable);
}
```

위 코드에서 findByNameLike() 메서드는 마지막 파라미터로 Pageable 타입을 갖는다. `Pageable` 타입은 인터페이스로 실제 **Pageable 타입 객체는 PageRequest 클래스를 이용해서 생성**한다. 

PageRequest와 Sort를 사용하면 정렬 순서를 지정할 수 있다. 

```java
Sort sort = Sort.by("name").descending();
PageRequest pageReq = PageRequest.of(l, 2, sort);
List<MemberData> user = memberDataDao.findByNameLike("사용자%" pageReq);
```

Page 타입을 사용하면 데이터 목록뿐만 아니라 조건에 해당하는 전체 개수도 구할 수 있다.

```java
public interface MemberDataDao extends Repository<MemberData, String> {
		Page<MemberData> findByBlocked(boolean blocked. Pageable pageable);
}
```

Pageable을 사용하는 메서드의 리턴 타입이 Page일 경우 스프링 데이터 JPA는 목록 조회 쿼리와 함께 COUNT 쿼리도 실행해서 조건에 해당하는 데이터 개수를 구한다. 
Page는 전체 개수, 페이지 개수 등 페이징 처리에 필요한 데이터도 함께 제공한다. 다음은 Page가 제공하는 메서드의 일부를 보여준다.

```java
Pageable pageReq = PageRequest.of(2, 3);
Page<MemberData> page = memberDataDao.findByBlocked(false, pageReq);
List<MemberData> content = page.getContentO; // 조회 결과 목록
long totalElements = page.getTotalElementsO; // 조건에 해당하는 전체 개수
int totalPages = page.getTotalPages(); // 전체 페이지 번호
int number = page.getNumberO; // 현재 페이지 번호
int numberOfElements = page.getNumberOfElementsO; // 조회 결과 개수
int size = page.getSizeO; // 페이지 크기
```

프로퍼티를 비교하는 findBy프로퍼티 형식의 메서드는 Pageable 타입을 사용하더라도 리턴 타입이 List면 COUNT 쿼리를 실행하지 않는다. 즉, 다음 두 메서드 중에서 두 번째 findByNameLike() 메서드는 전체 개수를 구하기 위한 COUNT 쿼리를 실행하지 않는다.

```java
Page<MemberData> findByBlocked(boolean blocked. Pageable pageable);
List<MemberData> findByNameLike(String name. Pageable pageable);
```

페이징 처리와 관련된 정보가 필요 없다면 Page 리턴 타입이 아닌 ListS 사용해서 불필요한 COUNT 쿼리를 실행하지 않도록 해야한다.

