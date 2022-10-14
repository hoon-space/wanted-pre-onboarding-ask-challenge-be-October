자바는 JPA에서 entity manager라는 객체에서 트랜잭션 인스턴스를 생성할 수 있다.
인턴스의 begin 메소드를 호출하고나서
commit 메소드나
rollback 메소드가 동작하기 전까지는 하나의 트랜잭션으로 동작한다.
대부분 try catch블록으로 감싸서 오류 발생시 catch블록에서 rollback, try블록 마지막에 commit메소드를 배치한다.
이를 관점분리를 통하여 AOP로 제공하는 어노테이션이 @Transational이다.
혹은 JDBC에서 queryrunner라는 객체를 통해서 진행할 수도 있다.


TypeORM에선 datasource에서 createQueryRunner메소드를 통해 트랜잭션을 관리하는 인스턴스를 반환할 수 있다.
connect메소드를 이용해 시작하며
release메소드를 이용해 끝낸다.

또는 manager 속성의 transaction 메소드에 콜백함수 내부에 한 트랜잭션에 해당하는 쿼리들을 담아 트랜잭션을 처리할 수도 있다.