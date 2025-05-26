---
title:
description:
author: codongmin
date: 2025-02-01T21:34:00
categories:
  - 자바
tags: [""]
image:
  path: /assets/img/thumbnail/.png
  lqip:
  alt: 이미지 설명
---


## 엘라스틱 서치 

Elasticsearch는 오픈소스 기술 스택으로, **검색 및 분석에 특화된 엔진**이다. 대량의 데이터를 빠르게 검색하고 필터링, 복잡한 조건으로 검색하는 기능이 뛰어나다. 

Elasticsearch는 **Apache Lucene**이라는 Java의 **고성능 오픈소스 검색 엔진 라이브러리**를 기반으로 만들어졌다. Lucene은 다음과 같은 기능들을 핵심 개념들을 제공한다. 
- 역색인(Inverted Index) 
- 검색 기능(필드검색, 토큰화, 랭킹)
- 다양한 강력한 쿼리 제공(구문 쿼리, 와일드카드 쿼리, 근접 쿼리, 범위 쿼리 등)

즉, Lucene은 Elasticsearch의 **핵심 뼈대(엔진)** 이고, 이를 기반으로 **기능을 확장하고, 분산처리가 가능**하도록 만들어진 것이 Elasticsearch이다.


## 엘라스틱서치 특징

Elasticsearch은 크게 다음과 같은 특징들을 갖는데, 주로 대규모 데이터에 적합, 수평 확장성, 안정성, 고가용성을 갖는것이 가장 두드러지는 특징이지 않을까 싶다. 

### 분산 특성

클러스터 내 사용 가능한 모든 노드에 걸쳐 데이터와 쿼리 부하를 자동으로 분산시킵니다4. 이를 통해 대규모 데이터를 거의 실시간으로 처리할 수 있습니다


## 엘라스틱서치 사용 예시


### 추천 & 개인화 



## 기본 데이터 단위

### Document & Field
Document는 Elasticsearch는 가장 **기본 데이터 단위**이다. RDB에서의 row(행)에 근접한 데이터 구조이며, Elasticsearch를 확장할때, Document를 단위 기준으로 여러 노드로 분산되어 저장된다. 

기본적으로 각 Document는 JSON 형태로 표현이 된다.

```json
{
  "id": 123,
  "title": "Spring Boot와 Elasticsearch 연동하기",
  "body": "이번 글에서는 Spring Boot 애플리케이션에서 Elasticsearch를 연동하는 방법을 설명합니다.",
  "authorId": 45,
  "authorName": "홍길동",
  "tags": ["spring", "elasticsearch", "java"],
  "createdAt": "2025-05-26T09:30:00",
  "updatedAt": "2025-05-26T10:00:00"
}

```

Field는 Elasticsearch에서 가장 작은 단위가 되는 값이다. 위의 Document에서 JSON형식의 Key-Value 쌍을 의미한다. 위의 Document 내부의 다음과 같은 값들을 필드라고 부른다.
```json
"authorId": 45
"authorName": "홍길동"
```

### Indices

Elasticsearch는 데이터들이 인덱스로 구성된다. 각 인덱스는 RDB에 대응했을때 테이블이라는 개념에 가깝다. 이 인덱스에 저장되는 데이터의 기본 단위는 위에서 알아본 Document가 된다. 다음과 같은 구조를 갖는다.

```json
PUT /posts
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "id":         { "type": "long" },
      "title":      { "type": "text", "analyzer": "nori" },
      "body":       { "type": "text", "analyzer": "nori" },
      "authorId":   { "type": "long" },
      "authorName": { "type": "keyword" },
      "tags":       { "type": "keyword" },
      "createdAt":  { "type": "date" },
      "updatedAt":  { "type": "date" }
    }
  }
}
```

Document가 인덱스에 추가될 때, Elasticsearch는 내부적으로 검색에 최적화된 형식으로 변환하여 인덱싱한다.


---

## 레플리카와 샤딩


### 샤딩 (Sharding)
- **인덱스를 작은 단위로 분할한 것** (하위 데이터 단위: _샤드_)
- 예: 100GB 인덱스를 5개 샤드로 나누면 각 샤드는 20GB
- 샤드는 **노드 간 분산 저장**되어 **병렬 검색**과 **수평 확장**을 가능하게 함

### 레플리카 (Replica)

- **샤드의 복제본**
- 기본 목적:
    1. **장애 대비** (Primary 샤드 손상 시 Replica가 대체)
    2. **검색 부하 분산** (읽기 요청을 여러 노드가 나눠 처리)

---



## 엘라스틱 아키텍처 

### 데이터 분산 저장

각 인덱스는 여러개의



## 엘리스틱서치의 확장, 안정성

Elasticsearch는 수평적으로 확장할 수 있도록 설계되었다. 데이터가 더 많아지거나, 트래픽이 더 많아질때 대비가 가능하다.

## Repliaction , 샤딩