## 📖 DB Key & Index 설계
### 1. Key
#### 1.1. Roles of Key
- **무결성(Integrity)**: 행을 유일하게 식별하거나 관계를 정의하여 데이터의 부패를 방지
- **성능(Performance)**: 인덱스와 결합하여 검색 범위를 기하급수적으로 줄임
---
#### 1.2. Key Types
| Key Type  | 역할  | 특징 / 주의점  |
| --  | -- | --  |
| **Primary Key (PK)** | 테이블 행 고유 식별               | - NULL 불가<br>- 단일 컬럼 또는 Composite 가능<br>- 자동 인덱스 생성(B-Tree)<br>- 조회/조인 성능 기준     |
| **Unique Key**       | 값의 유일성 보장                 | - NULL 허용 여부 DB별 차이<br>- 자동 인덱스 생성<br>- PK 외 유일성 확보용                             |
| **Foreign Key (FK)** | 참조 무결성 보장                 | - 관계 무결성 유지<br>- FK 컬럼 자체는 자동 인덱스가 안 될 수 있음 → 조회용 인덱스 별도 필요<br>- Composite FK 가능 |
| **Composite Key**    | 다중 컬럼 조합                  | - PK, Unique, Index 모두 가능<br>- **앞쪽 컬럼이 인덱스 성능 결정**                              |
| **Candidate Key**    | PK 후보                     | - 테이블 내 유일성 보장 가능한 컬럼<br>- PK 선택 시 고려<br>- 실무에서는 Unique Key로 구현                  |
| **Alternate Key**    | PK로 선택되지 않은 Candidate Key | - Unique 제약으로 유지<br>- 조회 성능 최적화 시 고려                                             |
| **Surrogate Key**    | 자연키 대신 생성한 인공 키           | - AUTO_INCREMENT, UUID 등<br>- 복합 자연키 대신 단순 PK로 사용, 조인/정렬 효율 ↑                    |
| **Natural Key**      | 실제 데이터 의미 있는 컬럼           | - 예: 이메일, 주민번호<br>- 변경 가능성 고려, 조인/수정 시 주의                                        |

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