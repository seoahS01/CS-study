# 외래키 값은 NULL이 들어올 수 있나요?

외래키는 기본적으로 NULL을 허용하며, 이는 외래키 제약이 "값이 존재한다면 부모 테이블의 기본키 중 하나여야 한다"는 의미이지, "반드시 값이 존재해야 한다"는 의미가 아니기 때문이다.

## 외래키란

외래키(Foreign Key)는 테이블 간의 관계를 정의하는 키로, 다른 테이블의 기본키를 참조하는 속성이다.

```sql
-- 부모 테이블: departments
CREATE TABLE departments
(
    id   BIGINT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

-- 자식 테이블: employees
CREATE TABLE employees
(
    id            BIGINT PRIMARY KEY,
    name          VARCHAR(50),
    department_id BIGINT,
    FOREIGN KEY (department_id) REFERENCES departments (id)
);
```

위 정의에서 `department_id`는 NULL을 허용한다. 즉, 아직 부서가 배정되지 않은 직원도 INSERT 가능하다.

```sql
-- 부서 없이 직원 등록 → 가능
INSERT INTO employees (id, name, department_id)
VALUES (1, 'Alice', NULL);
```

## NULL을 허용하지 않으려면

반드시 관계가 존재해야 한다면 `NOT NULL`을 명시한다.

```sql
CREATE TABLE employees
(
    id            BIGINT PRIMARY KEY,
    name          VARCHAR(50),
    department_id BIGINT NOT NULL, -- ← 필수 관계
    FOREIGN KEY (department_id) REFERENCES departments (id)
);

-- NULL 삽입 시도 → 에러 발생
INSERT INTO employees (id, name, department_id)
VALUES (1, 'Alice', NULL);
-- ERROR: column "department_id" violates not-null constraint
```

## 참조 동작(Referential Action)

부모 테이블의 행이 변경/삭제될 때 자식 테이블의 외래키에 대한 처리 방식을 지정한다. NULL 허용 여부와 직결되는 옵션이 있다.

```sql
-- 기본 데이터
-- departments: (1, 개발팀), (2, 디자인팀)
-- employees: (1, Alice, 1), (2, Bob, 1), (3, Carol, 2)
-- 이 상태에서 DELETE FROM departments WHERE id = 1 실행 시:
```

|    옵션     |         자식 동작         |            비고            |
|:---------:|:---------------------:|:------------------------:|
|  CASCADE  |   Alice, Bob 연쇄 삭제    |     의도치 않은 대량 삭제 위험      |
| SET NULL  | Alice, Bob의 FK → NULL |   FK에 NOT NULL이면 사용 불가   |
| RESTRICT  |     삭제 즉시 거부 (에러)     |   자식 행이 존재하면 부모 삭제 차단    |
| NO ACTION |  기본 동작, 트랜잭션 종료 시 검사  | RESTRICT와 유사하나 검사 시점이 다름 |

`SET NULL` 옵션을 쓰려면 외래키 컬럼이 반드시 NULL을 허용해야 한다. NOT NULL이면 이 옵션 자체를 적용할 수 없다.

## 실무에서 외래키를 사용하지 않는 이유

많은 실무 환경에서 외래키 제약을 의도적으로 생략한다.

- 성능 비용: INSERT마다 부모 테이블 조회 + 공유 락(자식 INSERT 동안 부모 행이 삭제/변경되지 못하도록 거는 락) 획득이 발생하여, 대량 INSERT 시 락 경합의 원인이 됨
- 분산 환경의 한계: 마이크로서비스 아키텍처에서 서비스별로 DB를 분리하면 크로스 DB FK 설정 자체가 불가능
- 대안: 애플리케이션 레벨 검증, 이벤트 기반 동기화 등으로 정합성을 보장
