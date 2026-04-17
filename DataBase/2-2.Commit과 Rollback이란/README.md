### **Q. Commit과 Rollback 이란?**

Commit은 트랜잭션의 모든 변경 사항을 데이터베이스에 영구적으로 반영하는 작업으로, 완료되면 트랜잭션이 정상 종료됩니다.

반면 Rollback은 오류 발생 시 트랜잭션의 모든 변경 사항을 취소하고 이전 상태로 되돌리는 작업으로, 데이터 정합성을 유지하기 위해 사용됩니다.

+) 이러한 과정은 Undo Log와 Redo Log, WAL 등을 기반으로 장애 상황에서도 데이터 정합성을 보장하도록 설계되어 있습니다.

</br>
</br>

**🧩 SQL 예시**

```sql
-- 성공 예시 (COMMIT)

START TRANSACTION;

UPDATE account SET balance = balance - 10000 WHERE user_id = 'A';
UPDATE account SET balance = balance + 10000 WHERE user_id = 'B';

COMMIT;
```
→ 두 UPDATE가 모두 정상적으로 수행되면 COMMIT을 통해 트랜잭션이 종료되고, 변경 내용이 데이터베이스에 영구 반영

```sql
-- 실패 예시 (ROLLBACK)

START TRANSACTION;

UPDATE account SET balance = balance - 10000 WHERE user_id = 'A';
-- 여기서 에러 발생

ROLLBACK;
```

→ 중간에 오류가 발생하면 이전 작업까지 모두 취소되고 ROLLBACK을 통해 트랜잭션 시작 전 상태로 복구

→ 트랜잭션은 commit 또는 rollback이 수행되는 시점에 종료

</br>
</br>

**🚨 추가 질문**

**Q. commit 하면 데이터 저장이 보장될까?**

- Commit이 완료되면 해당 트랜잭션의 변경 사항은 영구적으로 반영된 것으로 간주
- 하지만, 실제 DB 내부에서는 디스크 장애나 시스템 크래시와 같은 예외 상황이 발생할 수 있음
- DB는 WAL(Write-Ahead Logging)과 Redo Log를 통해 커밋된 트랜잭션이 유실되지 않도록 복구 가능한 구조로 동작
- → commit은 논리적으로 ‘영구 반영’을 의미하지만, 물리적으로는 로그 기반 복구 메커니즘을 통해 그 신뢰성을 보장

</br>

**Q. Rollback은 언제까지 가능할까?**

- rollback은 트랜잭션 commit 전까지 가능
- commit이 완료되면 해당 트랜잭션의 변경 사항은 확정되기 때문에 일반적인 rollback으로는 되돌릴 수 없음
- 이후에는 별도의 보상 트랜잭션이나 백업을 통해 복구해야 함

</br>

**Q. Auto Commit이 뭘까?**

- Auto Commit은 각 SQL 문이 실행될 때마다 하나의 트랜잭션으로 자동 처리되고, 실행이 완료되면 즉시 commit되는 방식
- 개발자가 명시적으로 commit과 rollback을 호출하지 않아도 각 쿼리가 독립된 트랜잭션 단위
- → SpringBoot에서는 기본적으로 JDBC auto-commit = true 기반으로 동작하지만, `@Transactional`을 사용하면 auto-commit이 비활성화되고 트랜잭션 단위로 commit과 rollback이 자동 관리됨.