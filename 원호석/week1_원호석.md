# DB Concurrency Control

## 1. Schedule, Serializability, Conflict Equivalence

### Schedule (스케줄)
- 여러 트랜잭션이 동시에 실행되는 명령어들의 순서.
- 종류:
  - **Serial Schedule**: 트랜잭션이 순차적으로 실행됨 (동시성 없음).
  - **Concurrent Schedule**: 트랜잭션들이 일부 연산을 교차하며 실행.

### Serializability (직렬 가능성)
- 하나의 스케줄이 **serial schedule과 동일한 결과**를 낼 수 있다면 직렬 가능(serializable)하다고 함.
- 종류:
  - **Conflict Serializability**: 충돌을 기준으로 직렬 가능 판단.
  - **View Serializability**: 읽은 값 기준으로 판단 (더 느슨한 기준).

### Conflict Equivalent (충돌 등가)
- 두 스케줄이 같은 연산 집합과 충돌 순서를 가진다면 conflict equivalent.
- Conflicting Operations:
  - 동일한 데이터 항목에 대해 최소 하나는 write이고, 다른 트랜잭션에서의 read/write.


## 2. Recoverability (회복 가능성)

### 📌 Recoverable Schedule
- T2가 T1의 데이터를 읽었다면, T1이 커밋된 후에 T2가 커밋되어야 함.

### 📌 Cascadeless Schedule
- T2는 오직 **커밋된 트랜잭션**의 데이터를 읽을 수 있음 → cascading rollback 방지.

### 📌 Strict Schedule
- 모든 read/write는 **커밋된 데이터**에만 접근 가능 → 가장 강력한 복구 성능.


## 3. Isolation Level, Snapshot Isolation

### 📌 ANSI SQL Isolation Level

| Isolation Level        | Dirty Read | Non-repeatable Read | Phantom Read |
|------------------------|------------|----------------------|---------------|
| Read Uncommitted       | ✅ 허용     | ✅ 허용              | ✅ 허용       |
| Read Committed         | ❌ 차단     | ✅ 허용              | ✅ 허용       |
| Repeatable Read        | ❌ 차단     | ❌ 차단              | ✅ 허용       |
| Serializable           | ❌ 차단     | ❌ 차단              | ❌ 차단       |

### 📌 Snapshot Isolation (MVCC 기반)
- 트랜잭션은 시작 시점의 snapshot을 읽음.
- 중간에 다른 트랜잭션의 변경을 보지 않음.
- 대부분의 DB(PostgreSQL, Oracle 등)에서 사용.
- 단점: **Write skew anomaly** 발생 가능.

### ⚠️ 1995년 비판: “A Critique of ANSI SQL Isolation Levels”

#### 🔸 주요 비판 내용:
1. Dirty/Non-repeatable/Phantom Read만으로는 anomaly 설명 부족
2. 구현체마다 같은 격리 수준의 동작이 다를 수 있음
3. Lost Update, Write Skew 등은 ANSI 기준에 없음
4. Snapshot Isolation 같은 현실적인 기법은 ANSI 표준에 없음

#### 📌 Snapshot Isolation vs Serializable

| 항목 | Snapshot Isolation | Serializable |
|------|--------------------|--------------|
| 읽기 동작 | 트랜잭션 시작 시 snapshot | 모든 충돌 차단 |
| 성능 | 높음 | 낮음 |
| anomaly | Write Skew 발생 가능 | 없음 |
| 구현 | PostgreSQL, Oracle 등 | 제한적 사용 |

#### ✅ 제안된 개선
- anomaly 중심으로 isolation level을 정의해야 함
- 이후 Serializable Snapshot 등 발전


## 4. Lock-Based Concurrency Control

### 📌 Lock 종류
- **Shared Lock (S-lock)**: 읽기 전용, 공유 가능
- **Exclusive Lock (X-lock)**: 쓰기 전용, 배타적

### 📌 2-Phase Locking (2PL)
- **직렬 가능성 보장**.
- 단계:
  1. Growing phase (lock 획득만 가능)
  2. Shrinking phase (lock 해제만 가능)

- **Strict 2PL**: 모든 lock은 트랜잭션 커밋/롤백 시점까지 유지

### 📌 Deadlock
- 서로의 lock을 기다리며 무한 대기
- 해결 방법:
  - 타임아웃
  - Wait-Die / Wound-Wait
  - Deadlock detection & rollback


