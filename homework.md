### 1. 트랜잭션 isolation 레벨 정리

ANSI SQL의 isolation 레벨은 4개로 구분된다.

read uncommitted / read committed / repeatable read / serializable

단계가 오를 수록 동시성은 제한되고 격리성을 얻게 된다.

read uncommitted 의 경우 commit 되지 않는 데이터를 조회하게 되는 dirty read 현상이 발생하고

read committed 의 경우 한 트랜잭션에서 단일 쿼리의 결과가 다르게 동작하는 nonrepeatable read 현상이 발생한다.

repeatable read 의 경우 한 트랜잭션에서 복수 쿼리의 결과가 다르게 동작하는 phantom read 현상이 발생한다.

> phantom read는 non-repeatable read의 특별한 경우입니다. [위키피디아](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29#Non-repeatable_reads:~:text=The%20phantom%20reads%20anomaly%20is%20a%20special%20case%20of%20Non%2Drepeatable%20reads)

mysql InnoDB 엔진의 경우 gap lock과 record lock을 이용해 쿼리가 발생한 range에 잠금을 걸어서 phantom read를 방지한다. 즉, 이후에 접근하게 되는 트랜잭션은 잠금을 얻을 때 까지 대기한다.

postgresql의 경우 한 트랜잭션에서 같은 조건의 발생하는 사이 해당 트랜잭션에서 적용된 부분만 조회된다. 단, 다른 트랜잭션에서 UPDATE, DELETE, MERGE, SELECT FOR UPDATE, SELECT FOR SHARE 이 커밋될 경우엔 `ERROR:  could not serialize access due to concurrent update` 라는 메세지와 함께 동시 접근이 발생한 트랜잭션이 취소된다.

serializable은 위의 이상현상은 발생하지 않지만 동시성이 많이 낮아진다.

mysql InnoDB 엔진의 기본 isolation level은 repeatable read 이고 postgresql의 기본 isolation level은 read committed이다.

https://www.postgresql.org/docs/current/transaction-iso.html

https://en.wikipedia.org/wiki/Isolation_%28database_systems%29

https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html

### 2. 본인이 사용하는 프레임워크에서 Transaction 기능이 어떻게 동작하는지 정리

#### typeorm

dataSource에서 connection을 얻어서 startTransaction을 시도하고 해당 connection을 이용해 쿼리를 진행해서 같은 세션의 트랜잭션을 유지한다.

#### sequelize

connection을 얻어서 startTransaction을 시도하는 것은 동일하고 해당 connection에서 쿼리를 진행하는 방식과 쿼리의 파라미터에 획득한 transaction host를 파라미터로 전달해 같은 세션의 트랜잭션을 유지하는 두가지 방식이 가능하다.

nodejs에서는 자바의 어노테이션(자바스크립트의 데코레이터)를 이용해 트랜잭션을 제어하는 방식은 주로 사용되지 않는다.