# 스프링 데이터 JPA를 이용한 조회 기능

## 스프링 데이터 JPA를 이용한 스펙 구현

목록 조회와 같은 기능은 다양한 검색 조건을 조합해야 할 때, 필요한 조합마다 find 메서드를 정의할 수도 있지만 이것은 좋은 방법이 아니다. <br>
검색 조건을 다양하게 조합해야 할 때 사용할 수 있는 것이 스펙이다. <br>
스프링 데이터 JPA는 검색 조건을 표현하기 위한 인터페이스인 Specification을 제공한다.

```java
// 스펙 인터페이스를 구현한 클래스 예시
public class OrdererIdSpec implements Specification<OrderSummary> {

    private String ordererId;
    
    public OrdererIdSpec(String ordererId) {
        this.ordererId = ordererId;
    }
    
    @Override
    public Predicate toPredicate(Root<OrderSummary> root,
                            CriteriaQuery<?> query,
                            CriteriaBuilder cb) {
        return cb.equal(root.get(OrderSummary.ordererId), ordererId);
    }
}
```

## 리포지터리/DAO에서 스펙 사용하기

```java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {
    List<OrderSummary> findAll(Specification<DrderSummary> spec); 
}
```

findAll() 메서드는 OrderSummary에 대한 검색 조건을 표현하는 스펙 인터페이스를 파라미터로 갖는다.

## 스펙 조합

스펙 인터페이스는 스펙을 조합할 수 있는 두 메서드를 제공하고 있다. 이 두 메서드는 and와 or다.

```java
public interface Specification<T> extends Serializable {
    
    default Specification<T> and(@Nullable Specification<T> other) { }
    default Specification<T> or(@Nullable Specification<T> other) { }
    
    @Nullable
    Predicate toPredicate(Root<T> root,
                          CriteriaQuery<?> query,
                          CriteriaBuilder criteriaBuilder);
} 
```

## 정렬 지정하기

스프링 데이터 JPA는 두 가지 방법으로 정렬을 지정할 수 있다.

- 메서드 이름에 OrderBy를 사용해서 정렬 기준 지정
- Sort를 인자로 전달

메서드 이름에 OrderBy를 사용하는 방법은 간단하지만 정렬 기준 프로퍼티가 두 개 이상이면 메서드 이름이 길어지는 단점이 있다. <br>
또한 메서드 이름으로 정렬 순서가 정해지기 때문에 상황에 따라 정렬 순서를 변경할 수도 없다. 이럴 때는 Sort 타입을 사용하면 된다.

```java
public interface OrderSummaryDao extends Repository<OrderSummary, String> {

    List<OrderSummary> findByOrdererId(String ordererId, Sort sort);
    List<OrderSummary> findAll(Specification<DrderSummary> spec, Sort sort);
}
```

```java
Sort sort = Sort.by("number").ascending();
List<OrderSummary> results = orderSummaryDao.findByOrdererId("user1", sort);
```

## 페이징 처리하기

스프링 데이터 JPA는 페이징 처리를 위해 Pageable 타입을 이용한다. <br>
Sort 타입과 마찬가지로 find 메서드에 Pageable 타입 파라미터를 사용하면 페이징을 자동으로 처리해준다.

```java
public interface MemberDataDao extends Repository<MemberData, String> {

    List<MemberData> findByNameLike(String name, Pageable pageable);
}
```





