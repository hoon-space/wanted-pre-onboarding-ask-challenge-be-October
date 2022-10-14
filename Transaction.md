# 2회차 과제 Transaction Isolation

## 트랜잭션의 격리 수준(isonation)이란?
> 동시에 트랜잭션이 처리될 때 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있도록 허용할지 말지를 결정하는것

## 트랜잭션 격리 수준(Isolation Level)의 종료
* READ UNCOMMITTED
* READ COMMITTED
* REPEATABLE READ
* SERIALIZABLE

### 1. READ UNCOMMITTED
* 아직 `Commit`되지 않은 데이터를 다른 트랜잭션이 읽는것을 허용한다.
* 정합성에 문제가 많은 격리 수준이기 때문에 사용하지 않는 것을 권장한다.
* `DIRTY READ`현상 발생, 트랜잭션이 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있게 되는 현상을 말한다.

### 2. READ COMMITTED
* RDB에서 대부분 기본적으로 사용되고 있는 격리 수준이다.
* `DIRTY READ`와 같은 현상은 발생하지 않는다.
* 데이터를 변경할 때 이전 데이터를 Undo 영역에 백업해 두었다가 커밋 전 데이터 조회가 들어오면 Undo 영역의 데이터를 보여준다.
* `NON-REPEATABLE READ`현상 발생, 한 트랜잭션에서 같은 쿼리를 두 번 수행할 때, 두 쿼리의 결과가 상이하게 나타나는 비 일관성 현상을 말한다.
* 한 트랜잭션이 Commit 하지 않은 상태에서 다른 트랜잭션이 값을 수정 또는 삭제함으로써 나타나는 현상이다.

 ### 3. REPEATABLE READ
 * 트랜잭션이 `ROLLBACK`될 가능성에 대하여 전 레코드를 Undo 영역에 백업핻두고 실제 레코드 값을 변경한다.
 * 이러한 변경방식을 MVCC(Multi Version Concurrency Control)라한다.
 <p align="center"><img src="https://user-images.githubusercontent.com/104122924/195833654-15a6fc8f-1bea-404e-9250-5cdbe3ffb90c.png" width="85%" height="auto"/></p>

 * Undo 영역에 백업된 이전 데이터를 통해 통일한 트랜잭션 내에서 동일한 결과를 보여준다.
 * `PHANTOM READ`현상 발생, 한 트랜잭션이 수행중일 때 다른 트랜잭션이 새로운 데이터를 생성함으로써 나타나는 현상이다.

### 4. SERIALIZABLE
* 가장 단순한 격리 수준이지만 가장 엄격한 격리수준이다.
* 한 트랜잭션에서 읽고 쓰는 데이터를 다른 트랜잭션에서 접근할 수 없다.
* 성능 측면에서는 동시 처리성능이 가장 낮다.
