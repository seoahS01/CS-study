### **Q.다수의 트랜잭션이 하나의 자원을 접근할 때 발생할 수 있는 문제들은 무엇이 있을까요?**

다수의 트랜잭션이 하나의 자원을 동시에 접근하면 동시성 문제로 인해 데이터 불일치가 발생할 수 있습니다.

대표적으로 아직 커밋되지 않은 데이터를 읽는 Dirty Read, 같은 행을 두 번 읽었을 때 값이 달라지는 Non-repeatable Read, 그리고 동일한 조건으로 조회했을 때 결과 행이 달라지는 Phantom Read 현상이 발생할 수 있습니다.

</br>
</br>

1️⃣ **Dirty Read (커밋, 롤백에 관한)**

다른 트랜잭션이 아직 커밋하지 않은 데이터를 조회했는데, 해당 트랜잭션이 롤백되면서 존재하지 않는 데이터를 읽게 되는 현상

```sql
T1: UPDATE user SET age=30 (commit 안함)
T2: SELECT age → 30
T1: ROLLBACK     // T1이 롤백해서 T2는 잘못된 데이터를 읽은 것이 됨
```

</br>

2️⃣ **Non-repeatable Read (수정에 관한)**

같은 행을 두 번 조회했을 때, 중간에 다른 트랜잭션이 해당 데이터를 수정-커밋하여 조회 결과가 달라지는 현상

```sql
T1: SELECT age → 20
T2: UPDATE age=30 COMMIT
T1: SELECT age → 30   // T2가 중간에 값 변경해서 커밋해서 값이 달라짐
```

</br>

**3️⃣ Phantom Read (삽입, 삭제에 관한)**

같은 조건으로 범위 조회를 했을 때, 중간에 다른 트랜잭션이 데이터를 추가/삭제하여 결과 집합이 달라지는 현상

```sql
T1: SELECT * FROM user WHERE age >= 20 → 10 rows
T2: INSERT age=25 COMMIT
T1: SELECT * FROM user WHERE age >= 20 → 11 rows
```

</br>

→ 이런 이상 현상을 방지하기 위해 트랜잭션 격리 수준(Isolation)을 설정하여 정합성과 성능 사이에서 trade-off를 조절한다.