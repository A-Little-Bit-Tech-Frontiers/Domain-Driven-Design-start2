# 2. 아키텍처 개요

### 계층 구조 아키텍처

응용 계층에서 도메인 계층의 의존성을 가질 때, 더 깊은 인프라 계층을 추가적으로 의존해서 편하게 기능을 구현할 수도 있다. 하지만 응용 계층이 인프라 계층을 의존하게 되면 몇가지 문제가 발생한다.

1. 응용 계층 서비스 테스트의 어려움
2. 기능 확장을 위한 구현 방식 변경의 어려움

### DIP

추상화한 인터페이스를 사용하면 저수준 모듈이 고수준 모듈에 의존하도록 바꿀 수 있다. 그렇기 때문에 **DIP, 의존 역전 원칙**으로 부른다.

- **DIP 를 사용한 문제 해결**
    1. 응용 계층 서비스 테스트의 어려움
        ⇒ **의존하는 객체의 인터페이스를 사용해서 대체 객체 주입으로 테스트 가능**
    2. 기능 확장을 위한 구현 방식 변경의 어려움
        ⇒ **다른 데이터베이스를 사용할 경우 인터페이스만 상속받아 기능 구현을 하면 기존과 같은 방식으로 사용 가능**
- **주의사항**
    - DIP 적용을 위한 인터페이스는 **고수준 모듈 관점**에서 도출한다. DB 엔진을 예로 들면, 서비스는 도메인 기능을 구현하기 위해 엔진을 사용하지만 엔진이 직접 연산하는데에는 관심이 없기 때문이다.
- 헥사고날 아키텍처와의 관계성
    - 공유 인터페이스를 **Port** 로 취급하고, 인터페이스를 사용해 명령을 내리는 부분인 고수준 모듈을 **Adaptor** 라고 할 수 있다. 즉, **헥사고날 아키텍처는 이러한 원칙을 사용하기 쉽게 만든 일종의 포맷**인 셈이다.

### 도메인 영역의 주요 구성요소

- 엔티티 (Entity)
- 값 (Value)
- 애그리거트 (Aggregate)
- 리포지터리 (Repository)
- 도메인 서비스 (Domain Service)

### 애그리거트

어떤 개념에 관련된 하위 객체를 하나로 묶어 상위 개념으로 표현하는 것으로, 도메인 모델의 전체 구조를 이해하는데 도움을 준다.

애그리거트는 **루트 도메인 모델 엔티티**를 가진다. 이 엔티티에서 하위 엔티티 및 값 객체를 사용해 애그리거트 기능을 제공한다. (Facade 와 비슷하게 내부 기능을 숨겨 구현의 캡슐화를 도와준다)

### 리포지터리

데이터를 물리적인 저장소에 영속화 하기 위한 도메인 모델이다.

리포지터리는 애그리거트 단위로 도메인 객체의 기능을 정의하고, 관점에 따라 두 가지로 분류된다.

1. 도메인 객체를 영속화 하기 위해 필요한 기능을 추상화 한 리포지터리는 고수준 모듈에 속한다.
2. 인프라 기술을 이용해 구현한 리포지터리는 인프라 계층에 속한다.

### 인프라와 응용 계층

스프링의 `@Transactional` 기능은 트랜잭션을 관리하는 좋은 기능이다. 만약 인프라 의존성을 지우기 위해 이러한 기능을 사용하지 않고, 프레임워크의 틀에서 엄격하게 도메인 주도 설계를 하게 되면 코드가 오히려 복잡해지고 프레임워크 기능에 맞추기 위해 복잡한 설정이 추가될 뿐이다.

### 모듈 구조 예시 - [**출처**](https://github.com/madvirus/ddd-start2)
```
.
├── README.md
├── docker.md
├── mvnw
├── mvnw.cmd
├── pom.xml
├── result.txt
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── myshop
    │   │           ├── ShopApplication.java
    │   │           ├── admin
    │   │           │   └── ui
    │   │           │       └── AdminOrderController.java
    │   │           ├── board
    │   │           │   └── domain
    │   │           │       ├── Article.java
    │   │           │       ├── ArticleContent.java
    │   │           │       └── ArticleRepository.java
    │   │           ├── catalog # 애그리거트 도메인 #
    │   │           │   ├── NoCategoryException.java
    │   │           │   ├── command
    │   │           │   │   └── domain
    │   │           │   │       ├── category # 하위 도메인 (애그리거트, 모델, 리포지터리는 한곳에)
    │   │           │   │       │   ├── Category.java
    │   │           │   │       │   ├── CategoryId.java
    │   │           │   │       │   └── CategoryRepository.java
    │   │           │   │       └── product # 하위 도메인
    │   │           │   │           ├── ExternalImage.java
    │   │           │   │           ├── Image.java
    │   │           │   │           ├── InternalImage.java
    │   │           │   │           ├── Option.java
    │   │           │   │           ├── Product.java
    │   │           │   │           ├── ProductId.java
    │   │           │   │           └── ProductRepository.java
    │   │           │   ├── query
    │   │           │   │   ├── category
    │   │           │   │   │   ├── CategoryData.java
    │   │           │   │   │   └── CategoryDataDao.java
    │   │           │   │   └── product
    │   │           │   │       ├── CategoryProduct.java
    │   │           │   │       ├── ImageData.java
    │   │           │   │       ├── ProductData.java
    │   │           │   │       ├── ProductDataDao.java
    │   │           │   │       ├── ProductQueryService.java
    │   │           │   │       └── ProductSummary.java
    │   │           │   └── ui
    │   │           │       └── ProductController.java
    │   │           ├── common
    │   │           │   ├── ValidationError.java
    │   │           │   ├── ValidationErrorException.java
    │   │           │   ├── VersionConflictException.java
    │   │           │   ├── event
    │   │           │   │   ├── Event.java
    │   │           │   │   ├── EventStoreHandler.java
    │   │           │   │   ├── Events.java
    │   │           │   │   └── EventsConfiguration.java
    │   │           │   ├── jpa
    │   │           │   │   ├── EmailSetConverter.java
    │   │           │   │   ├── MoneyConverter.java
    │   │           │   │   ├── Rangeable.java
    │   │           │   │   ├── RangeableExecutor.java
    │   │           │   │   ├── RangeableRepository.java
    │   │           │   │   ├── RangeableRepositoryImpl.java
    │   │           │   │   └── SpecBuilder.java
    │   │           │   ├── model
    │   │           │   │   ├── Address.java
    │   │           │   │   ├── Email.java
    │   │           │   │   ├── EmailSet.java
    │   │           │   │   └── Money.java
    │   │           │   └── ui
    │   │           │       └── Pagination.java
    │   │           ├── eventstore
    │   │           │   ├── api
    │   │           │   │   ├── EventEntry.java
    │   │           │   │   ├── EventStore.java
    │   │           │   │   └── PayloadConvertException.java
    │   │           │   ├── infra
    │   │           │   │   └── JdbcEventStore.java
    │   │           │   └── ui
    │   │           │       └── EventApi.java
    │   │           ├── integration
    │   │           │   ├── EventForwarder.java
    │   │           │   ├── EventSender.java
    │   │           │   ├── OffsetStore.java
    │   │           │   └── infra
    │   │           │       ├── MemoryOffsetStore.java
    │   │           │       └── SysoutEventSender.java
    │   │           ├── lock
    │   │           │   ├── AlreadyLockedException.java
    │   │           │   ├── LockData.java
    │   │           │   ├── LockException.java
    │   │           │   ├── LockId.java
    │   │           │   ├── LockManager.java
    │   │           │   ├── LockManagerException.java
    │   │           │   ├── LockingFailException.java
    │   │           │   ├── NoLockException.java
    │   │           │   └── SpringLockManager.java
    │   │           ├── member
    │   │           │   ├── command
    │   │           │   │   ├── application
    │   │           │   │   │   ├── BlockMemberService.java
    │   │           │   │   │   └── NoMemberException.java
    │   │           │   │   └── domain
    │   │           │   │       ├── IdPasswordNotMatchingException.java
    │   │           │   │       ├── Member.java
    │   │           │   │       ├── MemberBlockedEvent.java
    │   │           │   │       ├── MemberId.java
    │   │           │   │       ├── MemberRepository.java
    │   │           │   │       ├── MemberUnblockedEvent.java
    │   │           │   │       ├── Password.java
    │   │           │   │       └── PasswordChangedEvent.java
    │   │           │   ├── infra
    │   │           │   │   └── PasswordChangedEventHandler.java
    │   │           │   └── query
    │   │           │       ├── MemberData.java
    │   │           │       ├── MemberDataDao.java
    │   │           │       ├── MemberDataSpecs.java
    │   │           │       └── MemberQueryService.java
    │   │           ├── order
    │   │           │   ├── NoOrderException.java
    │   │           │   ├── command
    │   │           │   │   ├── application
    │   │           │   │   │   ├── CancelOrderService.java
    │   │           │   │   │   ├── ChangeShippingRequest.java
    │   │           │   │   │   ├── ChangeShippingService.java
    │   │           │   │   │   ├── NoCancellablePermission.java
    │   │           │   │   │   ├── NoOrderProductException.java
    │   │           │   │   │   ├── OrderProduct.java
    │   │           │   │   │   ├── OrderRequest.java
    │   │           │   │   │   ├── OrderRequestValidator.java
    │   │           │   │   │   ├── PlaceOrderService.java
    │   │           │   │   │   ├── RefundService.java
    │   │           │   │   │   ├── StartShippingRequest.java
    │   │           │   │   │   └── StartShippingService.java
    │   │           │   │   └── domain
    │   │           │   │       ├── AlreadyShippedException.java
    │   │           │   │       ├── CancelPolicy.java
    │   │           │   │       ├── Canceller.java
    │   │           │   │       ├── Order.java
    │   │           │   │       ├── OrderAlreadyCanceledException.java
    │   │           │   │       ├── OrderCanceledEvent.java
    │   │           │   │       ├── OrderLine.java
    │   │           │   │       ├── OrderNo.java
    │   │           │   │       ├── OrderPlacedEvent.java
    │   │           │   │       ├── OrderRepository.java
    │   │           │   │       ├── OrderState.java
    │   │           │   │       ├── Orderer.java
    │   │           │   │       ├── OrdererService.java
    │   │           │   │       ├── Receiver.java
    │   │           │   │       ├── ShippingInfo.java
    │   │           │   │       ├── ShippingInfoChangedEvent.java
    │   │           │   │       └── ShippingStartedEvent.java
    │   │           │   ├── infra
    │   │           │   │   ├── OrderCanceledEventHandler.java
    │   │           │   │   ├── OrdererServiceImpl.java
    │   │           │   │   ├── domain
    │   │           │   │   │   └── SecurityCancelPolicy.java
    │   │           │   │   └── paygate
    │   │           │   │       └── ExternalRefundService.java
    │   │           │   ├── query
    │   │           │   │   ├── application
    │   │           │   │   │   ├── ListRequest.java
    │   │           │   │   │   ├── OrderDetail.java
    │   │           │   │   │   ├── OrderDetailService.java
    │   │           │   │   │   ├── OrderLineDetail.java
    │   │           │   │   │   └── OrderViewListService.java
    │   │           │   │   ├── dao
    │   │           │   │   │   ├── OrderSummaryDao.java
    │   │           │   │   │   ├── OrderSummarySpecs.java
    │   │           │   │   │   └── OrdererIdSpec.java
    │   │           │   │   └── dto
    │   │           │   │       ├── OrderSummary.java
    │   │           │   │       ├── OrderSummary_.java
    │   │           │   │       └── OrderView.java
    │   │           │   └── ui
    │   │           │       ├── CancelOrderController.java
    │   │           │       ├── MyOrderController.java
    │   │           │       ├── OrderController.java
    │   │           │       └── OrderRequestValidator4Spring.java
    │   │           └── springconfig
    │   │               ├── security
    │   │               │   ├── CookieSecurityContextRepository.java
    │   │               │   ├── CustomAuthSuccessHandler.java
    │   │               │   └── WebSecurityConfig.java
    │   │               └── web
    │   │                   └── WebMvcConfig.java
    │   └── ...
    └── ...
```
