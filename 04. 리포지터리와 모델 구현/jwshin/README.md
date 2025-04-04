# 💡 4.1 Jpa를 이용한 리포지터리 구현

![image](https://github.com/user-attachments/assets/4e4cbac5-3f8c-48ff-baad-a039063085f8)

# 💡 4.2 스프링 데이터 JPA를 이용한 리포지터리 구현

- 여긴 그냥 기술 설명

# 💡 4.3 매핑 구현

### 4.3.1 엔티티와 밸류 기본 매핑 구현

- 애그리거트와 JPA 매핑을 위한 기본 규칙은 다음과 같다.
  - 애그리거트 루트는 엔티티이므로 @Entity로 매핑 설정한다.
  - 밸류는 @Embeddable로 매핑 설정한다.
  - 밸류 타입 프로퍼티는 @Embedded로 매핑 설정한다.
 
### 4.3.5 밸류 컬렉션： 별도 테이블 매핑

- Order 엔티티는 한 개 이상의 OrderLine을 가질 수 있다. OrderLine에 순서가 있다면 다음과 같이 List 타입을 이용해서 컬렉션을 프로퍼티로 지정할 수 있다.
- Order와 OrderLine을 저장하기 위한 테이블은 ［그림 4.4］와 같이 매핑 가능하다.

![image](https://github.com/user-attachments/assets/2a0eedbb-666d-4fe6-915f-383d4ad520f0)

- 밸류 컬렉션을 저장하는 ORDER_LINE 테이블은 외부키를 이용해서 엔티티에 해당하는 PURCHASE_ORDER 테이블을 참조한다. 이 외부키는 컬렉션이 속할 엔티티를 의미한다.
- 밸류 컬렉션을 별도 테이블로 매핑할 때는 `ElementCollection`과 `@CollectionTable`을 함께 사용한다. 관련 매핑 코드는 다음과 같다.

![image](https://github.com/user-attachments/assets/3d5555fe-e3d7-4906-91ed-90106ff04eaf)

- JPA는 `@OrderColumn` 애너테이션을 이용해서 지정한 칼럼에 리스트의 인덱스 값을 저장한다.
- `@CollectionTable`은 밸류를 저장할 테이블을 지정한다.

### 4.3.7 밸류를 이용한 ID 매핑

- 식별자라는 의미를 부각시키기 위해 식별자 자체를 밸류 타입으로 만들 수도 있다.
- 밸류 타입을 식별자로 매핑하면 @Id 대신 `@EmbeddedId` 애너테이션을 사용한다.

![image](https://github.com/user-attachments/assets/768cc635-ea1f-459b-ac51-18d95c775967)

- JPA에서 식별자 타입은 Serializable 타입이어야 하므로 식별자로 사용할 밸류 타입은 Serializable 인터페이스를 상속받아야 한다.
- 밸류 타입으로 식별자를 구현할 때 얻을 수 있는 장점은 식별자에 기능을 추가할 수 있다는 점이다.

### 4.3.8 별도 테이블에 저장하는 밸류 매핑

- 애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류이다.
- 루트 엔티티 외에 또다른 엔티티가 있다면 진짜 엔티티인지 의심해 봐야 한다.
- 단지 별도 테이블에 데이터를 저장한다고 해서 엔티티인 것은 아니다.
  - 주문 애그리거트도 OrderLine을 별도 테이블에 저장하지만 OrderLine 자체는 엔티티가 아니라 밸류이다.
- 밸류가 아니라 엔티티가 확실하다면 해당 엔티티가 다른 애그리거트는 아닌지 확인해야 한다.
  - 특히 자신만의 독자적인 라이프 사이클을 갖는다면 구분되는 애그리거트일 가능성이 높다.
  - 상품과 리뷰 예제를 떠올려 보자. Review는 엔티티가 맞지만 리뷰 애그리거트에 속한 엔티티이지 상품 애그리거트에 속한 엔티티는 아니다.
- 식별자를 찾을 때 매핑되는 테이블의 식별자를 애그리거트 구성요소의 식별자와 동일한 것으로 착각하면 안 된다.
  - 별도 테이블로 저장하고 테이블에 PK가 있다고 해서 테이블과 매핑되는 애그리거트 구성요소가 항상 고유 식별자를 갖는 것은 아니기 때문이다.

![image](https://github.com/user-attachments/assets/c93fe7f2-77b1-4baf-9a9a-755ea3fb668e)

- ArticleCotent는 밸류이므로 `@Embeddable`로 매핑한다. ArticleContent와 매핑되는 테이블은 Article과 매핑되는 테이블과 다르다.
- 이때 밸류를 매핑 한 테이블을 지정하기 위해 `@SecondaryTable`과 `@AttributeOverride`를 사용한다.

![image](https://github.com/user-attachments/assets/ce3eeff4-fa45-4f56-9fa6-d226271b2e61)

# 💡 4.4 애그리거트 로딩 전략

- JPA 매핑을 설정할 때 항상 기억해야 할 점은 애그리거트에 속한 객체가 모두 모여야 완전한 하나가 된다는 것이다.
- 즉, 다음과 같이 애그리거트 루트를 로딩하면 루트에 속한 모든 객체가 완전한 상태여야 함을 의미한다.

![image](https://github.com/user-attachments/assets/9bcade96-4e51-4edb-b94e-2703eede2e2b)

- 즉시 로딩 방식으로 설정하면 애그리거트 루트를 로딩하는 시점에 애그리거트에 속한 모든 객체를 함께 로딩할 수 있는 장점이 있지만 이것이 항상 좋은 것은 아니다.
- 특히 컬렉션에 대해 로딩 전략을 FetchType.EAGER로 설정하면 오히려 즉시 로딩 방식이 문제가 될 수 있다. (because. 카테시안곱 문제)
- 적절하게 즉시로딩과 지연로딩 전략을 구분해서 사용해야 한다.

# 💡 4.5 애그리거트의 영속성 전파

- 애그리거트가 완전한 상태여야 한다는 것은 애그리거트 루트를 조회할 때뿐만 아니라 저장하고 삭제할 때도 하나로 처리해야 함을 의미한다.
  - 저장 메서드는 애그리거트 루트만 저장하면 안 되고 애그리거트에 속한 모든 객체를 저장해야 한다.
  - 삭제 메서드는 애그리거트 루트뿐만 아니라 애그리거트에 속한 모든 객체를 삭제해야 한다.
- `@Embeddable` 매핑 타입은 함께 저장되고 삭제되므로 cascade 속성을 추가로 설정하지 않아도 된다.
- 반면에 애그리거트에 속한 @Entity 타입에 대한 매핑은 cascade 속성을 사용해서 저장과 삭제 시에 함께 처리되도록 설정해야 한다.  

# 💡 4.6 식별자 생성 기능

- 식별자는 크게 세 가지 방식 중 하나로 생성한다.
  - 사용자가 직접 생성
  - 도메인 로직으로 생성
  - DB를 이용한 일련번호 사용
- 식별자 생성 규칙이 있다면 엔티티를 생성할 때 식별자를 엔티티가 별도 서비스로 식별자 생성 기능을 분리해야 한다.

# 💡 4.7 도메인 구현과 DIP

- 구현 기술에 대한 의존 없이 도메인을 순수하게 유지하려면 스프링 데이터 JPA의 Repository 인터페이스를 상속받지 않도록 수정하고 ［그림 4.9]와 같이 ArticleRepository 인터페이스를 구현한 클래스를 인프라에 위치시켜야 한다.
- 또한 Article 클래스에서 @Entity @Table 같이 JPA에 특화된 애너테이션을 모두 지우고 인프라에 JPA를 연동하기 위한 클래스를 추가해야 한다.

![image](https://github.com/user-attachments/assets/4dba47c9-bd12-4757-892a-9b4c7ce07430)

- DIP를 적용하는 주된 이유는 저수준 구현이 변경되더라도 고수준이 영향을 받지 않도록 하기 위함이다.
- 필자는 도메인, 레포지토리 구현기술이 변하지 않을 것인데, 이렇게 나누는 것이 과하다고 생각해서 애그리거트, 리포지터리 등 도메인 모델을 구현할 때 타협했다고 한다. (🤔 확실히 JPA라는 기술이 standard해서 굳이 바꿀 필요를 못 느낄 것 같음. RDBMS도 마찬가지)
- DIP를 완벽하게 지키면 좋겠지만 개발 편의성과 실용성을 가져가면서 구조적인 유연함은 어느정도 유지하는 것도 방법이다. (도메인 계층에 JPA관련 코드가 있는 것)
