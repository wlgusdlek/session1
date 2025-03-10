## 1️⃣ 테이블 설계 및 데이터 삽입

### **테이블 생성**

먼저, 이메일 메시지를 저장할 테이블을 생성합니다.

```sql
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    email TEXT,
    sent_at TIMESTAMPTZ,
    subject TEXT,
    headers TEXT,
    body TEXT
);
```

- **컬럼 설명**:
    - `id`: 메시지의 고유 식별자 (자동 증가)
    - `email`: 발신자의 이메일 주소
    - `sent_at`: 이메일 발송 시간 (타임스탬프)
    - `subject`: 이메일 제목
    - `headers`: 이메일 헤더 정보 (원시 메타데이터)
    - `body`: 이메일 본문 (검색 대상)

### **데이터 삽입 예시**

샘플 데이터를 삽입하여 테이블을 채워봅니다.

```sql
INSERT INTO messages (email, sent_at, subject, headers, body) VALUES
('user@example.com', '2025-03-10 10:00:00', 'Hello World', 'Header info', 'This is the body of the email.'),
('admin@example.com', '2025-03-11 12:30:00', 'Meeting Schedule', 'Header info', 'Our next meeting is scheduled for tomorrow at 10 AM.'),
('staff@example.com', '2025-03-12 15:45:00', 'Project Update', 'Header info', 'The project is progressing well. Here are the latest updates.');
```

---

## 2️⃣ 전문 검색 설정

PostgreSQL의 **전문 검색(Full-Text Search)** 기능을 사용하면 자연어 검색을 보다 효율적으로 수행할 수 있습니다.

### **TSVECTOR 및 TSQUERY**

- **`tsvector`**: 텍스트를 분석하여 검색 가능한 형태로 변환하는 데이터 타입
- **`tsquery`**: 검색어를 표현하는 데이터 타입 (논리 연산자 사용 가능)

### **`tsvector` 생성 예제**

다음 명령어를 실행하면 텍스트가 색인화됩니다.

```sql
SELECT to_tsvector('english', 'This is a sample document.');
```

- 결과:
  ```
  'document':5 'sampl':4 'thi':1
  ```

단어가 정규화되었으며, 위치 정보도 포함됩니다.

### **`tsquery` 생성 예제**

```sql
SELECT to_tsquery('english', 'sample & document');
```

- 결과:
  ```
  'sampl' & 'document'
  ```

연산자 `&`는 AND 조건을 의미합니다.

---

## 3️⃣ 전문 검색 인덱스 생성

전문 검색을 최적화하려면 **GIN(Generalized Inverted Index)** 인덱스를 생성해야 합니다.

```sql
CREATE INDEX idx_messages_body ON messages USING GIN(to_tsvector('english', body));
```

- **GIN 인덱스의 장점**:
    - 다중 값이 포함된 텍스트 데이터의 검색 속도를 최적화
    - `to_tsvector()`의 결과를 미리 색인하여 검색 시 빠르게 접근 가능

---

## 4️⃣ 전문 검색 쿼리 실행

### **기본 검색 예제**

```sql
SELECT * FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'meeting & schedule');
```

- `@@` 연산자는 `tsvector`와 `tsquery`를 비교하여 일치하는 경우 `TRUE`를 반환합니다.
- 예제에서는 'meeting'과 'schedule'이 모두 포함된 이메일을 검색합니다.

### **부분 검색**

일부 단어만 포함된 문서를 검색하고 싶다면 `|`(OR 연산자)를 사용할 수 있습니다.

```sql
SELECT * FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'project | update');
```

- `|` 연산자는 OR 조건을 의미합니다.
- 'project' 또는 'update' 단어가 포함된 문서를 검색합니다.

---

## 5️⃣ 가중치 및 순위 설정

전문 검색에서는 **검색 결과의 중요도(랭킹)를 조정**할 수 있습니다.

### **가중치 부여**

제목(subject)에 더 높은 가중치를 부여하고, 본문(body)에는 상대적으로 낮은 가중치를 적용할 수 있습니다.

```sql
SELECT setweight(to_tsvector('english', coalesce(subject, '')), 'A') ||
       setweight(to_tsvector('english', coalesce(body, '')), 'B') AS document
FROM messages;
```

- 'A' > 'B' > 'C' > 'D' 순으로 높은 가중치를 가짐
- 제목(subject)은 중요한 키워드를 포함하는 경우가 많으므로, 높은 우선순위를 부여함

### **순위 계산**

검색된 결과의 순위를 매겨, 더 관련성이 높은 결과를 상위에 표시합니다.

```sql
SELECT ts_rank_cd(to_tsvector('english', body), to_tsquery('english', 'sample & document')) AS rank,
       *
FROM messages
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'sample & document')
ORDER BY rank DESC;
```

- `ts_rank_cd()` 함수는 검색어와 문서 간의 관련성을 평가하여 랭킹을 부여합니다.

---

## 6️⃣ 불용어(Stop Words) 처리

전문 검색에서는 **검색어로서 의미가 적은 단어(불용어, stop words)** 를 필터링할 수 있습니다.

### **사용 가능한 사전 확인**

```sql
SELECT * FROM pg_ts_config;
```

- PostgreSQL에서 지원하는 텍스트 구성 목록을 확인할 수 있습니다.

### **불용어 목록 확인**

```sql
SELECT * FROM ts_debug('english', 'This is a test');
```

- `is`, `a` 같은 단어는 불용어로 처리되어 색인에서 제외됩니다.

---

## 7️⃣ 전문 검색의 실제 적용

### **제목과 본문을 함께 검색**

제목(subject)과 본문(body)을 합쳐서 검색할 수 있습니다.

```sql
SELECT * FROM messages
WHERE to_tsvector('english', subject || ' ' || body) @@ to_tsquery('english', 'project & update');
```

- 제목 또는 본문에 'project'와 'update'가 모두 포함된 메시지를 찾음

---

## 8️⃣ 전문 검색 성능 최적화

### **인덱스 활용**

- `to_tsvector()`를 사용하여 **미리 색인화된 데이터**를 활용하면 검색 성능이 크게 향상됩니다.
- GIN 인덱스를 적용하면 **수백만 개의 행에서도 빠르게 검색 가능**합니다.

### **정규화 및 토큰화**

- **소문자 변환** 및 **불필요한 문자의 제거**를 수행하여 검색 효율을 높일 수 있습니다.

---

## 9️⃣ 전문 검색의 한계와 고려사항

- **언어 지원**: PostgreSQL은 여러 언어를 지원하지만, 각 언어별로 다른 토큰화 및 불용어 처리가 필요합니다.
- **복합 검색**: 여러 컬럼을 동시에 검색할 경우 `tsvector`를 결합하여 인덱스를 최적화해야 합니다.
- **정확도 vs 성능**: 검색 정확도를 높이기 위한 복잡한 쿼리는 성능을 저하시킬 수 있으며, 적절한 균형이 필요합니다.

---

## 📌 참고 자료

- [다이어그램의 파워포인트 슬라이드](https://www.pg4e.com)
- [이 강의의 샘플 SQL 명령](https://www.pg4e.com)
- 강의 URL: [https://www.pg4e.com/lectures/05-FullText](https://www.pg4e.com/lectures/05-FullText)
