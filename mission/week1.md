# 1. 트랜잭션 isolation 레벨 정리

## 트랜젝션 격리(Isolation)

- 트랜잭션을 서로 격리해서 다른 트랜잭션이 영향을 주지 못하게 함

## Read Uncommitted

- dirty read, dirty write 발생

## Read Committed

- 커밋된 값과 트랜잭션 진행 중인 값을 따로 보관
- 행 단위로 잠금 사용해서 같은 데이터를 수정한 트랜잭션이 끝날 때까지 대기

## Repeatable Read

- 트랜잭션 동안 같은 데이터를 읽게 하여 읽는 시점에 따라 데이터가 변경(read skew)되 문제를 없앰.

## Serializable

- 인덱스 잠금이나 조건 기반 잠금 등 사용

## 변경 유실을 막기 위한 3가지 방법

1. 원자적 연산 - DB가 지원하는 원자적 연산 사용
2. 명시적 잠금 - 조회활 때 수정할 행을 미리 잠금
3. CAS - 수정할 때 값이 같은지 비교

# 2. 본인이 사용하는 프레임워크에서 Transaction 기능이 어떻게 동작하는지 정리

저는 스프링 부트를 사용하기 때문에 스프링 부트에서 사용되는 Transaction 기능에 대해 정리하였습니다.

1. JDBC api 에서의 트랜잭션의 문제점
- 깔끔하던 Service 코드가 복잡해진다.
- 데이터 액세스 기술에 의존적인 코드를 작성한다
- 비즈니스 로직과는 다른 관심사의 일을 수행한다.
```java
public void move(MoveRequest moveRequest)throws SQLException{
        Connection conn=getConnection();
        conn.setAutoCommit(false);
        try{
                moveLogic(conn,moveRequest);
                conn.commit();
            }catch(Exception e){
                conn.rollback();
            }finally{
                conn.setAutoCommit(true);
                conn.close();
            }
        }
```
2. 스프링 트랜잭션으로 전환
- 더 이상 Connection을 파라미터로 받지 않는다.
- Service에서 생성한 Connection을 저장한다.
```java
public void move(MoveRequest moveRequest)throws SQLException{
        TransactionSynchronizationManger.initSynchronization();
        Connection conn= DataSourceUtils.getConnection(dataSource);
        conn.setAutoCommit(false);
        try{
                moveLogic(moveRequest);
                conn.commit();
            }catch(Exception e){
                conn.rollback();
            }finally{
                DataSourceUtils.releaseConnection(conn, dataSource);
                TransactionSynchronizationManager.unbindResource(dataSource);
                TransactionSynchronizationManager.clearSynchronization();
            }
        }
```

3. 스프링 트랜잭션의 특징
   1. 트랜잭션 동기화
      1. TransactionSynchronizationManger에 담아서 트랜잭션을 동기화 한다.
   2. 트랜잭션 추상화
      1. 데이터 접근형태마다 다른 인터페이스를 제공한다.
   3. 선언적 트랜잭션
      1. AOP 프록시를 통해 활성화되고 트랜잭션 관련 메타데이터를 참조하여 생성
      2. 부가 기능을 어노테이션으로 대체해 비즈니스 로직만 코드에 담을수 있다.
   
4. 스프링 트랜잭션의 흐름
트랜잭션 시작 -> 타겟 메서드 실행 -> 트랜잭션 종료