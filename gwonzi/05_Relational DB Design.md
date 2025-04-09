# 🗂️ 관계형 데이터베이스 설계(Relational Database Design)

## 1. 소개
- 관계형 데이터베이스 설계는 **데이터의 구조(모양)를 정의하고 그 관계를 모델링**하는 작업.
- 핵심 목표 : **효율적이고 중복 없는 구조로 데이터를 저장**

<br/>

## 2. 데이터 모델링이란?
- 데이터를 **테이블로 나누고**, 각 테이블의 관계를 **그림(ERD: Entity-Relationship Diagram)**으로 나타내는 과정
- 각 테이블은 "박스"로, 외래 키(Foreign Key)는 "선"으로 표현
- 프로젝트 규모가 클수록 이 데이터 모델이 **애플리케이션의 핵심 또는 설계의 근간**이 됨.

<br/>

## 3. 중복 제거의 원칙
### 🎯 목표: 중복 문자열 저장 방지 (Vertical Replication 방지)
- 예: 사용자의 이메일 `csev@umich.edu`가 여러 테이블에서 반복된다면 → ❌ 잘못된 설계
- 해결 방법:
  - 사용자 정보를 `users` 테이블에 저장하고, 이메일은 딱 한 번만 저장
  - 나머지 테이블에서는 사용자 ID(예: 116421)만 참조 → `FOREIGN KEY` 사용

```sql
-- 예시 테이블 구조
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    content TEXT
);
```

<br/>

## 4. 데이터 모델링 예시: 음악 컬렉션
### ❌ CSV 파일/스프레드시트 문제점
- 트랙마다 아티스트, 앨범, 장르를 계속 반복 입력 → "수직 중복"
- 오타 발생 시 전체 수정이 필요 → **유지보수 어려움**

### ✅ 데이터베이스 모델링 접근
- `artists`, `albums`, `tracks`, `genres` 테이블로 분리
- 각각의 문자열은 단 한 번 저장됨
- 관계는 `id`(숫자)를 통해 연결됨 → **정규화된 구조**

```sql
CREATE TABLE artists (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE albums (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    artist_id INTEGER REFERENCES artists(id)
);

CREATE TABLE tracks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    album_id INTEGER REFERENCES albums(id)
);
```

<br/>



## 5. 사용자 인터페이스와 데이터베이스 분리
- UI에서는 중복된 문자열(예: 아티스트명)이 여러 번 보여도 괜찮음 → 검색/정렬/사용자 경험에 중요
- 하지만 **데이터베이스는 중복을 최소화하고 ID로 참조**해야 함
- 올바른 접근:
  1. UI를 그대로 받아들임
  2. **정규화된 DB 설계**로 UI를 재구성할 수 있도록 모델링함

<br/>

## 6. 핵심 설계 원칙 요약
| 원칙 | 설명 |
|------|------|
| 중복 제거 | 동일한 문자열은 단 한 번만 저장 |
| 정규화 | 관련 데이터를 별도의 테이블로 분리하고 관계 정의 |
| 외래 키 사용 | 테이블 간 연결은 ID를 통해 수행 |
| 확장성 고려 | 데이터가 수백만 건이 되어도 성능 유지 가능하도록 설계 |
| 사용자 중심 | UI는 중복되어 보여도 DB는 효율적으로 유지 |

