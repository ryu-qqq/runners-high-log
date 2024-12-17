# 📋 이번 주 목표: 시스템 아키텍처 검토 및 리팩토링

---

## 1. 목표 요약

### 리팩토링 작업 (업무 시간)
- **MySQL 마스터-슬레이브 구조** 읽기 부하 분산 적용 및 검증.
- 기존 **ElasticSearch 인덱싱 배치 제거** 검토.
- **프로덕트 허브 MVP** 개발.

### 퇴근 후 공부 (이번 주 학습)
- **Spring 트랜잭션**과 **AbstractRoutingDataSource** 소스 코드 학습.
- **Grafana 메트릭**과 MySQL 모니터링 지표에 대한 이해.

---

## 2. 업무 시간 리팩토링 작업

### 2.1 MySQL 읽기 부하 분산 리팩토링
**문제**: `ProdRoutingDataSource` 코드가 작동하지 않아 마스터에만 모든 쿼리가 집중.  
**목표**: 마스터-슬레이브 구조를 통해 **읽기 쿼리**를 슬레이브로 분산.

#### 해야 할 작업:
1. **RoutingDataSource 문제 분석**
    - 코드 검토 및 로그 확인: `determineCurrentLookupKey()`가 올바르게 호출되는지 점검.
    - **Spring 트랜잭션**의 `@Transactional(readOnly = true)`가 제대로 적용되었는지 확인.

2. **수정 작업**
    - `TransactionSynchronizationManager.isCurrentTransactionReadOnly()` 동작 여부 확인.
    - 동작하지 않는 경우 **DataSource 설정**을 검토하고 보완.
    - 로그 추가 및 테스트 코드 작성으로 동작 검증.

3. **결과 확인**
    - **Grafana**에서 슬레이브 노드의 쿼리 메트릭 확인.
    - 슬레이브 사용률이 정상적으로 분산되는지 검증.

---

### 2.2 ElasticSearch 인덱싱 배치 제거 검토
**문제**: ElasticSearch 는 시스템 복잡도를 높이며 유지보수 비용이 큼.  
**목표**: MySQL 읽기 부하 분산을 통해 **LIKE 절**로 검색 기능을 대체할 수 있는지 확인.

#### 해야 할 작업:
1. **LIKE 절 성능 테스트**
    - MySQL 테이블에 **Full-Text Index** 추가 후 성능 비교.
    - 데이터 양이 많아질 경우 `LIKE '%keyword%'`의 성능 측정.

2. **ElasticSearch 제거 시 시나리오 정리**
    - 인덱싱 배치 제거 후 검색 기능 대체 가능성 문서화.
    - 러닝 커브를 낮추기 위한 계획 수립.

3. **결과 반영**
    - 성능 테스트 결과에 따라 ElasticSearch 유지 여부 최종 결정.

---

### 2.3 프로덕트 허브 MVP 개발
- **상품 전용 서버**의 **AWS RDS (낮은 스펙 MySQL)** 설정 및 성능 모니터링.
- MVP에서 **읽기/쓰기 트랜잭션** 구조를 설계하면서 **데이터베이스 접근 전략** 정립.

---

## 3. 퇴근 후 공부 (이번 주 학습 목표)

### 3.1 Spring 트랜잭션과 DataSource 동작 원리
**목표**: `@Transactional(readOnly = true)`와 `AbstractRoutingDataSource`의 동작 이해.

#### 학습 내용:
1. **Spring 트랜잭션의 동작 원리**
    - **Propagation**, **Isolation Level**, **readOnly** 옵션의 역할과 차이점 학습.

2. **AbstractRoutingDataSource**
    - `determineCurrentLookupKey()`의 동작과 트랜잭션 **ThreadLocal** 매커니즘 학습.

3. **실습**
    - 다양한 트랜잭션 상황에서 **마스터/슬레이브 라우팅 테스트**.

#### 학습 자료:
- [Spring 트랜잭션 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)
- AbstractRoutingDataSource 코드 분석.

---

### 3.2 Grafana와 MySQL 모니터링 메트릭 학습
**목표**: Grafana에서 수집한 **MySQL 메트릭**과 **Spring 서버 메트릭**의 의미를 정확히 이해.

#### 학습 내용:
1. **MySQL 주요 메트릭**
    - **Replication Lag**: 슬레이브 상태 지연 확인.
    - **Query Performance**: `Threads_running`, `Slow_queries`, `Connections`.
    - **CPU 및 메모리 사용량**: MySQL 프로세스의 자원 사용률.

2. **Spring 서버 메트릭**
    - JVM 메모리 사용량 (`jvm.memory.used`, `jvm.gc.pause`).
    - HTTP 요청 지표 (`http.server.requests.duration`).

3. **실습**
    - Grafana에서 슬레이브 노드 쿼리 메트릭을 분석해 슬레이브 활성화 확인.
    - **Alert 설정 실습**:
        - Replication Lag이 **10초 이상** 발생 시 알림.
        - 슬레이브 노드 사용률이 **일정 이하**일 경우 알림.

#### 학습 자료:
- [Grafana Dashboard 설정 가이드](https://grafana.com/docs/grafana/latest/dashboards/)
- MySQL Exporter 공식 문서.

---

## 4. 주간 목표 정리

### 리팩토링 작업 (업무 시간)
1. **MySQL 읽기 부하 분산 적용 및 테스트**.
2. **ElasticSearch 인덱싱 배치 제거 검토** 및 **LIKE 절 대체 성능 테스트**.
3. **프로덕트 허브 MVP 개발 진행**.

### 퇴근 후 공부
1. **Spring 트랜잭션 및 AbstractRoutingDataSource** 학습 및 실습.
2. **Grafana 및 MySQL 주요 메트릭** 이해를 통한 시스템 분석 능력 강화.

---

**이번 주 목표**  
현 시스템의 문제를 직면하고 성능 최적화 방향을 확립하여 **내실을 다지는 리팩토링**과 **학습**을 병행한다.  
