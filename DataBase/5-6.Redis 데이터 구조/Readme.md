<h2>Redis가 지원하는 데이터 구조에는 어떤 것들이 있나요?</h2>

>Redis의 자료구조는 크게 기본 5가지와 특수 자료구조로 나눌 수 있다.
>
>기본 자료구조는 String, List, Hash, Set, Sorted Set이다. 
> - String은 가장 기본이 되는 타입으로 문자열뿐 아니라 숫자, 바이너리도 저장할 수 있어 카운터나 캐시로 쓰인다.
> - List는 양방향 연결 리스트로 큐나 스택 구현에 적합하다.
> - Hash는 필드-값 쌍을 저장해 객체를 표현할 때 유용하다. 
> - Set은 중복 없는 집합 연산을, Sorted Set은 점수 기반 정렬이 필요한 랭킹 시스템에 사용된다.
>
> 특수 자료구조로는 Bitmap, HyperLogLog, Geospatial, Stream이 있다.
> - Bitmap은 비트 단위 연산으로 출석 체크나 대량 플래그 관리에 효율적이다. 
> - HyperLogLog는 확률적 자료구조로 수억 개의 유니크 카운팅을 12KB로 처리한다. 
> - Geospatial은 위도/경도 기반 위치 검색에, Stream은 Kafka와 유사한 메시지 큐 기능을 제공한다.