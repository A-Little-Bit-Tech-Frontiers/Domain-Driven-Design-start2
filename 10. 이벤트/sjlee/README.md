## 10.1 시스템 간 강결합 문제

쇼핑몰에서 구매를 취소하면 환불을 처리해야 한다. 이때 환불 기능을 실행하는 **주체**는 **주문 도메인 엔티티**가 될 수 있다. 도메인 객체에서 환불 기능을 실행하려면 아래 코드처럼 환불 도메인 서비스를 주문 도메인에 파라미터로 넘길 수 있다.

```java

public class Order {
	
	// 외부 서비스를 실행하기 위해 도메인 서비스를 파라미터로 전달받음
	public void cancel(RefundService refundService) {
		verifyNotYetShipped();
		this.state = OrderState.CANCELED;
	
		this.refundStatus = State.REFUND_STARTED;
		try {
			refundService.refund(getPaymentId());
			this.refundStatus = State.REFUND_COMPLETED;
		} catch(Exception ex) {
			// ...
		}
	}
...
```

응용 서비스에서 환불 기능을 실행할 수도 있다.

```java
 public class CancelOrderService {
	private RefundService refundService;

	@Transactional
	public void cancel(OrderNo orderNo) {
		Order order = findOrder(orderNo);
		order.cancel();
		
		order.refundStarted();
		try {
			refundService.refund(order.getPaymentIdO); // 외부 서비스 성능에 영향을 받음
			order.refundCompleted();
		} catch(Exception ex) {
			???
		}
...
```

보통 결제 시스템은 외부에 존재하므로 RefundService는 외부에 있는 결제 시스템이 제공하는 환불 서비스를 호출한다.

이때 두 가지 문제가 발생할 수 있다.

**첫 번째 문제**는 외부 서비스가 정상이 아닐 경우 트랜잭션 처리가 애매하다. 환불 기능을 실행하는 과정에서 익셉션이 발생하면 트랜잭션을 롤백해야 할까? 아니면 일단 커밋을 해야 할까?

외부의 환불 서비스를 실행하는 과정에서 익셉션이 발생하면 환불에 실패했으므로 주문 취소 트랜잭션을 롤백하는 것이 맞아보인다. 지만 반드시 트랜잭션을 롤백 해야 하는 것은 아니다. 주문은 취소 상태로 변경하고 환불만 나중에 다시 시도하는 방식으로 처리할 수도 있다.

**두 번째 문제**는 성능에 대한 것이다. 환불을 처리하는 외부 시스템의 응답 시간이 길어지면 그만큼 대기 시간도 길어진다. 환불 처리 기능이 30초가 걸리면 주문 취소 기능은 30초만큼 대기 시간이 증가한다. 즉, **외부 서비스 성능에 직접적인 영향**을 받게 된다.

(개인적으로 생각하는 **세번째 문제** - 다양한 도메인 서비스에서의 **중복된 애그리거트 메서드 호출**)

---

<img width="559" alt="image" src="https://github.com/user-attachments/assets/85722eec-77d6-4aa6-b75c-323c8f39fa39" />


그리고, 도메인 객체에 서비스를 전달하면 추가로 설계상 문제가 나타날 수 있다. 우선,  위처럼 주문 로직과 결제 로직이 섞이는 문제가 있다.

Order는 주문을 표현하는 도메인 객체인데 결제 도메인의 환불 관련 로직이 뒤섞이게 된다. **이것은 환불 기능이 바뀌면 Order도 영향을 받게 된다는 것을 의미**한다. 주문 도메인 객체의 코드를 결제 도메인 때문에 변경할지도 모르는 상황은 좋아 보이지 않는다.

도메인 객체에 서비스를 전달할 시 또 다른 문제는 기능을 추가할 때 발생한다. 만약 주문을 취소한 뒤에 환불뿐만 아니라 취소했다는 내용을 통지해야 한다면 어떻게 될까? 환불 도메인 서비스와 동일하게 파라미터로 통지 서비스를 받도록 구현하면 앞서 언급한 로직이 섞이는 문제가 더 커지고 **트랜잭션 처리가 복잡**해진다. 

결국 이 문제들이 발생하는 이유는 **주문 바운디드 컨텍스트와 결제 바운디드 컨텍스트간의 강결합 때문**이다. 주문이 결제와 강하게 결합되어 있어서 주문 바운디드 컨텍스트가 결제 바운디드 컨텍스트에 영향을 받게 되는 것이다.

이런 강한 결합을 없앨 수 있는 방법이 있다. 바로 이벤트를 사용하는 것이다.
