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
