# 리포지터리와 모델 구현

## JPA를 이용한 리포지터리 구현

### 모듈 위치

<img width="703" alt="스크린샷 2025-01-06 오후 8 33 27" src="https://github.com/user-attachments/assets/f31c9cc6-5b4d-4e1d-b1fb-af3ffa56a75a" />

### 리포지터리 기본 기능 구현

리포지터리가 제공하는 기본 기능은 다음 두 가지이다.

- ID로 애그리거트 조회하기
- 애그리거트 저장하기

애그리거트를 수정한 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다. <br>
JPA를 사용하면 트랜잭션 범위에서 변경한 데이터를 자동으로 DB에 반영하기 때문이다.

```java
public class ChangeOrderService {
    
    @Transactional
    public void changeShippingInfo(OrderNo no, ShippingInfo newShppingInfo) {
        Optional<Order> orderOpt = orderRepository.findById(no);
        Order order = orderOpt.orElseThrow(() -> new OrderNotFoundException());
        order.changeShippingInfo(newShippingInfo);
    }
}
```

changeShippingInfo() 메서드를 실행한 결과로 애그리거트가 변경되면 JPA는 변경 데이터를 DB에 반영하기 위해 UPDATE 쿼리를 실행한다.

## 매핑 구현

애그리거트와 JPA 매핑을 위한 기본 규칙은 다음과 같다.

- 애그리거트 루트는 엔티티이므로 @Entity로 매핑 설정한다.

<br>

한 테이블에 엔티티와 밸류가 같이 있다면

- 밸류는 @Embeddable로 매핑 설정한다.
- 밸류 타입 프로퍼티는 @Embedded로 매핑 설정한다.

주문 애그리거트를 예로 들어보자. 주문 애그리거트의 루트 엔티티는 Order이고 이 애그리거트에 속한 Orderer와 ShipppingInfo는 밸류이다.

<img width="710" alt="스크린샷 2025-01-06 오후 8 39 13" src="https://github.com/user-attachments/assets/0d7ce6f8-0f4a-4e5c-b137-65aadef4b9b5" />

## 애그리거트 로딩 전략

JPA 매핑을 설정할 때 항상 기억해야 할 점은 애그리거트에 속한 객체가 모두 모여야 완전한 하나가 된다는 것이다.

```java
// product는 완전한 하나여야 한다.
Product product = productRepository.findById(id);
```

**애그리거트는 개념적으로 하나여야 한다. 하지만 루트 엔티티를 로딩하는 시점에 애그리거트에 속한 객체를 모두 로딩해야 하는 것은 아니다.** <br>
애그리거트가 완전해야 이유는 두 가지 정도로 생각해 볼 수 있다. <br>
첫 번째 이유는 상태를 변경하는 기능을 실행할 때 애그리거트 상태가 완전해야 하기 때문이고, <br>
두 번째 이유는 표현 영역에서 애그리거트의 상태 정보를 보여줄 때 필요하기 때문이다.

상태 변경 기능을 실행하기 위해 조회 시점에 즉시 로딩을 이용해서 애그리거트를 완전한 상태로 로딩할 필요는 없다. <br>
四人는 트랜잭션 범위 내에서 지연 로딩을 허용하기 때문에 다음 코드처럼 실제로 상태를 변경하는 시점에 필요한 구성요소만 로딩해도 문제가 되지 않는다.

```java
@Transactional
public void removeOptions(ProductId id, int optIdxToBeDeleted) {
    // product를 로딩. 컬렉션은 지연 로딩으로 설정했다면, Option은 로딩하지 않음
    Product product = productRepository.findById(id);
    // 트랜잭션 범위이므로 지연 로딩으로 설정한 연관 로딩 가능
    product.removeOption(optIdxToBeDeleted);
}

@Entity
public class Product {

    @ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(name = "product_option", joinColumns = @JoinColumn(name = "product_id"))
    @OrderColumn(name = "list_idx")
    private List<Option> options = new ArrayListo();

    public void removeOption(int optIdx) {
        // 실제 컬렉션에 접근할 때 로딩
        this.options.remove(optIdx);
    }
}
```

## 애그리거트의 영속성 전파

애그리거트가 완전한 상태여야 한다는 것은 애그리거트 루트를 조회할 때뿐만 아니라 저장하고 삭제할 때도 하나로 처리해야 함을 의미한다. <br>

- 저장 메서드는 애그리거트 루트만 저장하면 안 되고 애그리거트에 속한 모든 객체를 저장해야 한다.
- 삭제 메서드는 애그리거트 루트뿐만 아니라 애그리거트에 속한 모든 객체를 삭제해야 한다.

@Embeddable 매핑 타입은 함께 저장되고 삭제되므로 cascade 속성을 추가로 설정하지 않아도 된다. <br>
반면에 애그리거트에 속한 @Entity 타입에 대한 매핑은 cascade 속성을 사용해서 저장과 삭제 시에 함께 처리되도록 설정해야 한다.

## 도메인 구현과 DIP

```java
@Entity
@Table(name = "article")
@SecondaryTable(
        name = "article_content",
        pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

```java
import org.springframework.data.repository.Repository;

public interface ArticleRepository extends Repository<Article, Long> {
    void save(Article article);
    Optional<Article> findById(Long id);
}
```

DIP에 따르면 @Entity, @Table은 구현 기술에 속하므로 Article과 같은 도메인 모델은 구현 기술인 JPA에 의존하지 말아야 하는데 이 코드는 도메인 모델인 Article이 영속성 구현 기술이 JPA에 의존하고 있다. <br>
리포지터리 인터페이스도 마찬가지다. ArticleRepository 인터페이스는 도메인 패키지에 위치하는데 구현 기술인 스프링 데이터 JPA의 Repository 인터페이스를 상속하고 있다. 즉 도메인이 인프라에 의존하는 것이다.

**DIP를 적용하는 주된 이유는 저수준 구현이 변경되더라도 고수준이 영향을 받지 않도록 하기 위함이다.** <br>
하지만 리포지터리와 도메인 모델의 구현 기술은 거의 바뀌지 않는다. <br>
변경이 거의 없는 상황에서 변경을 미리 대비하는 것은 과하다고 생각한다. 그래서 애그리거트, 리포지터리 등 도메인 모델을 구현할때는 타협을 했다.













