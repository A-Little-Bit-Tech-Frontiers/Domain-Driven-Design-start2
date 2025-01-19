# 5. 스프링 JPA 를 이용한 조회 기법 기능

### Specification 을 사용한 검색을 위한 조회

```java
import org.springframework.data.jpa.domain.Specification;

// 스펙 람다식 정의
public class PlayerSpecs {
    public static Specification<Player> nonBlocked() {
        return (root, query, cb) -> cb.equal(root.<Boolean>get("blocked"), false);
    }
    public static Specification<Player> nameLike(String keyword) {
        return (root, query, cb) -> cb.like(root.<String>get("name"), keyword + "%");
    }
}
...

// 스펙을 충족하는 엔티티를 가져오는 리포지터리
public interface PlayerRepository extends Repository<Player, String> {
	
    List<Player> findAll(
	    final Specification<Player> spec,
		  final Pageable pageable, **// 페이징을 위해 Pageable 객체를 받는다.**
		  final Sort sort 		     **// 정렬을 해야 한다면 Sort 객체를 받는다.**
		 );
    
    // 조회용 모델을 위한 JPQL 동적 인스턴스 생성
    @Query("""    
            select **new** com.game.player.query.dto.PlayerView(
                s.number, s.state, a.name, a.id
            )
            from Player p join s.skill sk, Ability a
            where p.id = :playerId
            and p.ability.id = a.id
            order by p.id desc
            """)
    List<PlayerView> findPlayerView(final String playerId);
}
...

@Test
void findAll() {
	final Specification<Player> spec = PlayerSpecs.nonBlocked();
	final Pageable pageable = PageRequest.of(1, 2);
	final Sort sort = Sort.by("created_at").descending();
	final List<Player> result = playerRepository.findAll(spec, pageable, sort);
}
```