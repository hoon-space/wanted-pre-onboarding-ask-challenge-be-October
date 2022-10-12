[velog link](https://velog.io/@peppermint100/Database-Isolation-Level)
## 격리 수준의 종류
격리 수준의 종류는 4가지가 있다.
```
1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE
```
현재 자신이 사용하는 데이터베이스의 격리 수준을 보고싶다면 show variables like '%tx%'; 쿼리를 날리면 확인 할 수 있다.

각각 트랜잭션에 따른 db의 쿼리 결과가 달라지고 일반적으로 2, 3을 사용한다고 한다.

이유는 1, READ UNCOMMITTED는 서비스에 적합하지 않을 정도로 격리수준이 너무 낮고, SERIALIZABLE은 서비스에 적합하지 않을 정도로 성능이 좋지 않다고 한다.

1~4로 갈수록 격리 수준이 높아지고, 성능은 안좋아진다.

## READ UMCOMMITTED
커밋되지 않은 읽기 라는 의미를 가지고 있다. 어떤 Row에 대해 update문을 포함한 트랜잭션이 있고 그 트랜잭션이 커밋되기전에 해당 Row를 읽으면 update가 된 상태의 값을 읽어온다.

즉 커밋되지 않은 트랜잭션의 결과도 읽어온다는 의미이다.
```sql
# session1
START TRANSACTION;
UPDATE application_user SET name="newname" WHERE user_id=1

# session2
SELECT name FROM application_user WHERE user_id=1; // 결과 newname

# session1
COMMIT:
```
트랜잭션이 실패할 가능성이 있음에도 이미 업데이트 된 결과를 불러오기 때문에 서비스에 적합하지 않다. 이를 Dirty Read 라고 한다.

## READ COMMITED
커밋된 읽기 라는 의미로 Update를 포함한 트랜잭션 중간에 Row의 값을 읽어가는 경우에는 Update 이전의 값, 즉 커밋되기 이전의 값을 가져간다는 의미이다.
```sql
# session1
START TRANSACTION;
UPDATE application_user SET name="newname" WHERE user_id=1

# session2
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname

# session1
COMMIT:

# session2
SELECT name FROM application_user WHERE user_id=1; // 결과 newname
```
READ COMMITTED에서는 update 쿼리가 날아오면 Row를 업데이트하고 이전 값을 UNDO 영역에 넣어둔다. 그리고 같은 Row에 접근하면 UNDO 영역에 있는 값을 보여준다.

또 롤백시엔 UNDO 영역에 있는 값으로 롤백을 해주는 방식으로 작동한다.

정상적인 비즈니스 로직을 떠올릴 때 대부분 이 방식이 맞아 떨어진다.

하지만 이 방식에도 허점은 있다.
```sql
# session1
START TRANSACTION-1;
UPDATE application_user SET name="newname" WHERE user_id=1

# session2
START TRANSACTION-2
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname

# session1
COMMIT TRASACTION-1:

# session2
SELECT name FROM application_user WHERE user_id=1; // 결과 newname
COMMIT TRASACTION-2;
```
session2의 결과만 보면 READ COMMITTED는 TRASACTION-2, 즉 한 트랜잭션에서 같은 쿼리의 값이 다른 결과를 보여주게 된다. 이를 NON_REPEATABLE READ 라고 한다.

## REPEATABLE READ
REPEATABLE READ는 가장 기본적으로 사용되는 격리수준이다. 이 방식은 위에서 발생하는 NON_REPEATABLE READ 문제를 해결할 수 있다.

REPEATABLE READ 에서는 서로 다른 세션들이 한 Row에 접근했을 때 각 세션마다 스냅샷 이미지를 보장해준다. 각 트랜잭션 마다 번호를 매겨서 UNDO 영역에 모든 변경을 저장하고 DB 엔진이 불필요하다고 판단하는 시점에 UNDO 영역의 스냅샷 이미지를 주기적으로 삭제해준다.

```sql
# session1
START TRANSACTION-1;
UPDATE application_user SET name="newname" WHERE user_id=1

# session2
START TRANSACTION-2
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname

// session-2의 스냅샷 생성 oldname

# session1
COMMIT TRASACTION-1:

# session2
// session-2에 저장된 스냅샷으로 부터 값을 가져옴
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname
COMMIT TRASACTION-2;
```
근데 여기서도 문제가 발생할 수 있다.

```sql
# session2
START TRANSACTION-2
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname
// session-2의 스냅샷 생성 oldname

# session1
START TRANSACTION-1;
DELETE FROM application_user WHERE user_id=1

# session1
COMMIT TRASACTION-1:

# session2
// session-2에 저장된 스냅샷으로 부터 값을 가져옴
SELECT name FROM application_user WHERE user_id=1; // 결과 oldname
COMMIT TRASACTION-2;
```
이렇게 한 트랜잭션 내에서 유저를 가져오고 그 사이에 delete 쿼리로 지운 다음에 session 2의 트랜잭션에서 다시 유저를 가져오면 UNDO 영역에서 지운 유저를 가져오는 문제가 발생할 수 있다.

삭제했는데 유저가 존재하는 것이다. 이를 Phantom Read 라고 한다.


## SERAIALIZABLE
가장 엄격한 격리 수준으로 직렬화라고 한다. 읽기작업, 쓰기작업 다른 트랜잭션이 진행중일 때 해당 레코드에 접근할 수 없게 된다.

이렇게 하면 위 Phantom Read 문제도 발생하지 않지만 동시처리 성능이 많이 떨어지기 때문에 보통 사용하지 않는다.
