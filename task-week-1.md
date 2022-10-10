## 1. 트랜잭션 isolation 레벨 정리


트랜잭션 격리레벨(Isolation Level)이란,                   
여러 개의 트랜잭션이 동시에 수행될 때, 트랜잭션끼리의 고립 수준을 나타내는 정도입니다.                       
특정 트랜잭션이 다른 트랜잭션에서 변경한 데이터를 어느정도까지 볼 수 있도록 허용할지 설정할 수 있습니다.                     

Spring에서는 이 격리 레벨을 의미하는 Isolation Enum이 있고 `@Transactional` 어노테이션의 속성으로 지정하여 사용할 수 있습니다.

<div align="center">
  <img src="/img/isolation.png">
</div>

### 1. DEFAULT

```
- 사용중인 데이터소스에 설정된 격리수준을 그대로 따릅니다. 속성에 isolation을 따로 지정하지 않았다면 이 설정이 적용됩니다.
- 대부분의 데이터베이스는 READ_COMMITED를 기본으로 사용하고 있습니다.
```

### 2. READ_COMMITED

```
- COMMIT이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있습니다.
```

### 3. READ_UNCOMMITED

```
- READ_COMMITED의 반대입니다. COMMIT / ROLLBACK에 상관없이 다른 트랜잭션에서 변경내용의 조회가 가능합니다.
- 더티 리드(Dirty Read) 현상이 발생할 수 있고, 데이터의 일관성을 유지하는 것이 불가능합니다.
- 성능상 가장 빠르지만, 가장 위험합니다.
```

### 4. REPEATABLE_READ
```
- 테이블의 같은 행에 대해 두 가지 이상 다른 행위를 하는 트랜잭션들이 있을 때,
  한번 읽은 행은 반복적으로 조회를 시도하여도 다른 트랜잭션의 변경 결과를 읽어오지 않습니다.
  즉, 트랜잭션이 시작되기 전 커밋된 내용에 대해서만 조회할 수 있습니다.
- 데이터를 추가하는 것에 대해서는 따로 제한을 두지 않아서, 새롭게 추가된 행이 나타날 수 있습니다.(Phantom read)
```

### 5. SERIALIZABLE
```
- 가장 강력한 격리수준입니다.
- 동시에 여러 트랜잭션이 하나의 테이블에 접근할 수 없습니다.
  각 트랜잭션을 먼저 도착한 순서대로 수행하고, 이전 트랜잭션이 완료된 후에 다음 트랜잭션을 수행합니다.
- 가장 안전하지만, 성능면에서는 가장 떨어지기 때문에 정말 안전해야 하는 상황이 아니라면 잘 사용하지 않습니다.
```

## 2. Spring에서의 Transaction 동작 원리

Spring에서 트랜잭션 처리를 위해 제공하는 `@Transactional` 어노테이션은 Spring AOP를 사용합니다.                 
Spring AOP는 기본적으로 Proxy 기반으로 동작합니다.

> proxy : 대리(행위)나 대리권, 대리 투표, 대리인 등을 의미

proxy란, 어떤 객체에 접근할 때, 객체를 직접 참조하는 것이 아닌, 해당 객체를 대행하는 proxy 객체를 통해 접근하는 방식을 의미합니다.                
Spring AOP는 `JDK Dynamic Proxy`(Spring의 default)와 `CGLib Proxy`(Spring Boot의 default)를 제공합니다.                

**JDK Dynamic Proxy**
```
Java Reflection API를 통해 생성된 Proxy객체를 의미합니다.
java.lang.reflect.Proxy 클래스가 동적으로 Proxy객체를 생성해 줍니다.
Target의 인터페이스를 기준으로 생성하기 때문에, 상위 인터페이스가 존재해야 합니다.
```

**CGLib Proxy**
```
Reflection을 이용하지 않습니다. 따라서 JDK Dynamic Proxy에 비해 성능 상 이점이 있습니다.
상속을 통해 Proxy화할 메소드를 오버라이딩 합니다.
```

### @Transactional의 동작 과정

<div align="center">
  <img src="/img/transaction_process.png">
</div>

- Caller에서 Proxy를 호출합니다. 이 때 Target은 호출하지 않습니다.
- AOP Proxy가 Transaction Advisor를 호출합니다. 이때 COMMIT / ROLLBACK이 수행됩니다.
- Custom Advisor가 있다면, Transaction Advisor 실행 전/후에 Custom interceptor가 실행됩니다.
- Custom Advisor가 Target메소드를 호출하고, 비즈니스 로직이 수행됩니다.
- 결과를 순서대로 리턴합니다.
