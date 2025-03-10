## 1️⃣ 관계형 데이터베이스 개요

### **1.1 관계형 데이터베이스 설계**

- 관계형 데이터베이스(RDBMS)는 데이터를 테이블의 행과 열로 구성하여 저장합니다.
- 데이터베이스 설계의 목표:
    - **중복 최소화**
    - **데이터 무결성 보장**
- 설계 과정:
    - 개념적 모델(ERD - 엔터티 관계 다이어그램)
    - 논리적 모델
    - 물리적 모델

### **1.2 데이터베이스 설계 핵심 개념**

- **기본 키(Primary Key)**: 테이블에서 각 행을 유일하게 식별하는 값.
- **논리 키(Logical Key)**: 외부 애플리케이션이 조회하는 키.
- **외래 키(Foreign Key)**: 다른 테이블의 기본 키를 참조하는 키.

---

## 2️⃣ 데이터베이스 정규화

### **2.1 정규화(Normalization) 개념**

- **제1 정규형(1NF)**: 중복 컬럼 제거, 데이터 원자성 유지.
- **제2 정규형(2NF)**: 부분 함수 종속 제거 (모든 속성이 기본 키 전체에 의존해야 함).
- **제3 정규형(3NF)**: 이행적 함수 종속 제거 (속성들이 기본 키에만 의존해야 함).
- **BCNF(Boyce-Codd Normal Form)**: 후보 키가 아닌 속성이 결정자가 되지 않도록 개선.

---

## 3️⃣ 다대다(Many-to-Many) 관계

### **3.1 다대다 관계란?**

- 하나의 엔터티가 여러 개의 다른 엔터티와 관계를 가질 수 있는 경우.
- 예제: `학생(Student)`은 여러 개의 `강의(Course)`를 수강할 수 있으며, `강의`도 여러 `학생`을 가질 수 있음.

### **3.2 다대다 관계 구현**

- **연결 테이블(Junction Table, Bridge Table)** 사용하여 해결.

### **예제 스키마**

```sql
CREATE TABLE student (
  id SERIAL PRIMARY KEY,
  name VARCHAR(128),
  email VARCHAR(128) UNIQUE
);

CREATE TABLE course (
  id SERIAL PRIMARY KEY,
  title VARCHAR(128) UNIQUE
);

CREATE TABLE member (
  student_id INTEGER REFERENCES student(id) ON DELETE CASCADE,
  course_id  INTEGER REFERENCES course(id) ON DELETE CASCADE,
  role INTEGER,
  PRIMARY KEY (student_id, course_id)
);
```

### **3.3 다대다 관계 데이터 조회**

### **데이터 삽입**

```sql
INSERT INTO student (name, email) VALUES ('Jane', 'jane@example.com');
INSERT INTO course (title) VALUES ('Python');
INSERT INTO member (student_id, course_id, role) VALUES (1, 1, 1);
```

### **JOIN을 이용한 데이터 조회**

```sql
SELECT student.name, course.title, member.role
FROM student
JOIN member ON member.student_id = student.id
JOIN course ON member.course_id = course.id;
```

---

## 4️⃣ JOIN을 활용한 데이터 조회

### **4.1 INNER JOIN 예제**

```sql
SELECT album.title, artist.name
FROM album
JOIN artist ON album.artist_id = artist.id;
```

### **4.2 CROSS JOIN 예제**

```sql
SELECT track.title, genre.name
FROM track CROSS JOIN genre;
```

### **4.3 다중 JOIN 예제**

```sql
SELECT track.title, artist.name, album.title, genre.name
FROM track
JOIN genre ON track.genre_id = genre.id
JOIN album ON track.album_id = album.id
JOIN artist ON album.artist_id = artist.id;
```

---

## 5️⃣ ON DELETE CASCADE 및 참조 무결성

### **5.1 ON DELETE CASCADE**

- 참조된 레코드가 삭제되면 연결된 레코드도 함께 삭제됩니다.

```sql
DELETE FROM genre WHERE name='Metal';
```

### **5.2 외래 키와 삭제 옵션**

- **RESTRICT**: 부모 키 삭제를 제한.
- **CASCADE**: 부모 키 삭제 시 자식 테이블의 연관 데이터도 삭제.
- **SET NULL**: 부모 키 삭제 시 자식 테이블의 외래 키 값을 NULL로 설정.

---

## 6️⃣ 고급 SQL 기능

### **6.1 JSONB 활용**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    data JSONB
);
INSERT INTO users (data) VALUES ('{"name": "Ham", "age": 30}');
SELECT data->>'name' FROM users WHERE data->>'age' = '30';
```

### **6.2 공통 테이블 표현식 (CTE) 활용**

```sql
WITH recent_users AS (
    SELECT * FROM users WHERE created_at > now() - interval '7 days'
)
SELECT * FROM recent_users;
```

### **6.3 윈도우 함수 활용**

```sql
SELECT name, department, salary,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;
```

### **6.4 트랜잭션 관리**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

---

## 7️⃣ 정리

- **관계형 데이터베이스**는 데이터를 구조적으로 관리하여 무결성을 보장합니다.
- **정규화**를 통해 중복을 제거하고 효율성을 향상합니다.
- **다대다 관계**는 **연결 테이블**을 사용하여 구현됩니다.
- **JOIN 연산**을 통해 여러 테이블을 효과적으로 조회할 수 있습니다.
- **고급 SQL 기능(JSONB, CTE, 윈도우 함수, 트랜잭션)** 을 활용하면 강력한 데이터 처리가 가능합니다.

---

## 📌 참고 자료

- [PostgreSQL 공식 문서](https://www.postgresql.org/docs/)
- [MySQL 공식 문서](https://dev.mysql.com/doc/)
- [SQL Server 공식 문서](https://docs.microsoft.com/en-us/sql/)
