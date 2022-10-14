# 1. 트랜잭션 isolation 레벨 정리

## Transaction
- 데이터의 정합성을 보장하기 위한 기능
- db read, write작업을 논리적으로 묶는 방식
- 트랜잭션으로 묶은 작업이 성공하면 commit,  실패 시  rollback을 하므로 부분성공, 부분 실패(Partial Update)의 위험이 없음

## **Lock**
- 동시성을 제어하기 위한 기능
- 여러 커넥션에서 동시에 **동일한 자원을 요청할 경우** 순서대로 **하나의 커넥션만 변경**할 수 있게 하는 역할

## Isolation level
- 동시에 하나 또는 여러 트랜잭션이 진행 될때, 다른 트랜잭션에서 변경된 데이터를 조회하는 기준
- 일반적인 서비스에서는 Read Commited, Repeatable Read를 사용

### Read Commited

대부분의 DBMS에서의 기본 설정

- 커밋된 데이터만 읽음
- `dirty read`가 발생하지 않도록 보장
- `Non-Repeatable Read` 발생 가능

### Read Uncommited
- postgresql에서 지원하지 않음
- 각 트랜잭션에서의 변경 내용이 Commit, Rollback 여부 상관없이 다른 트랜잭션에 조회됨
- `dirty read` 발생 가능

### Repeatable Read

- Undo 영역의 백업된 데이터를 이용해 동일 트랜잭션 내에서 같은 결과 보장

- MySql의 InnoDB 에서 기본으로 사용되는 격리 수준

  -   MVCC(Multi Version Concurrency Control)
    - InnoDB스토리지 엔진은 트랜잭션이 Rollback 가능성에 대비해 변경 전 레코드를 Undo 공간에 백업 후 실제 레코드값 변경

- Read Commited도 Undo영역의 백업된 데이터를 보여주지만, 둘의 차이는 백업된 데이터 중 어떤 이전 버전을 보여주는지 차이가 있다.

### Serializable
- 가장 엄격한 격리 수준
- 읽기 작업도 읽기 잠금을 획득해야 하며, 동시에 다른 트랜잭션은 해당 레코드 변경 불가

### 트랜잭션에서 발생 가능한 부정합 현상

1. Non-Repeatable Read
   - 하나의 트랜잭션 내에서 select 쿼리를 실행했을 때 항상 같은 결과가 나오지 않는 현상
   - 격리 수준 설정 시 따라오는 부정합 목록

2. Dirty Read
   - 트랜잭션에서 처리한 작업이 완료(Commit, Rollback)되지 않았을 경우 다른 트랜잭션에서 데이터를 읽는 현상

3. Phantom Read(Phantom Row)
   - 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다 안 보였다 하는 현상
   - select 하는 레코드에 쓰기 잠금을 걸어야 하지만,  **Undo영역 레코드에는 잠금이 불가**