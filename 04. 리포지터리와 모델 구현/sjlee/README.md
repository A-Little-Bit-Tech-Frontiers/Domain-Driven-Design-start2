## 1. 도메인 구현과 DIP

```java
@Entity
@Table(name = "article") @SecondaryTable(
  name = "article_content",
  pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY) private Long id;
```

```java
import org.springframework.data.repository.Repository;

public interface ArticleRepository extends Repository<Article, Long> {
  void save(Article article);
  Optional<Article> findById(Long id);
}
```

DIP에 따르면 @Entity, @Table은 구현 기술에 속하므로 Article^ 같은 도메인 모델은 구현 기술인 JPA에 의존하지 말아야 하는데, 이 코드는 도메인 모델인 Article이 영속성 구현 기술인
JPA에 의존하고 있다.

ArticleRepository가 도메인 패키지에서 JPA의 Repository 인터페이스를 상속하고 있는 것도 마찬가지 맥락

<br>

<img width="342" alt="image" src="https://github.com/user-attachments/assets/20639d8a-ddd1-4207-86b3-0bbc48124d45" />

원칙을 지키려면, Article 클래스에서 @Entity나 @Table 같이 JPA에 특화된 애너테이션을 모두 지우고 위와 같이 infra 모듈에 JPA 연동을 위한 클래스를 추가해야 한다.  
이 구조를 가지면 구현 기술을 변경하더라도 도메인이 받는 영향을 최소화할 수 있다.

---

필자의 생각

- 결국, DIP를 적용하는 주된 이유는 저수준 구현이 변경되더라도 고수준이 영향을 받지 않도록 하기 위함
- 하지만 리포지터리와 도메인 모델의 구현 기술은 거의 바뀌지 않음
- 이렇게 변경이 거의 없는 상황에서 변경을 미리 대비하는건 과하다고 생각함
- 그래서 필자는 애그리거트, 리포지터리 등 도메인 모델을 구현할 때 타협함
- JPA 애노테이션을 도메인 모델에 사용하면서 기술에 따른 구현 제약이 낮다면 합리적인 선택이라고 생각한다고 함

<br>

내 생각

- 실무에서 원칙을 철저하게 지키는 건 어려움. 때문에, 어느정도 동의하긴 함
- 근데, domain과 infra간 DIP를 적용한다는 것은 domain 모델 영역에 추상화를 적용하고, infra 모듈에서 실제 구현 클래스를 가진다는 의미
- 즉, 개발하다보면 infra 모듈에서 엔티티를 알아야 할 경우가 꽤 많을 것 같음(e.g. zero-payload 방식에서의 internal api 등)
- 도메인 모델이 변경되면 인프라도 그에 따라 필수적으로 변경되어야 하는데, 결합도가 생각보다 더 올라가지 않을까라는 생각
- "결국에 엔티티는 infra에 있을 수 밖에 없지 않나"라는 생각
- 또 한편으로는 인프라에 있는 엔티티에 필드가 추가되었다는 것 -> 도메인 모델의 비즈니스 로직이 영향이 있다는 것이니,, 타협 가능할 것 같기도 함 😝
