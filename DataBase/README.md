## 📊Database

<details>
  <summary>MySQL InnoDB의 Undo 영역과 Redo 영역에 대해 설명해 주세요.</summary>
  <h2>Redo</h2>
  DB 장애 발생 시 복구에 사용되는 log

  <h3>왜 Redo Log가 필요할까?</h3>
  우선, InnoDB Buffer Pool에 대해 이해해보자. Buffer Pool은 InnoDB 엔진이 Table Caching 및 Index Data Caching을 위해 이용하는 메모리 공간이다. 즉, Buffer Pool 크기(메모리 공간)가 클수록 상대적으로 캐싱되는 데이터의 양이 늘어나기 때문에 Disk에 접근하는 횟수가 줄어들고, 이것은 DB의 성능 향상으로 이어진다.

하지만, Buffer Pool은 메모리 공간이기 때문에 MySQL 장애 발생시 Buffer Pool에 있는 내용은 사라지게 된다. 이것은 ACID를 보장할수 없게 되고, 다시 해석하면 장애를 복구하더라도 데이터는 복구될 수 없다는 것을 의미한다.

  <img width="912" height="780" alt="image" src="https://github.com/user-attachments/assets/2c7b04df-2c7d-4753-afec-59550a7ee2e7" />

  위 그림은 MySQL의 Commit 실행과정이다.

실제 DB에서 Commit이 발생하면 바로 디스크 영역(Table Space)으로 들어가는 것이 아닌 메모리 영역(Buffer Poll & Log Buffer)에 들어가는 것을 확인할 수 있다.(DISK I/O 절약)

<h2>항상 Redo Log에 기록될까?</h2>
항상 쓰이진 않는다.

Redo Log에 기록할 때는 데이터 변경이 있을 때이다. DML, DDL, TCL 작업 등 데이터 변경이 일어나는 모든 것을 기록한다.(SELECT문은 데이터 변경 x)

<h2>Redo Log File이란?</h2>
여태까지 말한 것은 메모리 영역인 Redo Log Buffer에 저장되는 부분을 설명한 것이다. Redo Log Buffer 또한 메모리 영역이니 장애가 발생한다면 사라진다. '그렇다면 어떻게 이걸로 복구한다는거야?'라고 생각하실 수 있다. 바로 Redo Log File로 복구하는 것이다!

방금 Log Buffer는 메모리 영역이라고 했다. 메모리 영역이라는 것은 용량이 제한적이라는 뜻이다. 용량이 제한적이기 때문에 Checkpoint 이벤트 발생시점에 Redo Log Buffer에 있던 데이터들을 Disk에 File로 저장하게 된다. 이 파일을 Redo Log File라고 부른다.

Redo Log File은 두 개의 파일로 구성되는데, 하나의 파일이 가득차면 log switch가 발생하며 다른 파일에 쓰게된다. log switch가 발생할 때마다 Checkpoint 이벤트도 발생하는데, 이때 InnoDB Buffer Pool Cache에 있던 데이터들이 백그라운드 스레드에 의해 디스크에 기록된다.

> Checkpoint 이벤트가 발생하기 전 장애가 발생한다면, Buffer Pool에 있던 데이터들은 유실되지만 마지막 Checkpoint가 수행된 시점(=log swtich)까지의 데이터가 Redo Log File로 남아있기 때문에 이 파일을 사용하여 데이터를 복구할 수 있다.

<img width="968" height="605" alt="image" src="https://github.com/user-attachments/assets/ade7e969-35d2-4835-b952-caf152d4a1a9" />
오탈자) 3. Redo Log Buffer -> Redo Log File


선 로그 기법(Log Ahead)
: 데이터 변경 작업 시 데이터 변경에 대한 내용을 Redo Log Buffer에 미리 저장

<h2>언제 LGWR 프로세스가 실행될까?</h2>
- 데이터베이스 커밋(commit)이 수행되었을 때
- Redo 로그 버퍼가 1/3이상 찼을 때
- DBWR이 변경된 데이터 블록을 저장하기 전 3초마다
- LOG_CHECKPOINT_TIMEOUT파라미터 설정 시간에 의해 TIME-OUT이 발생할 때

<h3>Undo Log</h3>
실행 취소 로그 레코드의 집합으로 트랜잭션 실행 후 Rollback 시 Undo Log를 참조해 이전 데이터로 복구할 수 있도록 로깅 해놓은 영역

<h3>UNDO는 왜 필요할까?</h3>
작업 수행 중에 수정된 페이지들이 버퍼 관리자의 버퍼 교체 알고리즘에 따라서 디스크에 출력될 수 있다. 버퍼 교체는 전적으로 버퍼의 상태에 따라 결정되며, 일관성 관점에서 봤을 때는 임의의 방식으로 일어나게 된다. 즉, 아직 완료되지 않은 트랜잭션이 수정한 페이지들도 디스크에 출력될 수 있으므로, 만약 해당 트랜잭션이 어떤 이유든 정상적으로 종료될 수 없게 되면 트랜잭션이 변경한 페이지들은 원상 복구되어야 한다. 이러한 복구를 UNDO라고 한다.

<h3>어떻게 파일로 저장되는가?</h3>

```sql
-- 데이터 INSERT
INSERT INTO member(m_id, m_name, m_area) VALUES (12, '홍길동', '서울');
COMMIT;

-- 데이터 UPDATE
UPDATE member SET m_area='경기' WHERE m_id=12;
```

<img width="675" height="573" alt="image" src="https://github.com/user-attachments/assets/883c6389-e00d-4111-95fb-76cb15b6148c" />

Undo Log도 Redo Log와 마찬가지로 Log Buffer에 기록된다. Undo Recodrs 영역에 기록되는 것이다. 저장되는 데이터는 PK값과 변경되기 전의 데이터 값이다.

위 sql 코드와 그림을 보면 update 쿼리가 실행되면 (commit/rollback 전) InnoDB buffer pool에 캐싱된 데이터는 update한 정보로 수정된다. 데이터를 수정함과 동시에 rollback을 대비하기 위해, 업데이트 전의 데이터를 undo records로 기록하는 것이다.

<h3>주의점</h3>
- 너무 긴 트랜잭션
  : 하나의 트랜잭션이 너무 길어지게 되면 다른 트랜잭션들에 의해 지속적인 checkpoint가 발생하기 때문에 redo / undo log가 지속적으로 파일에 쓰여지게 되고 심각한 경우 disk 100% 사용까지도 가능해진다.

- rollback은 무거운 작업
  rollback은 결국 수 많은 undo log 정보를 참조하여 기존의 레코드로 정보를 리턴하는 작업이다. 즉, 트랜잭션이 길면 길수록 undo log에 써진 내용이 많아 rollback 작업은 무거워질수 밖에 없다.

<h2>한 줄 정리</h2>
- Redo 로그 (변경 후의 값을 기록) forwarding

- Undo 로그 (변경 전의 값을 기록) backing

</details>

<details>
  <summary>MySQL의 Insert Buffer란?</summary>
  레코드가 Insert, Update되었을 때, 데이터의 변경 뿐만 아니라 인덱스를 업데이트 하는 작업도 필요하다.

InnoDB는 변경해야 할 인덱스 페이지가 Buffer Pool에 있으면 바로 업데이트를 수행하지만, 그렇지 않을 경우 Disk로부터 페이지를 읽어온 후, 업데이트를 해야해서 이를 즉시 수행하지 않고 임시저장공간인 인서트 버퍼(Insert Buffer)에 저장해두고 결과는 바로 사용자에게 반환하는 형태로 성능을 향상시켰다.

즉, Index에 대한 업데이트 작업은 Insert Buffer에 지연시키고 사용자에게는 작업이 완료되었다고 먼저 통보를 한다.
  
</details>

<details>
  <summary>MySQL InnoDB의 MVCC에 대해 설명해주세요</summary>
  MVCC(Multi Version Concurrency Control)는 하나의 레코드에 대해 여러 버전이 유지되고, 필요에 따라 보여지는 데이터가 다른 구조를 의미한다.

  트랜잭션 격리수준에서 READ_UNCOMMITTED를 제외한 상위 레벨의 격리 수준의 경우 커밋되지 않은 데이터는 다른 트랜잭션에서 볼 수 없기에 InnoDB Buffer pool이나 Disk에 있는 내용 대신 Undo Log에 기록해준 변경되기 이전의 데이터를 반환해준다. 이러한 과정을 MVCC라고 한다.
  
</details>

<details>
  <summary>MySQL 락에 대해 설명해주세요</summary>
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
  
</details>

<details>
  <summary>MySQL에서 기본키를 설정하지 않고 테이블을 만들면 어떻게 될까요?</summary>
  
  - 기본값인 innoDB엔진은 데이터를 저장하고 indexing하기 위해 PK를 요구한다. 그래서 PK를 지정하지 않을 경우 auto_increment 속성의 사용자에게 노출되지 않는 hidden PK가 생성된다.

  - 하지만 해당 속성의 경우 데이터 관리를 하는데 어려움이 있어 명시적으로 설정하는 것이 좋다.

    - 기본키와 Unique제약조건이 없는 테이블을 만들경우, 데이터를 사용할 때 자동으로 만들어지는 PK를 사용하지 않게 되고 secondary index도 없는 테이블이 만들어진다. 그 결과 조회, 삭제와 같은 연산을 수행할 때 사용할 적절한 Index가 없어 성능 저하가 발생한다.
      
    - 기본키가 없을 경우 테이블간의 관계 모델링을 하는 것이 일반적으로 불가능하다.

</details>

<details>
  <summary>MySQL, Oracle의 Default 아이솔레이션 레벨 전략이 무엇인지 설명해 주세요.</summary>
  
</details>
