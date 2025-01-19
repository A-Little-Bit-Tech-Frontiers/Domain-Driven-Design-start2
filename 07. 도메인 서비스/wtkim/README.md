# 7. 도메인 서비스

한 애그리거트 객체가 다른 애그리거트의 기능을 필요로 해서 한 기능을 온전히 책임지기 어려운 경우, 애그리거트에서 직접 구현하지 않고 별도의 도메인 서비스를 구현해서 처리할 수 있다.

- **도메인 서비스를 구현해야 하는 경우**
    - 여러 애그리거트의 협력이 필요하여 한 애그리거트에 넣기 복잡한 로직
    - 외부 API 혹은 타사 서비스 연동이 필요한 도메인 로직

1. **사용 주체가 애그리거트인 도메인 서비스**

```java
// Domain Service
@Service
public class DamageCalculateService {

		public void calculate(final Player player, final Enemy enemy) {
			final int damage = player.getAbility().getDamage();
			enemy.minusHp(damage);
		}
}

// Player Aggregate
public class Player {

    // 데미지를 주는 주체가 플레이어 (사용 주체가 플레이어 애그리거트)
	public void attackToEnemy(
	
		// 애그리거트에서 의존성을 직접 주입하지 않고 도메인 서비스를 외부에서 전달받도록 한다.
		final DamageCalculateService damageCalculateService,
		final Enemy enemy
	) {
		damageCalculateService.calculate(this, enemy);
	}
}

// Application Layer Service
@Service
@RequiredArgsConstructor
public class PlayerAttackFacadeService {
	private final DamageCalculateService damageCalculateService;
	private final PlayerService playerService;
	private final EnemyService enemyService;
	
	@Transactional
	public void attackToEnemy(final UUID playerUuid, final UUID enemyUuid) {
		final Player player = playerService.findByUuid(playerUuid);
		final Enemy enemy = enemyService.findByUuid(enemyUuid);
		
		player.attackToEnemy(damageCalculateService, enemy);
	}
}
```

2. **응용 서비스에서 도메인 서비스를 사용해서 애그리거트 들을 전달하는 경우**

```java
@Service
@RequiredArgsConstructor
public class PlayerAttackFacadeService {
	private final DamageCalculateService damageCalculateService;
	private final PlayerService playerService;
	private final EnemyService enemyService;
	
	@Transactional
	public void attackToEnemy(final UUID playerUuid, final UUID enemyUuid) {
		final Player player = playerService.findByUuid(playerUuid);
		final Enemy enemy = enemyService.findByUuid(enemyUuid);
		
		damageCalculateService.calculate(player, enemy);
	}
}
```

3. **외부 시스템을 연동하는 도메인 서비스를 구현할 경우**
```java
// Infra Service Interface
public interface AnalysisService {
		public void sendAnalysisData(final TodayPlayerBehavior behavior);
}

public class AnalysisServiceImpl implements AnalysisService {

		@Override
		public void sendAnalysisData(final TodayPlayerBehavior behavior) {...}
}

// Domain Service
@Service
@RequiredArgsConstructor
public class AnalysisPlayerBehaviorService {
	private final AnalysisService analysisService;
	
	public void sendAnalysisData(final List<Player> players) {
		final TodayPlayerBehavior behavior = TodayPlayerBehavior.from(players);
		analysisService.sendAnalysisData(behavior);
	}
}

// Application Layer Service
@Service
@RequiredArgsConstructor
public class AnalysisPlayerBehaviorFacadeService {
	private final AnalysisPlayerBehaviorService analysisService;
	private final PlayerService playerService;
	private final EnemyService enemyService;
	
	@Transactional(readonly = true)
	public void analysis() {
		final List<Player> players = playerService.findAll(playerUuid);
		analysisService.sendAnalysisData(players);
	}
}
```