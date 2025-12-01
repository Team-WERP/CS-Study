# Elasticsearch

## 1. Elasticsearch 기본 개념

### 1.1. Elasticsearch란?

**Elasticsearch는 Apache Lucene 기반의 오픈소스 분산형 검색 엔진이다.**

Lucene은 역색인 구조를 통해 뛰어난 텍스트 검색 성능을 제공한다. Elasticsearch는 이를 유지하면서 아래 기능을 추가해 실제 서비스에서 바로 사용할 수 있을 만큼 완성된 형태로 제공한다.

- 분산 저장 및 자동 샤딩
- 자동 복제(Replica) 및 장애 복구
- 실시간에 가까운 검색 성능(NRT)
- REST API 기반의 간단한 연동

** Lucene은 라이브러리여서 사용하려면 추가 개발이 필요함

<br>

### 1.2. 핵심 특징

**1) 실시간에 가까운 빠른 검색**
- 색인 → 검색 사이클이 매우 짧음(약 1초 내)
- 역색인 구조 덕분에 텍스트 검색 성능이 DB보다 압도적으로 빠름

**2) 분산 저장 & 자동 확장 (Scale-Out)**
- 인덱스를 샤드(shard) 단위로 나누어 여러 노드에 자동 분산
- - 노드만 추가하면 용량/성능이 자동 증가

**3) JSON 기반의 유연한 문서 모델**
- Schema 정의 없이 JSON 그대로 저장 가능
- 구조가 조금 달라져도 문제 없음(스키마리스)

**4) 강력한 Aggregation(집계) 기능**
- SQL의 GROUP BY보다 훨씬 유연하고 빠른 집계 가능
- 로그 분석, 매출 분석, 사용자 행동 분석에 강함

**5) REST API 기반**
- HTTP 명령(GET/POST/PUT/DELETE)으로 데이터 조회·삽입·수정
- 언어/플랫폼에 관계없이 쉽게 연동 가능

<br>

## 2. Elasticsearch vs RDB

### 2.1. 용어 매핑

<img width="800" src="https://github.com/user-attachments/assets/1a9676ef-c88d-491b-bb39-b0803626e251" />
<img width="800" src="https://github.com/user-attachments/assets/21a35c34-22e2-4422-aa2b-d2fc84903923" />


- Index = Database
- Document = Row
- Field = Column

### 2.2. CRUD 방식의 차이

<img width="1280" height="307" alt="image" src="https://github.com/user-attachments/assets/027eeb26-0e11-4f0c-b6f2-21dfb0dcde9b" />

- RDB: SQL을 사용해 직접 DB 서버에 질의
- ES: REST API로 조작 (GET, POST, PUT, DELETE)

** 다만, 데이터 삽입인 POST는 RDBMS와 다르게 스키마가 미리 저장되어 있지 않더라도 자동으로 필드를 생성하고 저장한다.

<br>

## 3. Elasticsearch 핵심 개념

### 3.1. 역색인(Inverted Index)

Elasticsearch는 데이터를 저장할 때 **역색인(Inverted Index)** 구조를 사용한다. 때문에 문서를 처음부터 스캔하지 않아도 특정 단어가 등장한 문서를 즉시 찾을 수 있으며, 전문 검색(Full-text Search)에 매우 강하다.

**예시**

<img width="601" height="452" alt="image" src="https://github.com/user-attachments/assets/10c528da-8e43-4936-8a87-7d34f6b4325a" />

여러 개의 문서(Document)가 있을때

- Document 1: `"The bright blue butterfly hangs on the breeze."`
- Document 2: `"It's best to forget the great sky and to retire from every wind."`
- Document 3: `"Under blue sky, in bright sunlight, one need not search around."`

이 문서들은 바로 인덱싱되지 않고 아래의 과정을 거친다.

1. 토큰화(Tokenization)
    - 문장을 공백·구두점 등을 기준으로 단어 단위(토큰)로 분리한다.
    - 예시: “The bright blue butterfly hangs on the breeze”
    →["the", "bright", "blue", "butterfly", "hangs", "on", "the", "breeze"]
2. 불용어(Stopword) 제거
    - 검색 품질을 해치지 않는 일반적인 단어(예: the, a, and, on, in 등)는 제거한다.
    - 그림 중앙의 Stopword list가 그 역할을 한다.
    - 예시:
        
        "the", "on", "in", "a" → 제거
        
        "bright", "blue", "butterfly" → 유지
        
3. 역색인(Inverted Index) 생성
    - 보통 DB는 “문서 → 포함된 단어” 순서로 저장된다.
    - 하지만 역색인은 반대로 “단어 → 해당 단어가 등장한 문서 목록” 형태로 저장한다.
    - 예시
        
        | Term | Document |
        | --- | --- |
        | bright | 1, 3 |
        | blue | 1, 3 |
        | breeze | 1 |
        | best | 2 |
        | forget | 2 |
        | search | 3 |
        | sky | 2, 3 |
        
        → **"bright"라는 단어가 어떤 문서에 등장하는가? :** Document 1, Document 3
        
        이 정보를 단번에 알 수 있기 때문에 검색 시 모든 문서를 처음부터 훑어볼 필요가 없다.

<br>

|관계형DB|Elasticsearch|
|--|--|
|<img width="1260" height="624" alt="image" src="https://github.com/user-attachments/assets/34354372-785f-4695-86c6-1c791a89d0a2" />|<img width="1270" height="798" alt="image" src="https://github.com/user-attachments/assets/bba8e284-9f8a-4a85-a1c7-9f0700adf494" />|

<br>

### 3.2. 클러스터, 노드, 샤드 구조

Elasticsearch는 기본부터 **분산 시스템**으로 설계된다.

**Cluster → Node → Index → Shard → Document**

- **Cluster (클러스터)**
    - 가장 큰 단위로, 여러 개의 Elasticsearch 노드(Node)가 모여 하나의 클러스터를 구성한다.
    - 애플리케이션에서는 클러스터 전체를 하나의 거대한 검색 엔진처럼 사용한다.
- **Node (노드)**
    - Elasticsearch가 실행되는 단일 서버 또는 인스턴스이다.
    - 노드가 여러 개 있을수록 저장 용량과 처리 성능이 향상된다.

- **Index (인덱스)**
    - 검색·저장 단위
- **Shard (샤드)**
    - Index(인덱스)의 데이터를 여러 조각(Shard)으로 나누어 저장한 단위이다.
    - 예를 들어 인덱스를 3개의 샤드(P0, P1, P2)로 나누면 데이터가 Node 1·Node 2·Node 3으로 분산 저장된다.
    - **예시**
        - **Primary Shard(P0, P1, P2)**: 실제 데이터를 저장하는 원본 샤드
        - **Replica Shard(R0, R1, R2)**: Primary의 복제본으로, 장애 시 데이터 보호 + 읽기 성능 향상

<img width="1176" height="516" alt="image" src="https://github.com/user-attachments/assets/695e552b-b349-4089-8362-4159c773fbed" />

- 인덱스는 여러 샤드로 나뉘어 여러 노드에 분산 저장됨
- Primary 샤드와 Replica 샤드가 서로 다른 노드에 배치되어 장애에 강함
- 노드를 추가하면 샤드를 재분배하면서 자동으로 **수평 확장(scale-out)** 가능

<br>

### 3.3. RDB와의 주요 차이점

1. **스키마 유무**
    - RDB: 반드시 사전에 테이블 구조를 정의해야 함
    - ES: 스키마리스(Schemaless), 데이터 입력 시 자동으로 필드 생성
2. **데이터 형식**
    - RDB: 행(Row)과 열(Column) 구조 → **정형 데이터 중심**
    - ES: 문서(Document) 기반 JSON 저장 → **정형 + 비정형 데이터 모두 처리 가능**
3. **검색 방식**
    - RDB: 값 매칭 중심 (=, LIKE '%keyword%')
    - ES: **역색인(Inverted Index)** 기반 → 전문 검색(Full-text Search)에 강점
4. **확장성**
    - RDB: 주로 **수직적 확장(Scale-Up)** → 더 좋은 서버 필요
    - ES: **수평적 확장(Scale-Out)** → 여러 노드에 데이터를 샤드 단위로 분산 저장
5. **실시간성**
    - RDB: 트랜잭션 중심 → 데이터 안정성이 강점
    - ES: 준실시간(NRT) → 빠른 검색과 분석이 강점

<br>

## 4. Elasticsearch 활용

Elasticsearch는 많은 데이터를 빠르게 검색, 집계, 분석해야 하는 상황에서 사용된다.

### 4.1. 주요 활용 사례

1. 검색 서비스
    - 쇼핑몰 검색, 문서 검색
    - 자동완성, 추천 검색어
2. 로그 분석/모니터링
    - 서버·애플리케이션 로그 실시간 분석
    - ELK / Elastic Stack 구축 (Kibana 대시보드)
3. 데이터 분석
    - 시간 기반 분석(트래픽 추이)
    - 사용자 행동 분석
    - 지리 기반 분석(Geo)

<br>

### 4.2. Elastic Stack (ELK 확장판)

- 로그 분석 및 관찰 (로그 수집 → 정제 → 저장 → 분석 → 시각화)  >> Elasticsearch 단독으로는 부족하다.
- 이 과정을 통합적으로 처리할 수 있도록 Elastic이 제공하는 전체 제품군을 Elastic Stack이라고 한다.

<br>

**Elastic Stack 구성요소**

<img width="1018" height="350" alt="image" src="https://github.com/user-attachments/assets/da8ea232-10b4-4538-ae36-789325f47ce3" />

1. **Elasticsearch**: 저장·검색·분석 엔진
2. **Logstash**: 데이터 수집 및 파싱
3. **Kibana**: 시각화 대시보드/UI
4. **Beats**: 경량 로그·메트릭 수집기 (Filebeat 등)
5. **APM**: 애플리케이션 성능/오류 모니터링
6. **Security**: 권한/SIEM, 보안 분석
7. ...

** 초기에는 Elasticsearch + Logstash + Kibana 3가지 요소만 포함되어 **ELK Stack**이라고 불렸으나, 이후 다양한 기능들(Beats, APM 등)이 추가되면서 확장된 전체 생태계를 Elastic Stack이라는 이름으로 통합하고 있다.

<br>

## 🔗 출처

- https://youtu.be/LIw-UzSM7iQ?si=beQHvjgsv70Flsfv
- https://www.elastic.co/guide/en/elasticsearch/reference/8.19/elasticsearch-intro-what-is-es.html
- https://victorydntmd.tistory.com/308
- https://www.samsungsds.com/kr/insights/elastic_data_modeling.html
- https://ks1171-park.tistory.com/169
- https://velog.io/@dat0802/Elasticsearch%EC%99%80-RDB%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC
