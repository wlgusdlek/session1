# Elasticsearch 강의 노트

이 문서는 **Elasticsearch**의 개념과 아키텍처, 활용 방법을 다룹니다. Elasticsearch는 오픈 소스 검색 엔진으로, **대규모 데이터 검색 및 분석**을 위해 사용됩니다.

---

## 1️⃣ Elasticsearch 개요

### **1.1 Elasticsearch란?**
- **오픈 소스 검색 엔진**으로, Apache Lucene 기반.
- **확장 가능(Scalable)**한 데이터 검색 시스템.
- **역색인(Inverted Index) 기반**의 전체 텍스트 검색 지원.
- **랭킹(Relevance) 및 추천 시스템** 기능 제공.
- **NoSQL 데이터베이스 기능**을 포함.
- **Google과 유사한 검색 기능 제공**.

### **1.2 Elasticsearch의 역사**
- 기존의 검색 엔진을 **오픈 소스로 제공하고자 하는 목적**에서 개발됨.
- 초창기에는 단순 검색 엔진이었으나, **분산 데이터베이스로 발전**.
- 현재는 **ELK Stack (Elasticsearch, Logstash, Kibana)**의 핵심 구성 요소.

---

## 2️⃣ Elasticsearch의 라이선스 및 적용 사례

### **2.1 라이선스**
- **"오픈 코어(Open Core)" 모델**을 따름.
- 핵심 기능은 **Apache 2.0 라이선스**로 제공됨.
- Elastic 회사는 **추가적인 서비스(호스팅, 컨설팅, 프리미엄 기능)**을 판매.

### **2.2 대표적인 활용 사례**
1. **Sakai**: 오픈 소스 학습 관리 시스템(LMS)에서 검색 엔진으로 사용.
2. **ELK Stack**:
   - **Elasticsearch**: 분산형 NoSQL 데이터베이스.
   - **Logstash**: 실시간 데이터 수집 및 변환.
   - **Kibana**: 데이터 시각화 및 대시보드 제공.

```bash
# Kibana 대시보드 링크
https://www.elastic.co/guide/en/kibana/current/dashboard.html
```

---

## 3️⃣ Elasticsearch 아키텍처

### **3.1 기본 아키텍처**
- **RESTful API** 기반의 웹 서비스.
- **JSON 형식의 데이터 저장 및 검색**.
- **이벤트 기반(Eventual Consistency)** 데이터 모델.

### **3.2 주요 개념**
- **인덱스(Index)**: 데이터가 저장되는 기본 단위.
- **도큐먼트(Document)**: JSON 형식으로 저장되는 개별 데이터.
- **샤드(Shard)**: 데이터를 여러 서버에 분산 저장하여 성능을 향상.
- **노드(Node)**: 개별적인 Elasticsearch 서버 인스턴스.
- **클러스터(Cluster)**: 여러 개의 노드가 모여 하나의 검색 엔진을 형성.

```json
{
    "index": "my_index",
    "type": "_doc",
    "id": "1",
    "body": {
        "title": "Elasticsearch Tutorial",
        "content": "Elasticsearch is a powerful search engine"
    }
}
```

---

## 4️⃣ Elasticsearch의 데이터 저장 및 검색

### **4.1 기본적인 검색 쿼리 (Match All Query)**
모든 문서를 검색하는 가장 기본적인 쿼리.

```json
GET /_search
{
    "query": {
        "match_all": {}
    }
}
```

**Python을 활용한 검색 예제:**
```python
import requests
import json

query_url = "http://pg4e_86f9:*@es.py4e.com:9210/prefx/testindex/_search?pretty"
query_body = json.dumps({"query": {"match_all": {}}})
headers = {'Content-type': 'application/json; charset=UTF-8'}

response = requests.post(query_url, headers=headers, data=query_body)
print(response.json())
```

---

## 5️⃣ Python과 Elasticsearch 연동

### **5.1 Python Elasticsearch 라이브러리 설치**
```bash
pip3 install elasticsearch
```

### **5.2 Elasticsearch 연결 및 검색**
```python
from elasticsearch import Elasticsearch

es = Elasticsearch(
    ["http://localhost:9200"],
    http_auth=("user", "password"),
    scheme="http"
)

# 검색 쿼리 실행
res = es.search(index="testindex", body={"query": {"match_all": {}}})
print(res)
```

---

## 6️⃣ Elasticsearch의 주요 기능

### **6.1 전체 텍스트 검색**
- 역색인(Inverted Index)을 활용하여 빠른 검색 수행.
- 자연어 처리(NLP)와 연동 가능.

### **6.2 분산 검색 및 데이터 저장**
- **샤딩(Sharding)**을 통해 대규모 데이터를 여러 노드에 저장.
- **레플리케이션(Replication)**으로 고가용성(High Availability) 보장.

### **6.3 실시간 데이터 분석**
- **ELK Stack (Elasticsearch + Logstash + Kibana)**을 통해 **로그 및 실시간 이벤트 분석** 가능.

---

## 7️⃣ Elasticsearch의 장점 및 활용

### **7.1 장점**
✅ **빠른 검색 성능**: 역색인을 활용하여 대량의 데이터를 빠르게 검색.  
✅ **확장성(Scalability)**: 수평 확장이 가능하여 데이터 증가 시 손쉽게 확장 가능.  
✅ **JSON 기반의 REST API**: 간편한 API 연동 가능.  
✅ **NoSQL 데이터베이스 기능**: 문서 저장 및 쿼리 기능 제공.  

### **7.2 Elasticsearch를 활용한 검색 시스템**
- **검색 엔진**: 웹사이트 검색, 문서 검색, 로그 분석.
- **추천 시스템**: 사용자의 관심사를 기반으로 추천.
- **실시간 모니터링**: 로그 및 보안 이벤트 감지.

```json
{
    "query": {
        "match": {
            "content": "machine learning"
        }
    }
}
```

---

## 8️⃣ 결론 및 요약

- **Elasticsearch는 Apache Lucene 기반의 오픈 소스 검색 엔진**.
- **RESTful API와 JSON 데이터 포맷을 사용하여 확장성과 접근성이 뛰어남**.
- **NoSQL 데이터베이스로 활용 가능하며, 로그 분석 및 실시간 데이터 처리가 가능**.
- **ELK Stack과 함께 사용하면 로그 수집, 저장, 분석 및 시각화를 한 번에 수행 가능**.

---

## 📌 참고 자료

- [Elasticsearch 공식 문서](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Python Elasticsearch 라이브러리](https://elasticsearch-py.readthedocs.io/en/master/)
- [Elasticsearch GitHub](https://github.com/elastic/elasticsearch)
