<h1>MySQL InnoDB의 MVCC에 대해 설명해주세요</h1>


MVCC(Multi Version Concurrency Control)는 하나의 레코드에 대해 여러 버전이 유지되고, 필요에 따라 보여지는 데이터가 다른 구조를 의미한다.

트랜잭션 격리수준에서 READ_UNCOMMITTED를 제외한 상위 레벨의 격리 수준의 경우 커밋되지 않은 데이터는 다른 트랜잭션에서 볼 수 없기에 InnoDB Buffer pool이나 Disk에 있는 내용 대신 Undo Log에 기록해준 변경되기 이전의 데이터를 반환해준다. 이러한 과정을 MVCC라고 한다.