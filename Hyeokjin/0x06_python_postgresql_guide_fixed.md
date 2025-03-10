## 1️⃣ Python과 PostgreSQL 연동

### **1.1 psycopg2 라이브러리 설치**

Python에서 PostgreSQL과 상호작용하기 위해 `psycopg2` 라이브러리를 설치해야 합니다.

```bash
pip install psycopg2
```

### **1.2 데이터베이스 연결**

PostgreSQL에 연결하는 기본 코드입니다.

```python
import psycopg2

conn = psycopg2.connect(
    host='localhost',
    database='mydatabase',
    user='myuser',
    password='mypassword',
    connect_timeout=3
)
```

### **1.3 커서 생성 및 SQL 실행**

커서를 사용하여 SQL 문을 실행할 수 있습니다.

```python
cur = conn.cursor()

# 테이블 삭제 및 생성
cur.execute('DROP TABLE IF EXISTS pythonfun CASCADE;')
cur.execute('CREATE TABLE pythonfun (id SERIAL, line TEXT);')

# 변경사항 저장
conn.commit()
```

---

## 2️⃣ 예제: 책 텍스트 로드 및 처리

### **2.1 프로젝트 구텐베르크 책 다운로드**

다음 명령어를 사용하여 프로젝트 구텐베르크에서 책을 다운로드할 수 있습니다.

```bash
wget http://www.gutenberg.org/cache/epub/19337/pg19337.txt
```

### **2.2 책 데이터베이스에 로드**

책의 각 단락을 데이터베이스에 저장하는 `loadbook.py` 스크립트를 실행합니다.

```bash
python3 loadbook.py
```

이 스크립트는 `pg19337` 테이블을 생성하고, 책의 내용을 개별 행으로 삽입합니다.

### **2.3 GIN 인덱스 생성 및 검색**

검색 속도를 최적화하기 위해 **GIN 인덱스**를 생성합니다.

```sql
CREATE INDEX pg19337_gin ON pg19337 USING gin(to_tsvector('english', body));
```

이제 특정 키워드를 검색할 수 있습니다.

```sql
SELECT body FROM pg19337
WHERE to_tsquery('english', 'goose') @@ to_tsvector('english', body)
LIMIT 5;
```

---

## 3️⃣ 이메일 메시지 데이터베이스 구축

### **3.1 테이블 생성**

이메일 메시지를 저장하기 위한 테이블을 생성합니다.

```sql
CREATE TABLE IF NOT EXISTS messages (
    id SERIAL PRIMARY KEY,
    email TEXT,
    sent_at TIMESTAMPTZ,
    subject TEXT,
    headers TEXT,
    body TEXT
);
```

### **3.2 데이터 로드**

웹에서 이메일 데이터를 가져와 `messages` 테이블에 저장합니다.

```python
cur.execute(
    "INSERT INTO messages (email, sent_at, subject, headers, body) VALUES (%s, %s, %s, %s, %s)",
    ('user@example.com', '2025-03-10 10:00:00', 'Hello World', 'Header info', 'This is the email body.')
)

conn.commit()
```

### **3.3 GIN 인덱스 생성 및 활용**

이메일 본문에 대한 **전문 검색**을 위해 GIN 인덱스를 생성합니다.

```sql
CREATE INDEX messages_gin ON messages USING gin(to_tsvector('english', body));
```

검색어가 포함된 이메일을 찾으려면 다음과 같은 쿼리를 사용할 수 있습니다.

```sql
SELECT subject, email FROM messages
WHERE to_tsquery('english', 'meeting') @@ to_tsvector('english', body)
LIMIT 10;
```

---

## 4️⃣ Python을 사용한 데이터 조회 및 처리

### **4.1 데이터 조회**

Python에서 데이터베이스의 데이터를 가져오는 코드입니다.

```python
cur.execute("SELECT subject, email FROM messages WHERE body LIKE %s LIMIT 5;", ('%meeting%',))
rows = cur.fetchall()

for row in rows:
    print(row)
```

### **4.2 트랜잭션 처리**

데이터베이스 트랜잭션을 관리하는 방법입니다.

```python
try:
    cur.execute("UPDATE messages SET subject='Updated Subject' WHERE id=1;")
    conn.commit()
except Exception as e:
    conn.rollback()
    print("Error:", e)
```

---

## 5️⃣ 성능 최적화 및 인덱스 활용

### **5.1 GIN 인덱스 활용**

GIN 인덱스를 활용하면 텍스트 검색 성능이 향상됩니다.

```sql
SELECT subject FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'urgent');
```

### **5.2 데이터 정규화 및 최적화**

- **중복된 데이터를 제거**하여 성능을 향상시킵니다.
- **인덱스를 추가**하여 검색 속도를 최적화합니다.

---

## 6️⃣ 정리

- **Python의 `psycopg2` 라이브러리를 사용하여 PostgreSQL과 연결**할 수 있습니다.
- **`psycopg2.cursor()`를 사용하여 SQL 문을 실행**하고 데이터를 저장, 수정 및 조회할 수 있습니다.
- **GIN 인덱스를 활용하여 텍스트 검색 성능을 최적화**할 수 있습니다.
- **트랜잭션 관리 및 오류 처리 기법**을 적용하여 데이터 정합성을 유지합니다.

---

## 📌 참고 자료

- [PostgreSQL 공식 문서](https://www.postgresql.org/docs/)
- [Psycopg2 공식 문서](https://www.psycopg.org/docs/)
- [Project Gutenberg](https://www.gutenberg.org/)
