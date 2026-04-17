### **Q. 트랜잭션의 격리 수준에 대해 설명해주세요**

트랜잭션의 격리 수준은 동시에 실행되는 트랜잭션 간의 간섭을 어느 정도 허용할지를 결정하는 기준입니다.

크게 Read Uncommitted, Read Committed, Repeatable Read, Serializable 네 가지가 있으며, 격리 수준이 높아질수록 데이터 정합성은 높아지지만 동시성 성능은 낮아지고, 낮아질수록 성능은 좋아지지만 데이터 이상 현상이 발생할 수 있습니다.

각 수준에 따라 Dirty Read, Non-repeatable Read, Phantom Read와 같은 동시성 문제가 발생하거나 방지됩니다.

따라서 서비스의 특성에 따라 정합성과 성능의 균형을 고려해 적절한 격리 수준을 선택하는 것이 중요합니다.

</br>
</br>

1️⃣ **Read Uncommitted**

- 커밋되지 않은 데이터도 읽을 수 있는 격리 수준
- **→ Dirty Read 발생 가능**
    
    ![Dirty Read](DirtyRead.png)

</br>

2️⃣ **Read Committed**

- 커밋된 데이터만 읽을 수 있는 격리 수준
- 중간에 다른 트랜잭션이 조회한다면 Undo Log에 기록된 변경 전 데이터를 반환
- **→ Non-repeatable Read 발생 가능**
    
    ![Read Committed 동작 과정](ReadCommitted.png)
    
    Read Committed 동작 과정
    
    ![Non-repeatable Read](NonRepeatableRead.png)
    
    Non-repeatable Read

 </br>   

3️⃣ **Repeatable Read**

- 한 트랜잭션 내에서 동일한 데이터를 여러 번 조회하더라도 항상 같은 결과를 보장
- **→ Phantom Read 발생 가능**
- → 이미 조회한 row에 대해서는 스냅샷을 유지해 일관된 값을 보장하지만, 조회 조건에 해당하는 새로운 row의 삽입까지는 제어하지 못하기 때문에 Phantom Read 발생 가능

</br>

4️⃣ **Serializable**

- 가장 높은 격리 수준으로, **트랜잭션을 순차적으로 실행**하는 것과 동일하게 동작
- 모든 이상 현상을 방지하지만, 동시성이 크게 떨어져 성능 저하 발생

→ 일반적으로는 Read Committed나 Repeatable Read를 많이 사용하며, **MySQL의 InnoDB 스토리지 엔진의 경우 기본 격리 수준은 Repeatable Read**이다