---
title: Elasticsearch 1
author: Jiny
date: 2021-08-31 14:30:00 +0800
categories: [ElasticSearch]
tags: [elasticsearch, bigdata, elk]
toc: false
---
 
# Elasticsearch 1

___

## 💿 **환경설정**

### **방법**

1. elasticsearch.in.sh, elasticsearch.yml의 파일을 변경
2. es실행 시 -D* 또는 --*옵션을 이용
3. es 실행 후 REST API를 이용

### **yml**

- 클러스터
  - 별도로 설정하지 않으면 클러스터 명은 elasticsearch로 설정
- 노드
  - es는 하나 이상의 노드로 구성된다. 
    - 실행된 하나의 es 프로세스를 한 노드라고 하고 각 노드가 연결된 전체 시스템을 elasticsearch cluster