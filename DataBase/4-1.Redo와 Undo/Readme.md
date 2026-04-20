<h1>MySQL InnoDB의 Undo 영역과 Redo 영역에 대해 설명해 주세요.</h1>

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


선 로그 기법(Log Ahead): 데이터 변경 작업 시 데이터 변경에 대한 내용을 Redo Log Buffer에 미리 저장

<h3>언제 LGWR 프로세스가 실행될까?</h3>
- 데이터베이스 커밋(commit)이 수행되었을 때
- Redo 로그 버퍼가 1/3이상 찼을 때
- DBWR이 변경된 데이터 블록을 저장하기 전 3초마다
- LOG_CHECKPOINT_TIMEOUT파라미터 설정 시간에 의해 TIME-OUT이 발생할 때

<h2>Undo Log</h2>
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

    하나의 트랜잭션이 너무 길어지게 되면 다른 트랜잭션들에 의해 지속적인 checkpoint가 발생하기 때문에 redo / undo log가 지속적으로 파일에 쓰여지게 되고 심각한 경우 disk 100% 사용까지도 가능해진다.


- rollback은 무거운 작업
  
    rollback은 결국 수 많은 undo log 정보를 참조하여 기존의 레코드로 정보를 리턴하는 작업이다. 즉, 트랜잭션이 길면 길수록 undo log에 써진 내용이 많아 rollback 작업은 무거워질수 밖에 없다.

<h2>한 줄 정리</h2>
- Redo 로그 (변경 후의 값을 기록) forwarding

- Undo 로그 (변경 전의 값을 기록) backing

