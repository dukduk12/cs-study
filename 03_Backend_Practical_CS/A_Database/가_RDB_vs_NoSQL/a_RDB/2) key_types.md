## 📖 DB Key & Index 설계
### 1. Key
#### 1.1. Roles of Key
- **무결성(Integrity)**: 행을 유일하게 식별하거나 관계를 정의하여 데이터의 부패를 방지
- **성능(Performance)**: 인덱스와 결합하여 검색 범위를 기하급수적으로 줄임
---
#### 1.2. Key Types
| Key Type  | 역할  | 특징 / 주의점  |
| --  | -- | --  |
| **`Primary Key (PK)`** | 테이블 행 고유 식별  | - NULL 불가<br>- 단일 컬럼 또는 Composite 가능<br>- 자동 인덱스 생성(B-Tree)<br>- 조회/조인 성능 기준     |
| **`Unique Key`**       | 값의 유일성 보장  | - NULL 허용 여부 DB별 차이<br>- 자동 인덱스 생성<br>- PK 외 유일성 확보용                             |
| **`Foreign Key (FK)`** | 참조 무결성 보장  | - 관계 무결성 유지<br>- FK 컬럼 자체는 자동 인덱스가 안 될 수 있음 → 조회용 인덱스 별도 필요<br>- Composite FK 가능 |
| **`Composite Key`**    | 다중 컬럼 조합  | - PK, Unique, Index 모두 가능<br>- **앞쪽 컬럼이 인덱스 성능 결정**  |
| **Candidate Key**    | PK 후보  | - 테이블 내 유일성 보장 가능한 컬럼<br>- PK 선택 시 고려<br>- 실무에서는 Unique Key로 구현   |
| **Alternate Key**    | PK로 선택되지 않은 Candidate Key | - Unique 제약으로 유지<br>- 조회 성능 최적화 시 고려  |
| **Surrogate Key**    | 자연키 대신 생성한 인공 키           | - AUTO_INCREMENT, UUID 등<br>- 복합 자연키 대신 단순 PK로 사용, 조인/정렬 효율 ↑  |
| **Natural Key**      | 실제 데이터 의미 있는 컬럼           | - 예: 이메일, 주민번호<br>- 변경 가능성 고려, 조인/수정 시 주의  |
> 🔑 Key Hierarchy
> 
> `Super Key`: 최소성 상관없이 유일하게 식별만 하면 됨. </br>
> `Candidate Key` (후보키): 유일성 + 최소성(딱 필요한 컬럼만). </br>
> `Primary Key` (기본키): 후보키 중 선택된 단 하나.
> `Alternate Key` (대체키): 후보키 중 탈락한 나머지들.
---
### 2. Index와의 관계
- **PK/Unique** → **자동 인덱스 생성**
- **FK** → MySQL 등에서 **별도 인덱스 필요** (JOIN 성능, Deadlock 방지)
- **Clustered vs Secondary**
  - Clustered Index → PK 컬럼 기준 데이터 정렬
  - Secondary Index → PK 주소 참조(Look-up 필요)
- **Composite Index** → 조회 패턴 기반, 앞쪽 컬럼 선택도가 전체 성능 결정
---
### 3. Cardinality & 선택도
> 특정 컬럼이 가지는 고유한 값의 개수
```text
선택도(Selectivity) = 카디널리티 / 전체 행 수
```
- **카디널리티 높음** → **선택도 높음** → 인덱스 효율 ↑
- **카디널리티 낮음** → **선택도 낮음** → 인덱스 효율 ↓
- **효율적 인덱스**: 선택도 5~10% 이상 컬럼에 사용
- 예시:
  - 성별(M/F), 삭제여부(Y/N) → 선택도 낮음 → 단독 인덱스 비효율
  - 주민번호, 이메일 → 선택도 100% → 인덱스 효율 극대화
- **다중 컬럼 인덱스**: 앞쪽 컬럼 선택도가 낮으면 전체 인덱스 효율 제한

> 💡 팁:
> - 인덱스 설계 시 **조회 패턴 + 카디널리티 + 키 타입**을 같이 고려해야 성능을 보장
> - 불필요한 단독 인덱스는 **쓰기 성능 저하 + 저장공간 낭비**
---
### Suppl. Optimizer
> SQL을 실행할 때 빠르고 비용이 적게 드는 경로를 결정하는 DB의 두뇌(엔진)
- `Index 사용 여부`, `Join 순서`, `Scan 방식` 등을 카디널리티 통계를 바탕으로 판단.
- 실행 계획(`EXPLAIN`)을 확인하여 옵티마이저가 잘못된 경로를 선택했을 때 인덱스를 강제하거나 통계를 갱신(`ANALYZE`)해야 함.
---
### Suppl. Checkpoint
| 항목 | 체크 포인트  |
| -- | -- |
| FK | 자식 테이블에 인덱스 없으면 JOIN/DELETE/UPDATE에서 데드락 발생 가능 |
| Surrogate vs Natural | Natural Key를 PK로 사용하면 조인/정렬 성능 저하, 변경 시 FK 무결성 깨질 수 있음 |
| Composite Key        | 앞쪽 컬럼 선택도 낮으면 전체 인덱스 효율 제한, ORDER BY/PREDICATE 영향      |
---
### 📌 Summary
| 항목  | 주의 내용 | 팁|
| -- | -- | -- |
| FK 인덱스 | 자동 생성 안 됨 | JOIN 컬럼 인덱스는 선택이 아닌 필수 (Deadlock 방지) |
| Composite Key | 앞쪽 컬럼 선택도가 핵심 | Leftmost Prefix 원칙: 가장 많이 필터링되는 컬럼을 왼쪽으로 배치 |
| NULL & Unique | DB별 처리 상이 | 가급적 NOT NULL 설계로 옵티마이저 판단 모호성 제거 |
| Surrogate vs Natural Key  | Natural Key PK 사용 시 조인/정렬 성능 저하, FK 변경 위험 | 대리 키로 PK, Natural Key는 Unique로 관리 |
| 인덱스 오버헤드 | 선택도 낮은 컬럼 단독 인덱스, 자주 업데이트 컬럼에 생성 시 비용 | 무분별한 인덱스는 DB를 무겁게 함. 조회 패턴 기반 최소화 |
| 카디널리티 / 통계 | 옵티마이저 맹신 금지 | 데이터 급증 시 **ANALYZE TABLE**로 통계 최신화 필수 |
| Index Look-up | Secondary Index 조회 시 PK 재탐색 발생 | 커버링 인덱스 고려, 불필요한 Look-up 최소화 |
---
