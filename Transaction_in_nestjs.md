# Nest.js에서의 Transaction

## TypeORM의 트랜잭션을 사용하는 3가지 방법.

- `QueryRunner`의 `DataSource`, `EntityManager`.
- `transaction` 객체 생성하기.
- `@Transaction`, `TransactionManager`, `TransactionRepository` 데코레이터 사용 : Nest에서 권하는 방식은 아니라고 한다.

## 트랜잭션 격리 수준 지정하기

```typescript
await myDataSource.manager.transaction(
  "SERIALIZABLE",
  (transactionalEntityManager) => {}
);
```

- **MySQL**, **PostgreSQL**, **SQL Server**는 표준 격리 수준(`READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`)을 지원한다.
- **SQLite**는 기본적으로 `SERIALIZABLE`로 설정되어있다. → 공유 캐시 모드가 활성화된 경우, `READ UNCOMMITTED` 수준을 사용할 수 있다.
- **Oracle**은 `READ COMMITTED`, `SERIALIZABLE`만 지원한다.

## QueryRunner로 제어하기

- Nest.js 공식 문서에 따르면, `QueryRunner` 클래스는 트랜잭션을 완전히 제어할 수 있기 때문에 권장한다고 한다.
- 트랜잭션을 제어하는 3가지 방법.

  - **startTransaction** : QueryRunner 인스턴스 내에서 새 트랜잭션을 시작한다.
  - **commitTransaction** : QueryRunner 인스턴스를 사용하여 만든 모든 변경 사항을 커밋한다.
  - **rollbackTransaction** : QueryRunner 인스턴스를 사용하여 만든 수행한 모든 변경 사항을 롤백한다.

### 트랜젝션 생성 및 사용

0. DataSource 또는 EntityManager 를 사용하여 트랜잭션을 생성한다.

> ⚠️ 전역 entity manager가 아닌 `transactionalEntityManager`를 사용해야한다

```typescript
await myDataSource.manager.transaction(async (transactionalEntityManager) => {
  await transactionalEntityManager.save(users);
  await transactionalEntityManager.save(photos);
  // ...
});
```

1.  class에 `DataSource` 객체를 inject 한다.

```typescript
import { DataSource } from "typeorm";

@Injectable()
export class UsersService {
  constructor(private dataSource: DataSource) {}
}
```

2. 해당 객체로 트랜잭션을 만들 수 있다.

```typescript
async createMany(users: User[]) {
  const queryRunner = this.dataSource.createQueryRunner();

  await queryRunner.connect();
  await queryRunner.startTransaction();
  try {
    await queryRunner.manager.save(users[0]);
    await queryRunner.manager.save(users[1]);

    await queryRunner.commitTransaction();
  } catch (err) {
    // 변경 사항을 롤백하기 위해 에러를 발생시킨다.
    await queryRunner.rollbackTransaction();
  } finally {
    // 수동으로 인스턴스화된 queryRunner를 릴리즈해야한다.
    await queryRunner.release();
  }
}
```

<!-- 3. `DataSource` 객체의 트랜잭션 메소드와 함께 콜백 스타일 접근 방식을 사용할 수 있다.

```typescript
async createMany(users: User[]) {
  await this.dataSource.transaction(async manager => {
    await manager.save(users[0]);
    await manager.save(users[1]);
  });
}
``` -->

## transaction 객체 생성하기

- transaction 객체는 `entityManager`를 콜백으로 받아 주어진 함수를 실행(트랜잭션으로 래핑)한다.

```typescript
// 지정된 함수 실행을 트랜잭션으로 래핑한다. 제공된 엔티티 관리자를 사용하여 모든 데이터베이스 작업을 실행해야 한다.
transaction<T>(runInTransaction: (entityManager: EntityManager) => Promise<T>): Promise<T>;

private async saveUserUsingTransaction(name: string, email: string, password: string, signupVerifyToken: string) {
  await this.connection.transaction(async manager => {
    const user = new UserEntity();
    user.id = ulid();
    user.name = name;
    user.email = email;
    user.password = password;
    user.signupVerifyToken = signupVerifyToken;

    await manager.save(user);

    // throw new InternalServerErrorException();
  })
}
```

### 참고

- https://docs.nestjs.com/techniques/database#transactions
- https://typeorm.io/transactions
- https://wikidocs.net/158616
