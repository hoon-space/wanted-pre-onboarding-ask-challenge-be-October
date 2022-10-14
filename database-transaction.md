# Week 1-2 과제
> 택일하여 공부 및 정리해보기
> 1. transaction isolation level 정리
> 2. 본인이 사용하는 프레임워크 내 transaction 동작 정리

## Transaction isolation level
> **동시에 여러 트랜잭션이 처리될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지 결정하는 것**  
> READ UNCOMMITTED - READ COMMITTED - REPEATABLE REAd - SERIALIZABLE 순서대로 격리 수준이 높다.

### 1. READ UNCOMMITTED
각 트랜잭션에서 데이터의 변경 내용이 commit, rollback 여부에 상관없이 다른 트랜잭션에서 보여진다.  
-> DIRTY READ가 발생할 수 있다.   
RDBMS 표준에서 트랜잭션의 격리 수준으로 인정하지 않을 정도로 정합성에 문제가 많다.(최소 READ COMMITTED 사용 권장)

- DIRTY READ
    - 트랜잭션에서 처리한 작업이 완료되지 않았음에도 불구하고 다른 트랜잭션에서 볼 수 있게 되는 현상
    - 데이터가 나타났다 사라졌다 하는 현상을 초래할 수 있으므로 개발자와 사용자를 상당히 혼란스럽게 만듦

### 2. READ COMMITTED
어떤 트랜잭션에서 데이터를 변경하더라도 commit이 완료된 데이터만 다른 트랜젝션에서 보여진다.  
-> DIRTY READ가 발생하지 않고, undo record를 사용해 데이터를 보여준다.  
-> NON-REPEATABLE READ가 발생할 수 있다.

- undo record
    - InnoDB 시스템 테이블 스페이스 undo log에 기록
        - undo log: update, delete 같은 문장으로 데이터를 변경했을 때, 변경되기 전의 데이터를 보관하는 곳
    - 트랜잭션 격리 수준을 보장하기 위한 용도 뿐 아니라 트랜잭션의 rollback에 대한 복구에도 사용
  
- NON-REPEATABLE
    - 하나의 트랜잭션 내에서 동일한 select 문장으로 데이터를 조회했을 때, 항상 같은 결과를 보장해야한다는 정합성에 어긋남
    - 금전적인 처리에서 다른 결과를 가져와 문제를 초래할 수 있음
  
### 3. REPEATABLE READ
어떤 트랜잭션에서 데이터를 변경하더라도 commit이 완료된 데이터만 다른 트랜잭션에서 보여진다.  
-> undo record에 변경되기 전 데이터를 백업하고 변경해 NON-REPEATABLE READ가 발생하지 않는다.(MVCC 방식)  
-> PHANTOM READ가 발생할 수 있다.

- MVCC
    - multi version concurrency control, 다중 버전 병행수앵 제어
    - 서로 다른 세션이 동일한 데이터에 접근했을 때, 각 세션마다 스냅샷 이미지를 보장해주는 메커니즘
    - READ COMMITTED와 REPEATABLE READ의 차이는 undo record에 백업된 버전 중 몇 번째 버전까지 찾느냐에 따라 다름
  
- PHANTOM READ(PHANTOM ROW)
    - 다른 트랜잭션에서 수행한 변경 작업에 의해 데이터가 보였다가 안보였다가 하는 현상
    - `SELECT ... FOR UPDATE;'로 임의 데이터에 잠금을 걸었을 때, undo record에는 잠금을 걸 수 없어 다른 데이터 결과를 가져와 문제를 초래할 수 있음
  
### 4. SERIALIZABLE
한 트랜잭션에서 읽고 쓰는 데이터를 다른 트랜잭션에서는 절대 접근할 수 없다.  
-> PHANTOM READ가 발생하지 않는다.  
가장 단순하면서 가장 엄격한 격리 수준이고, 동시 처리 성능이 다른 격리 수준보다 현저히 떨어진다.

### Transaction isolation 변경 방법
```shell
# MYSQL
show variables like '%isolation';
SET [GLOBAL | SESSION] TRANSACTION transaction_characteristic [, transaction_characteristic] ...

# POSTGRESQL
show transaction isolation level;
ALTER SYSTEM SET DEDFAULT_TRANSACTION_ISOLATION TO '[transaction isolation level]';
SET TRANSACTION transaction_mode [, ...]
```

### References
@https://zzang9ha.tistory.com/381  
@https://dar0m.tistory.com/225  
@https://luran.me/325  
@https://luran.me/340
