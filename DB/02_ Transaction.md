## 트랜잭션(Transaction)

### 트랜잭션이란?
- 데이터베이스의 상태를 변환시키는 하나의 논리적 수행하기 위한 작업의 단위
- 한꺼번에 모두 수행되어야 할 일련의 연산들을 의미

<br/>

### 트랜잭션의 특징 (ACID)

**1️⃣ Atomicity (원자성)**
- 트랜잭션의 연산은 데이터베이스에 모두 반영되든지 아니면 전혀 반영되지 않아야 한다. (All or Nothing)
- 트랜잭션 내의 모든 명령은 반드시 완벽히 수행되어야 하며, 모두가 완벽히 수행되지 않고 어느 하나라도 오류가 발생하면 트랜잭션 전부가 취소되어야 한다. (부분 성공해서는 안된다.)

**2️⃣ Consistenty (일관성)**
- 트랜잭션이 성공적으로 완료하면 언제나 일관성 있는 데이터베이스 상태로 변환한다.
- 시스템이 가지고 있는 고정 요소(ex 데이터베이스의 제약 또는 규칙)는 트랜잭션 수행 전과 수행 완료 후의 상태가 같아야 한다.
- ex)
    - "모든 학생은 학번이 존재해야 한다"라는 학사 시스템 데이터베이스의 제약 조건이 존재
    - 입학 예정인 신입생은 등록 전이라 학번이 없으므로 임시 학번 부여

**3️⃣ Isolation (격리성, 고립성, 독립성)**
- 둘 이상의 트랜잭션이 동시에 병행 실행되는 경우 어느 하나의 트랜잭션 실행 중에 다른 트랜잭션의 연산이 끼어들 수 없다.
- 수행 중인 트랜잭션은 완전히 완료될 때까지 다른 트랜잭션에서 수행 결과를 참조할 수 없다.
- 트랜잭션 수행 시 서로 끼어들지 못하는 것

**4️⃣ Durability (지속성, 영속성)**
- 성공적으로 완료된 트랜잭션 결과는 시스템이 고장나더라도 영구적으로 반영되어야 한다.

<br/>

### 트랜잭션 격리 수준(Isolation Level)
<img height="300" alt="image" src="https://github.com/user-attachments/assets/f7b099dc-62fc-491f-b3a0-921cb827bb03" />

<br/>
<br/>

**READ UNCOMMITTED (레벨 0) - 커밋되지 않는 읽기**
- 각 트랜잭션에서의 변경 내용을 Commit 이나 Rollback 여부와 상관없이 다른 트랜잭션에서 값을 읽을 수 있다.
- 즉, 커밋되지 않은 데이터조차 접근할 수 있는 격리 수
- 정합성에 문제가 많은 격리 수준이므로 사용하지 않는 것을 권장
- DIRTY READ 현상 발생
   - **DIRTY READ**: 트랜잭션 작업이 완료되지 않았음에도 다른 트랜잭션에서 볼 수 있다.
   <img height="300" alt="image" src="https://github.com/user-attachments/assets/2740567f-0d80-4383-84a7-9db750b728e5" />
   
   - Commit 되지 않은 상태지만 Update된 값을 다른 트랜잭션에서 읽기 가능

<br/>


**READ COMMITTED (레벨 1) - 커밋된 읽기**

<img height="300" alt="image" src="https://github.com/user-attachments/assets/5fb07627-dd92-4b41-a073-d33b810dbc57" />

- RDB에서 대부분 기본적으로 사용되고 있는 격리 수준
- 실제 테이블 값을 가져오는 게 아니라 Undo 영역에 백업된 레코드에서 값을 가져온다.
- DIRTY READ 현상 발생 X
- Non-repeatable Read 현상 발생
   - **Non-repeatable Read**: 한 트랜잭션에서 같은 쿼리를 두 번 실행할 때 그 사이에 다른 트랜잭션 값을 수정 또는 삭제하면서 두 쿼리의 결과가 상이하게 나타나는 일관성이 깨진 현상
  <img height="300" alt="image" src="https://github.com/user-attachments/assets/d008a419-d071-4963-b48e-2066e418982d" />

<br/>

**REPEATABLE READ (레벨 2) - 반복가능한 읽기**

<img height="300" alt="image" src="https://github.com/user-attachments/assets/ce6eb61a-271f-47fa-85dc-c9e6355faa60" />

- Undo 공간에 백업해두고 실제 레코드 값을 변경
- MySQL에서는 트랜잭션마다 트랜잭션 ID를 부여해 트랜잭션 ID보다 작은 번호에서 변경한 것만 읽음
- 백업된 데이터는 불필요하다고 판단하는 시점에 주기적으로 삭제
- Undo에 백업된 레코드가 많아지면 MySQL 서버의 처리 성능이 떨어질 수 있음
- 이러한 변경방식을 MVCC(Multi Version Concurrency Control) 라고 부름

- PHANTOM READ 현상 발생
   - PHANTOM READ: 다른 트랜잭션에서 수행한 변경 작업에 의해 레코드가 보였다가 안 보였다가 하는 현상
   - REPEATABLE READ 에 의하면 원래 출력되지 않아야 하는데 Update 영향을 받은 후부터 출력
   - 방지 방법: 쓰기(write) 잠금을 걸어야 함
  <img height="300" alt="image" src="https://github.com/user-attachments/assets/8d660194-61be-4149-9dac-c442cc37d086" />

<br/>

**SERIALIZABLE (래밸 3) - 직렬화 가능**
- 가장 엄격한 격리 수준, 완벽한 읽기 일관성 모드 제공
- 여러 트랜잭션이 동일한 레코드에 동시 접근 불가
- 트랜잭션이 순차적으로 처리되어야 하므로 성능 측면에서 동시 처리 성능이 가장 낮음

<br/>

### 트랜잭션의 연산

**Commit 연산**
<img height="300" alt="image" src="https://github.com/user-attachments/assets/d863a90c-ca3e-44aa-93a0-f20aea4f5ba8" />

- 트랜잭션이 성공적으로 수행되었음을 선언하는 명령어
- 처리 과정을 데이터베이스에 영구적으로 저장
- Commit 연산 수행 후에 하나의 트랜잭션 과정 종료 및 이전 데이터가 갱신된다.

<br/>

**Rollback 연산**
<img height="300" alt="image" src="https://github.com/user-attachments/assets/a8e25200-2ffa-4e79-a2b2-99f6b8378db1" />

- 작업 중 문제 발생으로 인해 트랜잭션의 처리 과정에서 발생한 변경 사항 취소하는 명령어
- 트랜잭션 일부가 정상적으로 처리되어도 트랜잭션이 행한 모든 연산 취소 (원자성)
- 트랜잭션 시작되기 이전의 상태로 되돌린다.

<br/>

### 트랜잭션 상태
<img height="300" alt="image" src="https://github.com/user-attachments/assets/708de719-1ec0-4548-97eb-fc5d9a5094cd" />

**Active (활동)**
- 트랜잭션이 실행 중인 상태

**Partially Committed (부분 완료)**
- 트랜잭션의 마지막 연산까지 실행되고 Commit 연산이 실행되기 직전의 상태

**Failed (실패)**
- 트랜잭션 실패 상태 (더이상 트랜잭션이 정상적으로 진행될 수 없는 상태)

**Committed (완료)**
- 트랜잭션이 정상적으로 완료된 상태

**Aborted (철회)**
- 트랜잭션이 취소되고, 트랜잭션 실행 이전 데이터로 돌아간 상태

<br/>

### 참고 문헌
- https://velog.io/@yu-jin-song/DB-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98Transaction
- https://star7sss.tistory.com/823#1)_%EC%9B%90%EC%9E%90%EC%84%B1_(Atomicity)
- https://coding-factory.tistory.com/226
- https://velog.io/@shasha/Database-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%A0%95%EB%A6%AC
