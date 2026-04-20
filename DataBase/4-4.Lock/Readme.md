<h1>MySQL 락에 대해 설명해주세요</h1>

<h2>Row-Level Lock</h2>
InnoDB에서 Row-level의 Lock은 Shared Lock(공유락) 과 Exclusive Lock(베타락) 으로 2가지 유형이 존재한다.

<h3>Shared Lock(공유 락, S Lock)</h3>
특정 Row를 읽을(Read) 때 사용되는 Lock이다.

- 여러 트랜잭션이 동시에 한 Row에 Shared Lock을 걸 수 있다. → 하나의 Row를 여러 트랜잭션이 동시에 읽을 수 있다.

- Shared Lock이 설정된 Row에는 Exclusive Lock을 사용할 수 없다.

- InnoDB에서 일반적인 SELECT 쿼리는 Lock을 사용하지 않는다. 하지만 SELECT .. FOR SHARE 등의 일부 쿼리는 각 Row에 Shared Lock을 건다.

<h2>Exclusive Lock(배타 락, X Lock)</h2>
특정 Row를 변경(write)할 때 사용된다.

- 특정 Row에 Exclusive Lock이 걸려있을 경우, 다른 트랜잭션은 읽기 작업을 위해 Shared Lock을 걸거나, 쓰기 작업을 위해 Exclusive Lock을 걸 수 없다. → 쓰기 작업을 하고 있는 Row에는 모든 접근이 불가하다.

- SELECT … FOR UPDATE, UPDATE, DELETE 등의 수정 쿼리들이 실행될 때 Row에 걸린다.

<h2>Index record & Gap Lock</h2>
Record Lock, Gap Lock, Next Key Lock 은 Row가 아닌 DB의 index record와 record사이의 간격에 걸리는 Lock이다.

<h3>레코드(Record) 락</h3>

- 레코드 락이란 레코드 자체만을 잠그는 것을 의미한다. InnoDB 스토리지 엔진은 레코드 자체가 아니라 인덱스의 레코드를 잠그는 방식으로 동작한다.

- record lock안에서도 Shared Lock과 Exclusive Lock이 존재한다.
    - 읽기 작업이 발생할 경우 Shared Lock, 쓰기 작업이 있을 경우 Exclusive Lock이 걸린다.

<h3>갭(GAP) 락</h3>
갭 락은 레코드 자체가 아니라 레코드와 바로 인접한 레코드 사이의 간격을 잠그는 것을 의미한다. 갭 락의 역할은 레코드와 레코드 사이의 간격에 새로운 레코드가 생성되는 것을 제어한다.

<h3>넥스트 키 락(Next key lock)</h3>
레코드 락과 갭 락을 합쳐놓은 것을 의미한다.