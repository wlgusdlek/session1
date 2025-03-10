## 1️⃣ 텍스트 데이터 생성 및 성능 테스트

### **랜덤 데이터 생성**
PostgreSQL에서는 **repeat()** 함수로 긴 문자열을 생성하고, **generate_series()**로 다량의 행을 만들 수 있다.

```sql
-- 특정 문자열을 반복 생성 (가로 확장)
SELECT repeat('Neon ', 5);
-- 결과: 'Neon Neon Neon Neon Neon'

-- 연속된 숫자 생성 (세로 확장)
SELECT generate_series(1, 5);
-- 결과:
-- 1
-- 2
-- 3
-- 4
-- 5
```

**랜덤 숫자 생성 예제**
```sql
SELECT random(), random(), trunc(random() * 100);
```
- `random()`: 0~1 사이의 난수를 생성
- `trunc(random() * 100)`: 0~100 사이의 정수 생성

---

## 2️⃣ 문자열 관련 함수

### **대소문자 변환**
```sql
SELECT UPPER('hello world'); -- 'HELLO WORLD'
SELECT LOWER('HELLO WORLD'); -- 'hello world'
```

### **부분 문자열 추출**
```sql
SELECT LEFT('PostgreSQL', 4);  -- 'Post'
SELECT RIGHT('PostgreSQL', 4); -- 'SQL'
SELECT SUBSTRING('PostgreSQL' FROM 5 FOR 3); -- 'gre'
```

### **문자열 포함 여부 (위치 검색)**
```sql
SELECT strpos('PostgreSQL', 'SQL'); -- 7 (SQL이 시작하는 위치)
```

### **문자열 대체**
```sql
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');
-- 결과: 'Hello PostgreSQL'
```

---

## 3️⃣ LIKE 및 정규 표현식 활용

### **LIKE 연산자**
```sql
SELECT * FROM textfun WHERE content LIKE '%example%';
-- 'example'이 포함된 문자열 검색
```

### **정규 표현식 활용**
```sql
SELECT * FROM textfun WHERE content ~ 'regex_pattern';
```

- `~` : 정규 표현식 패턴과 일치하는 값 찾기  
- `~*` : 대소문자 구분 없이 검색  
- `!~` : 정규 표현식과 일치하지 않는 값 찾기  

**예제: 이메일 형식 검증**
```sql
SELECT * FROM email_table WHERE email ~ '^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$';
```

---

## 4️⃣ 텍스트 검색 최적화

### **B-Tree 인덱스 적용**
```sql
CREATE INDEX textfun_b ON textfun(content);
```

### **EXPLAIN ANALYZE 활용**
```sql
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content LIKE 'example%';
```

### **인덱스를 활용한 검색**
```sql
EXPLAIN ANALYZE 
SELECT content FROM textfun WHERE content IN ('example1', 'example2');
```

---

## 5️⃣ 해싱과 인덱스

### **MD5 해시 적용**
```sql
SELECT md5('hello');
-- 결과: 5d41402abc4b2a76b9719d911017c592

CREATE INDEX textfun_md5 ON textfun (md5(content));
```

### **UUID 활용**
```sql
CREATE TABLE cr3 (
    id SERIAL,
    url TEXT,
    url_md5 UUID UNIQUE,
    content TEXT
);

UPDATE cr3 SET url_md5 = md5(url)::UUID;
```

---

## 6️⃣ UTF-8 및 문자 인코딩

### **문자셋 확인**
```sql
SHOW SERVER_ENCODING; -- UTF8
```

### **ASCII 값 변환**
```sql
SELECT ascii('A'), chr(65); -- 65, 'A'
```

---

## 7️⃣ 정규 표현식 활용 (예제)

### **이메일 검색**
```sql
SELECT email FROM users WHERE email ~ '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-z]{2,}$';
```

### **문장에서 해시태그 추출**
```sql
SELECT regexp_matches(tweet, '#([A-Za-z0-9_]+)', 'g') FROM tweets;
```

---

## 8️⃣ 인덱스를 활용한 성능 최적화

### **B-Tree 인덱스 vs 해시 인덱스**
```sql
-- B-Tree 인덱스 생성
CREATE INDEX textfun_btree ON textfun(content);

-- 해시 인덱스 생성
CREATE INDEX textfun_hash ON textfun USING hash(content);
```

### **EXPLAIN ANALYZE를 활용한 성능 비교**
```sql
-- B-Tree 인덱스 검색
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content = 'example';

-- 해시 인덱스 검색 (PostgreSQL 10 이상 권장)
EXPLAIN ANALYZE SELECT content FROM textfun WHERE content = 'example'::text;
```

---

## 9️⃣ 해싱을 활용한 대규모 데이터 처리

### **SHA-256 해시 적용**
```sql
SELECT sha256('hello');
```

### **UUID를 활용한 해시 적용**
```sql
CREATE TABLE cr4 (
    id SERIAL,
    url TEXT,
    url_md5 UUID UNIQUE,
    content TEXT
);

UPDATE cr4 SET url_md5 = md5(url)::UUID;
```

---

## 🔟 PostgreSQL의 정규 표현식 활용

### **문자열 특정 패턴 검색**
```sql
SELECT * FROM users WHERE email ~* '^[a-z]+@[a-z]+\.[a-z]+$';
```

### **특정 문자 포함 여부**
```sql
SELECT * FROM comments WHERE comment_text ~ 'error|fail|bug';
```

### **문장에서 특정 패턴 추출**
```sql
SELECT regexp_matches(comment_text, '#([A-Za-z0-9_]+)', 'g') FROM comments;
```

---

## 📌 **마무리**
이 문서에서는 PostgreSQL에서의 **텍스트 처리, 인덱스 활용, 해싱, 성능 최적화, 정규 표현식 활용**에 대해 다루었다.  
실제 프로젝트에 적용하면서 연습하면 더욱 효과적으로 PostgreSQL을 활용할 수 있다! 🚀
