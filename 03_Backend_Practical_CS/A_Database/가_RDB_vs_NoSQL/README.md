## 📖 RDB vs NoSQL
### 1. CAP Theorem
> `CAP` : 분산 데이터 저장소는 아래 3가지 중 동시에 2가지만 만족할 수 있음

> 
| 요소                      | 의미                 |
| ----------------------- | ------------------ |
| Consistency (C)         | 모든 노드가 동일한 데이터를 반환 |
| Availability (A)        | 모든 요청에 응답을 반환      |
| Partition Tolerance (P) | 네트워크 분리 상황에서도 동작   |

> 분산 환경에서 네트워크 분할(P)은 현실에서 피할 수 없으므로
> 실제 선택은 `CP vs AP`의 문제가 된다.
---
#### 1.1. CAP 조합별 실제 시스템
| CAP 조합 | 의미 | 특징 | 대표 시스템 |
| -- | -- | -- | -- |
| **CP** | 정합성 우선   | 일부 요청 차단 가능    | HBase, MongoDB (Replica Set), ZooKeeper |
| **AP** | 가용성 우선   | 일시적 불일치 허용     | Cassandra, DynamoDB, CouchDB            |
| **CA** | 단일 노드 기준 | 분산 환경에서는 성립 불가 | MySQL (Single), PostgreSQL (Single)     |
---
#### 1.2. CAP 조합별 동작 방식
| 상황         | CP 계열    | AP 계열                 |
| ---------- | -------- | --------------------- |
| 네트워크 분할 발생 | 일부 요청 실패 | 모든 요청 응답              |
| 데이터 일관성    | 항상 동일    | Eventually Consistent |
| 쓰기 실패      | 허용       | 거의 없음                 |
| 읽기 결과      | 최신 보장    | 지연 반영 가능              |

---
### 2. Core Philosophy (Model & Trade-off)
| 구분     | RDB             | NoSQL            |
| ------ | --------------- | ---------------- |
| CAP 선택 | CA *(단일 노드 기준)* | AP *(분산 전제)*     |
| 데이터 모델 | Relation (정규화)  | Aggregate (비정규화) |
| 스키마    | Schema-on-Write | Schema-on-Read   |
| 무결성 책임 | DB              | Application      |
| 확장 전략  | Scale-up 중심     | Scale-out 전제     |

> - `RDB`는 정합성 우선
> - `NoSQL`은 가용성·확장성 우선
> - 성능 차이가 아니라 철학 차이
---
### 3. Consistency & Transaction (ACID vs BASE)
| 항목          | RDB (ACID) | NoSQL (BASE)  |
| ----------- | ---------- | ------------- |
| Consistency | Strong     | Eventual      |
| Atomicity   | 보장         | 제한적           |
| Isolation   | MVCC 기반    | 없음 / 제한       |
| Rollback    | 엔진 레벨      | 보상 트랜잭션(Saga) |

> - NoSQL은 “트랜잭션이 없다”기보다
분산 트랜잭션 비용을 피한다
> - 데이터 복구는 재처리/보정 로직에 의존
---
### 4. Query & Performance Model
| 항목          | RDB                | NoSQL                |
| ----------- | ------------------ | -------------------- |
| Join        | Native + Optimizer | 비권장 / App-side       |
| Query       | Ad-hoc Query 강함    | 패턴 기반                |
| Aggregation | SQL 기반             | Pipeline / MapReduce |
| Index       | B-Tree, 다양         | 제한적 필드               |
> - RDB: 쿼리를 먼저 쓰고 데이터 설계
> - NoSQL: 쿼리 패턴을 먼저 고정하고 모델링
---
### 5. Scaling & Architecture
| 항목          | RDB        | NoSQL          |
| ----------- | ---------- | -------------- |
| Replication | Binlog 기반  | Quorum 기반      |
| Sharding    | 수동 설계      | 자동/내장          |
| Rebalancing | 복잡         | 자동 (비용 발생)     |
| 비용 구조       | 고사양 + 라이선스 | 범용 HW + 운영 복잡도 |
> - NoSQL은 노드 추가가 쉬운 대신
운영 복잡도는 애플리케이션으로 전가
---
### 6. Use Case Decision Matrix
| 상황       | 적합한 선택           | 이유              |
| -------- | ---------------- | --------------- |
| 금융 / 정산  | RDB              | 강한 정합성 필수       |
| 주문 / 결제  | RDB              | 상태 전이의 원자성      |
| 로그 / 이벤트 | NoSQL            | 대량 쓰기 + 유실 허용   |
| 캐시 / 세션  | In-memory NoSQL  | 휘발성 데이터         |
| SNS 피드   | NoSQL (Document) | 비정형 데이터 + 대량 조회 |

---
### 📍 Summary
- **정합성이 비즈니스의 핵심인가?** → RDB
- **트래픽 증가 시 수평 확장이 필수인가?** → NoSQL
- **스키마 변경이 잦은가?** → NoSQL
- **복잡한 관계를 Join으로 풀어야 하는가?** → RDB

> 기술 자체의 우열은 없다.
> 비즈니스 요구사항에 따른 Trade-off을 고려하여 선택
---