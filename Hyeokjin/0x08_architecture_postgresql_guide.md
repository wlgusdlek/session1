## 1️⃣ 데이터베이스 정규화 (3NF)

### **1.1 정규화 개요**
- 데이터베이스 정규화는 **데이터 중복을 최소화하고 데이터 무결성을 유지**하기 위한 과정입니다.
- 중복 데이터를 저장하는 대신 **참조(reference)**를 통해 데이터를 연결합니다.
- 각 테이블에 **고유한 키(primary key)** 를 추가하여 참조 무결성을 보장합니다.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE
);
```

**📌 참고:** [정규화 위키백과](https://en.wikipedia.org/wiki/Database_normalization)

---

## 2️⃣ 관계형 vs. 비관계형 데이터베이스

### **2.1 관계형 데이터베이스 (SQL)**
- 데이터를 **행(Row)과 열(Column)** 형식으로 저장합니다.
- 데이터 정합성을 유지하기 위해 **ACID 원칙**을 따릅니다.
- 대표적인 SQL 데이터베이스:
    - **PostgreSQL**
    - **MySQL**
    - **Oracle**
    - **SQL Server**

### **2.2 비관계형 데이터베이스 (NoSQL)**
- 데이터를 **문서(Document), 키-값(Key-Value), 그래프(Graph)** 형태로 저장합니다.
- 데이터 정합성보다 **확장성과 속도**에 초점을 둡니다.
- 대표적인 NoSQL 데이터베이스:
    - **MongoDB** (문서 기반)
    - **Cassandra** (키-값 기반)
    - **BigTable** (구글의 대용량 분산 데이터베이스)

**📌 참고:** [NoSQL 위키백과](https://en.wikipedia.org/wiki/NoSQL)

---

## 3️⃣ ACID vs. BASE

### **3.1 ACID 모델 (신뢰성이 중요한 시스템)**
ACID는 데이터의 **일관성과 무결성**을 보장하기 위한 원칙입니다.
- **A (Atomicity, 원자성)**: 트랜잭션이 **완전히 수행되거나 완전히 롤백**됨.
- **C (Consistency, 일관성)**: 데이터베이스가 **일관된 상태**를 유지.
- **I (Isolation, 고립성)**: 트랜잭션 간의 **간섭 방지**.
- **D (Durability, 지속성)**: 트랜잭션이 **성공적으로 저장되면 영구 보존**.

ACID를 지원하는 데이터베이스 예시:
- **PostgreSQL**
- **Oracle**
- **MySQL**
- **SQLite**

### **3.2 BASE 모델 (대규모 분산 시스템)**
BASE는 확장성을 강조하는 **NoSQL 데이터베이스의 원칙**입니다.
- **B (Basically Available, 기본적인 가용성)**: 일부 장애가 발생해도 **서비스 지속 가능**.
- **A (Soft-state, 소프트 상태 유지)**: 데이터가 즉시 일관되지 않을 수 있음.
- **E (Eventual Consistency, 최종적 일관성)**: 시간이 지나면 **일관된 상태**를 유지.

BASE를 지원하는 데이터베이스 예시:
- **MongoDB**
- **Cassandra**
- **BigTable**

**📌 참고:** [ACID 원칙](https://en.wikipedia.org/wiki/ACID), [Eventual Consistency](https://en.wikipedia.org/wiki/Eventual_consistency)

---

## 4️⃣ 데이터베이스 확장 기법

### **4.1 수직 확장 (Vertical Scaling)**
- 서버의 **CPU, RAM, 디스크 성능을 업그레이드**하여 성능을 향상.
- 간단하지만, **비용이 급격히 증가**하는 한계가 있음.

### **4.2 수평 확장 (Horizontal Scaling)**
- 여러 서버를 추가하여 **부하를 분산**.
- **마스터-슬레이브 복제(Master-Slave Replication)** 또는 **샤딩(Sharding)**을 사용.

```sql
-- 마스터-슬레이브 복제 예시
SELECT * FROM replica_db.users;
```

**📌 참고:** [Scaling Databases](https://en.wikipedia.org/wiki/Scalability)

---

## 5️⃣ 데이터베이스 분산 구조

### **5.1 마스터-슬레이브 복제**
- **마스터(Master)**: 모든 쓰기 연산을 처리.
- **슬레이브(Slave)**: 읽기 전용으로 데이터 복제.
- **장점**: 읽기 부하 분산 가능.
- **단점**: 쓰기 연산 부하를 줄이기 어려움.

### **5.2 멀티 마스터 복제 (Multi-Master)**
- **여러 개의 마스터 노드**가 존재하며, 각각의 마스터에서 **쓰기 연산 가능**.
- **장점**: 쓰기 부하 분산.
- **단점**: 데이터 충돌 가능성이 있음.

```sql
-- Multi-Master Replication을 위한 트랜잭션 예제
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

## 6️⃣ 클라우드 데이터베이스 및 NoSQL

### **6.1 클라우드 데이터베이스**
- **Google Cloud SQL, Amazon RDS, Microsoft Azure SQL** 등의 서비스가 대표적.
- 확장성이 뛰어나며 관리 부담이 적음.

### **6.2 NoSQL의 등장**
- Google, Facebook, Amazon 같은 **대규모 웹 서비스**에서는 기존의 관계형 DB로는 확장이 어려웠음.
- 이를 해결하기 위해 **Cassandra, MongoDB, DynamoDB** 같은 NoSQL DB가 등장.

```json
{
    "name": "John Doe",
    "email": "johndoe@example.com",
    "orders": [
        {"item": "Laptop", "price": 1200},
        {"item": "Mouse", "price": 20}
    ]
}
```

**📌 참고:** [Google BigTable](https://en.wikipedia.org/wiki/Bigtable), [Amazon DynamoDB](https://en.wikipedia.org/wiki/Amazon_DynamoDB)

---

## 7️⃣ 결론 및 요약

- **관계형 데이터베이스(RDBMS)는 정규화를 통해 데이터 무결성을 유지**.
- **NoSQL 데이터베이스는 확장성과 가용성에 초점**을 둠.
- **ACID 모델은 강력한 데이터 정합성을 제공하지만 확장성이 떨어짐**.
- **BASE 모델은 확장성이 뛰어나지만 즉각적인 일관성이 보장되지 않음**.
- **마스터-슬레이브 복제, 멀티 마스터 복제, 샤딩을 통해 확장성 개선 가능**.

---

## 📌 참고 자료

- [PostgreSQL 공식 문서](https://www.postgresql.org/docs/)
- [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)
- [ACID vs BASE](https://en.wikipedia.org/wiki/ACID)
- [Google BigTable](https://en.wikipedia.org/wiki/Bigtable)
- [Amazon DynamoDB](https://en.wikipedia.org/wiki/Amazon_DynamoDB)
