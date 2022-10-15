# 스프링에서 트랜젝션을 다루는 방법

## 트랜젝션 관리 2가지 방식
- Global Transaction : 여러 자원들(relational databases and message queues) 간의 트랜젝션 관리
  - JTA(Java Transaction API), EJB CMT (Container Managed Transaction)를 사용
- Local Transaction : 하나의 자원에 대한 트랜젝션 관리
  - JDBC Connection을 통한 단일 DBMS 트랜젝션 처리

## Spring Framework Transaction Abstraction
> 스프링에서는 트랜젝션 전략을 추상화하여 제공하며 최상위 인터페이스는 org.springframework.transaction.TransactionManager이다. 
> 
> Transaction을 적용할 자원이 DBMS인지, Message Queue인지는 이 추상화 계층에서는 알 수 없다.
- `TransactionManager`는 mark interface이다.
- 실질적인 기능은 `TransactionManager`를 상속받은 `PlatformTransactionManager`,`ReactiveTransactionManager` 에서 확인할 수 있다. 

### 1. platformTransactionManager.getTransaction
트랜젝션 전파방식에 따라 현재 transaction을 반환하거나, 새로운 transaction을 생성하는 역할을 수행
  > `TransactionStatus` getTransaction(@Nullable `TransactionDefinition` definition)
    throws TransactionException

- #### TransactionStatus
  - 트랜젝션의 상태정보를 나타내는 객체
  - isCompleted, isReadOnly, isRollbackOnly, hasSavePoint 등의 메서드를 제공한다.
  - 또한 SavePoint를 생성하고 관리하는 역할도 가지고 있다.

- #### TransactionDefinition
  - 새로운 트랜젝션 생성을 위한 트랜젝션 속성정보를 담은 객체
    - ReadOnly
    - PropagationBehavior
    - IsolationLevel
    - Timeout
  - 만일 getTransaction에서 기존의 트랜젝션을 반환하는 경우엔 적용되지 않는다.

### 2. AbstractPlatformTransactionManager
  - 기본적인 transaction workflow를 제공한다. 특히 테스트시 사용되는 rollback only flag를 검사하는 기능 및 
    - determines if there is an existing transaction
    - applies the appropriate propagation behavior
    - suspends and resumes transactions if necessary
    - checks the `rollback-only flag` on commit
    - applies the appropriate modification on rollback (actual rollback or setting rollback-only)
    - triggers registered `synchronization` callbacks (if transaction synchronization is active)
        > 트랜잭션 동기화는 트랜잭션에 포함된 모든 Task간에 동일한 Resource를 사용 하기 위한 기술이다. 
        > 예를 들어 동일한 JDBC Connection 혹은 동일한 Hibernate Session을 트랜젝션 내에서 사용할 수 있다.  
  - 서비스 제공자는 이 클래스를 상속받아 template method를 구현하면 된다.
      #### 예시) org.springframework.orm.jpa.JpaTransactionManager
        JPA를 사용할때의 TransactionManager로써 AbstractPlatformTransactionManager를 재정의한 클래스
        DataSource 및 JPA의 EntityManager를 가지고 있다.

### 3. 트랜젝션 전파 전략(TransactionDefinition)
  - PROPAGATION_REQUIRED: 기본값. 현재 트랜젝션을 사용한다. 만일 현재 트랜젝션이 없는경우 새로 만든다. 
  - PROPAGATION_SUPPORTS: 현재 트랜젝션을 사용하되, 없으면 트랜젝션과 무관하게 task를 실행한다. 
  - PROPAGATION_MANDATORY: 현재 트랜젝션을 사용한다. 만일 현재 트랜젝션이 없는경우 예외를 반환한다.
  - PROPAGATION_REQUIRES_NEW: 항상 새로운 트랜젝션을 생성한다. 기존 트랜젝션은 suspending 된다.
  - PROPAGATION_NOT_SUPPORTED: 항상 트랜젝션과 무관하게 task를 처리한다.  
  - PROPAGATION_NEVER: 현재 트랜젝션이 있다면 예외를 발생시킨다.
  - PROPAGATION_NESTED: 현재 트랜젝션이 있다면 nested transaction에서 task를 실행한다. 현재 트랜젝션이 없는경우 새로 만든다.

### 4. 트랜젝션 격리 수준(TransactionDefinition)
  - ISOLATION_DEFAULT: DataStore의 기본 트랜젝션 격리 수준을 사용한다.
  - ISOLATION_READ_UNCOMMITTED: 커밋되지 않은 다른 트랜젝션에서 변경된 데이터도 읽는다.
    - dirty read 현상이 발생할 수 있다. 
  - ISOLATION_READ_COMMITTED: 트랜젝션 실행중 다른 트랜젝션에서 커밋된 데이터만 읽는다.
    - non-repeatable reads: 중간에 다른 트랜젝션에 의해 데이터가 바뀐경우 동일한 SELECT문의 결과가 달라질 수 있다.
    - phantom read: 중간에 다른 트랜젝션에 의해 데이터가 `추가`된 경우 동일한 WHERE절을 가진 SELECT문의 결과 ROW수가 달라질 수 있다. 
  - ISOLATION_REPEATABLE_READ: non-repeatable reads를 방지한다. 
    - phantom read 현상이 여전히 발생할 수 있다. 
  - ISOLATION_SERIALIZABLE
    - dirty read, non-repeatable reads, phantom read 는 사라지지만, 트랜젝션 동시 처리능력이 감소한다. 

### 5. @Transactional
  - 주로 Application Service Layer에서 사용한다.
  - 트랜젝션과 관련된 속성들을 명시할 수 있다. 
    - Rollback rule, 트랜젝션 격리, 트랜젝션 전파, timeout
  - org.springframework.transaction.annotation.SpringTransactionAnnotationParser에 의해 parsing된다.

# 참고 링크 
- https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction
- https://mangkyu.tistory.com/154
- Spring Transaction 소스 내 주석
