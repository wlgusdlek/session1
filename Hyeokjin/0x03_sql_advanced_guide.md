## 1️⃣ 테이블 생성 및 데이터 조작

### **1.1 테이블 생성**

```sql
CREATE TABLE account (
    id SERIAL PRIMARY KEY,
    email VARCHAR(128) UNIQUE,
    created_at DATE NOT NULL DEFAULT NOW(),
    updated_at DATE NOT NULL DEFAULT NOW()
);

CREATE TABLE post (
    id SERIAL PRIMARY KEY,
    title VARCHAR(128) UNIQUE NOT NULL,
    content TEXT,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE comment (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    post_id INTEGER REFERENCES post(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE fav (
    id SERIAL PRIMARY KEY,
    post_id INTEGER REFERENCES post(id) ON DELETE CASCADE,
    account_id INTEGER REFERENCES account(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (post_id, account_id)
);
```

### **1.2 테이블 수정 (ALTER TABLE)**

- **컬럼 삭제:**
```sql
ALTER TABLE fav DROP COLUMN oops;
```

- **컬럼 타입 변경:**
```sql
ALTER TABLE post ALTER COLUMN content TYPE TEXT;
```

- **새로운 컬럼 추가:**
```sql
ALTER TABLE fav ADD COLUMN howmuch INTEGER;
```

---

## 2️⃣ 시간 및 날짜 관련 기능

### **2.1 기본 날짜 및 시간 타입**

- `DATE`: 날짜만 (`YYYY-MM-DD` 형식)
- `TIME`: 시간만 (`HH:MM:SS` 형식)
- `TIMESTAMP`: 날짜 + 시간 (`YYYY-MM-DD HH:MM:SS` 형식)
- `TIMESTAMPTZ`: 날짜 + 시간 + 타임존 정보

```sql
SELECT NOW();
SELECT NOW() AT TIME ZONE 'UTC';
SELECT NOW() AT TIME ZONE 'Asia/Seoul';
```

### **2.2 날짜 연산 및 변환**

```sql
SELECT NOW(), NOW() - INTERVAL '2 days', (NOW() - INTERVAL '2 days')::DATE;
SELECT DATE_TRUNC('day', NOW());
```

---

## 3️⃣ 성능 최적화 및 데이터 조회

### **3.1 테이블 스캔과 성능 최적화**

- 날짜 필드에서 특정 날짜만 조회할 때:
```sql
SELECT id, content, created_at FROM comment WHERE created_at::DATE = NOW()::DATE;
```

### **3.2 DISTINCT 및 GROUP BY**

- **중복 제거(DISTINCT)**
```sql
SELECT DISTINCT model FROM racing;
```

- **그룹화 및 집계(GROUP BY)**
```sql
SELECT make, model FROM racing GROUP BY model;
```

### **3.3 HAVING 절 활용**

```sql
SELECT COUNT(abbrev), abbrev FROM pg_timezone_names GROUP BY abbrev HAVING COUNT(abbrev) > 10;
```

---

## 4️⃣ 서브쿼리 활용

- **특정 사용자(email)가 작성한 댓글 조회:**
```sql
SELECT content FROM comment WHERE account_id = (SELECT id FROM account WHERE email='ed@umich.edu');
```

---

## 5️⃣ 동시성 처리 및 트랜잭션

### **5.1 트랜잭션 처리**
```sql
BEGIN;
UPDATE tracks SET count=count+1 WHERE id = 42;
COMMIT;
```

### **5.2 충돌 처리 (ON CONFLICT)**

```sql
INSERT INTO fav (post_id, account_id, howmuch) VALUES (1, 1, 1)
ON CONFLICT (post_id, account_id) DO UPDATE SET howmuch = fav.howmuch + 1 RETURNING *;
```

---

## 6️⃣ 트리거 및 저장 프로시저

### **6.1 트리거 설정**
```sql
CREATE OR REPLACE FUNCTION trigger_set_timestamp() RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_timestamp
BEFORE UPDATE ON fav
FOR EACH ROW
EXECUTE PROCEDURE trigger_set_timestamp();
```

### **6.2 저장 프로시저 사용**
```sql
CREATE FUNCTION update_howmuch() RETURNS TRIGGER AS $$
BEGIN
    NEW.howmuch = NEW.howmuch + 1;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER increase_howmuch
BEFORE UPDATE ON fav
FOR EACH ROW
EXECUTE FUNCTION update_howmuch();
```

---

## 7️⃣ CSV 파일 처리 및 정규화

```sql
SELECT * FROM xy;
SELECT * FROM y;
```

---

## 8️⃣ 고급 SQL 테크닉

### **8.1 테이블 락 (LOCK)**
```sql
LOCK TABLE account IN ACCESS EXCLUSIVE MODE;
```

### **8.2 병렬 쿼리 실행**
```sql
SET max_parallel_workers_per_gather = 4;
```

### **8.3 JSONB 데이터 처리**
```sql
CREATE TABLE user_data (
    id SERIAL PRIMARY KEY,
    details JSONB
);

INSERT INTO user_data (details) VALUES ('{"name": "Alice", "age": 25}');
SELECT details->>'name' FROM user_data WHERE details->>'age' = '25';
```

---

## 9️⃣ 정리

- **트랜잭션**을 활용하여 데이터 정합성을 유지합니다.
- **JOIN, 서브쿼리, DISTINCT, GROUP BY, HAVING** 등의 SQL 기능을 활용하여 데이터를 효과적으로 조회합니다.
- **트리거 및 저장 프로시저**를 활용하여 데이터 변경 시 자동 처리 기능을 구현합니다.
- **고급 SQL 테크닉** (JSONB 처리, 병렬 쿼리 실행, 테이블 락) 등을 활용하여 데이터베이스 성능을 최적화할 수 있습니다.

---

## 📌 참고 자료

- [PostgreSQL 공식 문서](https://www.postgresql.org/docs/)
- [MySQL 공식 문서](https://dev.mysql.com/doc/)
- [SQL Server 공식 문서](https://docs.microsoft.com/en-us/sql/)
