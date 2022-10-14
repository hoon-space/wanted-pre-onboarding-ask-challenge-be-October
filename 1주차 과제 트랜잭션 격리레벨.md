| 격리 수준        | dirty read | non-repeatable read | phantom read   |
| ---------------- | ---------- | ------------------- | -------------- |
| READ UNCOMMITTED | O          | O                   | O              |
| READ COMMITTED   |            | O                   | O              |
| REPEATABLE READ  |            |                     | O(InnoDB 제외) |
| SERIALIZABLE     |            |                     |                |


### phantom read
내 조회쿼리가 실행되던 중 커밋 이전에 INSERT, DELETE, UPDATE 트랜잭션이 커밋될 경우 원치 않는 결과가 더해져서 반환되는 경우이며 SERIALIZABLE 격리 수준일 경우 조회쿼리 중 INSERT,DELETE,UPDATE 쿼리는 막혀 해당 현상이 발생하지 않는다.

## read uncommitted
각 트랜잭션의 변경내용이 롤백이나 커밋 여부에 관계없이 해당 시점에 보이게 됩니다.

### dirty read
커밋이나 롤백될 경우 중간시점에 데이터를 조회하거나 변경하는 다른 쿼리의 시점에서는 다양한 버그를 초래하며 이것을 dirty read라고 합니다.

따라서 예상하지 않은 오류가 발생할 확률이 매우 높습니다.
마치 쓰레드의 동시성 오류처럼 결제나 수치변화가 잦은 정보의 경우 피해야하는 격리수준입니다.

## read committed
오라클에서 디폴트로 사용되는 격리 수준입니다.
기본적으로 쿼리진행시 조회할 데이터는 commit이 완료된 데이터로 한정합니다.
따라서 dirty read가 발생하지 않습니다.
커밋 전까지 다른 쿼리가 해당 데이터를 조회시 rollback을 위한 undo영역에 백업한 원래 데이터를 제공합니다.

## repeatable read
InnoDB엔진에서 기본적으로 제공하는 격리 수준입니다. 롤백 이전의 데이터를 저장하는 undo영역의 데이터를 기준으로 쿼리를 계속해서 진행하는 방식입니다. 이로 인해 트랜잭션 중간에 별도의 변경쿼리가 커밋되더라도 해당 데이터가 아닌 시작시점의 undo 데이터를 기준으로 쿼리를 진행하기 때문에 nonrepeatable read가 발생하지 않습니다. 

하지만 마찬가지로 중간에 새로운 row가 추가되거나 삭제되는 트랜잭션이 커밋된 경우 undo영역과는 별개로 새로운 row도 조회에 들어가기 때문에 phantom read가 발생합니다. 하지만 InnoDB의 경우 특이한 undo 관리방식으로 phantom read가 발생하지 않습니다.

### non repeatable read
read committed에서 한트랜잭션에서 최초 조회 이후 중간에 다른 변경쿼리가 commit된 경우 이후 쿼리를 진행을 할 때 원래 데이터와 다른 데이터를 기준으로 연산을 하게되는데 이를 non repeatable read라고 합니다.

### 주의 
이 격리 수준의 경우 트랜잭션의 길이가 길어질 수록 undo에 저장되는 데이터가 늘어나기 때문에 성능저하가 일어날 수 있습니다.

## serealizable
한 레코드에 대해서 한 트랜잭션만 동시에 접근할 수 있도록 락을 겁니다.
뮤텍스와 비슷한 방법으로 앞서 설명한 문제들이 모두 발생하지 않지만 성능저하가 발생할 수 있습니다.
