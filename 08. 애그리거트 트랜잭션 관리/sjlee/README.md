## 8.1 애그리거트와 트랜잭션

한 주문 애그리거트에 대해 운영자는 배송 상태로 변경할 때 사용자는 배송지 주소를 변경하면 어떻게 되는가?

<img width="430" alt="image" src="https://github.com/user-attachments/assets/32631299-8b2c-4efb-a211-4c5ae349130f" />


위 그림은 운영자와 고객이 동시에 한 주문 애그리거트를 수정하는 과정을 보여준다. 트랜잭션마다 리포지터리는 새로운 애그리거트 객체를 생성하므로, 서로 다른 스레드는 같은 주문 애그리거트를 나타내는 다른 객체를 구하게 된다.

운영자 스레드와 고객 스레드는 개념적으로 동일하지만 물리적으로 서로 다른 객체를 사용한다. 때문에 운영자 스레드가 주문 애그리거트 객체를 배송 상태로 변경하더라도 고객 스레드가 사용하는 주문 애그리거트 객체에는 영향을 안준다. 즉 배송 상태와 배송지 변경 사이에 일관성이 깨질 수 있다.

Pessimistic Lock과 Optimistic Lock에 대해 살펴보자.

## 8.2 Pessimistic Lock & Optimistic Lock

먼저 애그리거트를 구한 스레드가 애그리거트 사용이 끝날 때까지 다른 스레드가 해당 애그리거트를 수정하지 못하게 막는 방식이다. 

<img width="437" alt="image" src="https://github.com/user-attachments/assets/87c7a6f7-f48d-4548-9e7f-1d03bbe4a0c5" />


스레드1이 애그리거트를 먼저 구한 뒤 이어서 스레드2가 같은 애그리거트를 구하는 방식이다. DeadLock 발생이 가능하므로,  적절한 타임아웃을 설정하여 하나의 트랜잭션은 예외를 발생시킬 수 있다.

스프링 데이터 JPA는 @QueryHints 애노테이션을 사용해서 쿼리 힌트를 지정할 수 있다.

```java
public interface MemberRepository extends Repository<Member, MemberId> { @Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({
	@QueryHint(name = "javax.persistence.lock.timeout", value = "2000")
})
@Query("select m from Member m where m.id = :id")
Optional<Member> findByIdForUpdate(@Param("id") MemberId memberId);
```

Optimistic Lock은 버저닝을 통해 특정 데이터의 업데이트 가능성을 판단한다.

## 8.2 오프라인 선전 잠금

즉, 한 트랜잭션 범위에서만 적용되는 경우가 아니라 **여러 트랜잭션에 걸쳐서 락을 획득하고 락을 해제하는 경우가 필요할 수 있다.**

예를 들어, 문서 수정 기능을 생각해보자. 수정 기능은 보통 두 개의 트랜잭션으로 구성된다.

- TX 1: 문서 수정 폼 요청
- TX 2: 문서 수정

오프라인 선전 잠금을 사용하면 아래와 같은 컨셉으로 구성할 수 있다.

<img width="588" alt="image" src="https://github.com/user-attachments/assets/acc07ba4-b5a2-4f1f-b3dc-ace9e472d0b4" />


위 그림에서 사용자 A가 과정 3의 수정 요청을 수행하지 않고 프로그램을 종료하면 다른 사용자는 영원히 락을 획득할 수 없는 상황이 발생한다. 때문에 이도 락 유효 시간을 설정해야 한다.

사용자 A가 잠금 유효 시간이 지난 후 1초 뒤에 3번 과정을 수행했다고 가정하자. 그럼 락이 해제되어 사용자 A는 수정에 실패하게 되는데, 이러한 UX까지 고려한다면 일정 주기로 유효 시간을 증가시키는 방식을 적용시켜도 된다.

### 8.2.1 DB를 이용한 LockManager 구현

DB를 이용해 LockManager를 구현할 수 있다. 잠금 정보를 저장할 테이블과 인덱스를 아래와 같이 생성한다.

```sql
create table locks (
	'type' varchar(255),
	id varchar(255),
	lockid varchar(255), expiration_time datetime, primary key ('type', id)
) character set utf8;

create unique index locks_idx ON locks (lockid);
```
