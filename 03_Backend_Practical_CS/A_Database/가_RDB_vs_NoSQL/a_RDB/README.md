## 📖 RDB CheatSheet
```sql
SELECT   user_id, COUNT(*) as cnt      -- 5: 어떤 컬럼을 가져올 것인가?
    FROM     orders                    -- 1: 어느 테이블에서?
    WHERE    status = 'SHIPPED'        -- 2: 어떤 조건의 행들만? (필터링)
    GROUP BY user_id                   -- 3: 어떤 기준으로 그룹핑할 것인가?
    HAVING   cnt > 5                   -- 4: 그룹핑된 결과 중 어떤 조건?
    ORDER BY cnt DESC                  -- 6: 어떤 순서로 정렬할 것인가?
    LIMIT    10;                       -- 7: 몇 개나 보여줄 것인가?
```
```sql
-- 조인 (Join) 
SELECT m.name, o.product_name
    FROM members m
    JOIN orders o ON m.id = o.member_id  -- 교집합
    WHERE m.status = 'ACTIVE';

-- 합계, 평균, 개수 (Aggregate)
SELECT SUM(price), AVG(price), COUNT(*) 
    FROM products;

-- 데이터 수정/삭제  
UPDATE members SET last_login = NOW() WHERE id = 10;
DELETE FROM members WHERE id = 10;
```
---
### 1. SQL 분류
| 구분 | 풀네임 | 목적 | 대표 명령어  |
| -- | -- | -- | -- |
| **DDL** | Data Definition Language     | 스키마 정의  | `CREATE`, `ALTER`, `DROP`, `TRUNCATE`  |
| **DML** | Data Manipulation Language   | 데이터 조작  | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| **DCL** | Data Control Language        | 권한 관리   | `GRANT`, `REVOKE`                      |
| **TCL** | Transaction Control Language | 트랜잭션 제어 | `COMMIT`, `ROLLBACK`, `SAVEPOINT`      |
---
#### 1.1. DDL 
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL,
    deleted_at TIMESTAMP NULL
);
```
| 항목            | 주의 포인트                 | 이유                |
| ------------- | ---------------------- | ----------------- |
| `ALTER TABLE` | 운영 중 대규모 테이블 주의        | MySQL은 메타데이터 락 발생 |
| 컬럼 타입         | VARCHAR 남발 금지          | 인덱스 효율 저하         |
| PK 설계         | Auto Increment vs UUID | 클러스터드 인덱스 성능      |
| NULL 허용       | 최소화 권장                 | 옵티마이저 통계 왜곡       |

> MySQL InnoDB에서 PK = 클러스터드 인덱스
> → PK 변경 = 테이블 재구성 비용 큼
---
#### 1.2. DML 
```sql
SELECT * FROM users WHERE email = 'a@b.com';
INSERT INTO users (id, email) VALUES (1, 'a@b.com');
UPDATE users SET email = 'c@d.com' WHERE id = 1;
DELETE FROM users WHERE id = 1; -- 실무에서는 Soft Delete
```
| 항목              | 문제           | 해결       |
| --------------- | ------------ | -------- |
| WHERE 없는 UPDATE | 전체 락         | 조건 필수    |
| 대량 DELETE       | Undo/Redo 폭증 | 배치 처리    |
| SELECT *        | 불필요 I/O      | 컬럼 명시    |
| OR 조건           | 인덱스 무력화      | UNION 분해 |
---
#### 1.3. DCL 
| 명령어    | 예시                                 | 주의사항     |
| ------ | ---------------------------------- | -------- |
| GRANT  | `GRANT SELECT ON db.* TO user;`    | 최소 권한 원칙 |
| REVOKE | `REVOKE DELETE ON db.* FROM user;` | 운영 계정 분리 |

> 애플리케이션 계정 ≠ 관리자 계정
> 운영 장애의 70%는 권한 오남용
---
#### 1.4. TCL  
```sql
START TRANSACTION;
UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;
COMMIT;
```
---
#### 1.5. Isolation Level & Side Effect
| Level            | Dirty Read | Non-Repeatable | Phantom | 비고            |
| ---------------- | ---------- | -------------- | ------- | ------------- |
| READ UNCOMMITTED | O          | O              | O       | 거의 사용 X       |
| READ COMMITTED   | X          | O              | O       | PostgreSQL 기본 |
| REPEATABLE READ  | X          | X              | △       | MySQL 기본      |
| SERIALIZABLE     | X          | X              | X       | 성능 ↓          |

> MySQL RR에서도 Phantom 방지 위해 Gap Lock 사용
> Gap Lock → 예상치 못한 데드락 원인
---
### 2. Index
| 종류 | 특징        | 사용 시점     |
| --------- | --------- | --------- |
| B-Tree    | 범위 조회 최적  | 대부분       |
| Hash      | 동등 비교     | In-memory |
| Composite | 선행 컬럼 중요  | 다중 조건     |
| Covering  | 테이블 접근 없음 | 조회 성능 최고  |

#### 인덱스 설계 원칙
- 카디널리티 높은 컬럼을 앞에 
- WHERE -> ORDER BY -> LIMIT 순서 고려
- 인덱스 = 읽기 성능 ↑ / 쓰기 성능 ↓
---
### 3. 실행 계획 (EXPLAIN)
```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@b.com';
```
| 항목 | 의미 | 체크 |
| -- | -- | -- |
| type | 접근 방식   | `ALL`이면 위험 |
| key  | 사용 인덱스  | NULL이면 미사용 |
| rows | 예상 스캔 수 | 많으면 튜닝     |

---
### 4. Lock & Deadlock
| Lock | 설명 | 주의 |
| -- | -- | -- |
| Shared    | 읽기           | SELECT FOR SHARE |
| Exclusive | 쓰기           | UPDATE/DELETE    |
| Gap Lock  | 범위 보호        | MySQL RR         |
| Next-Key  | Record + Gap | 팬텀 방지            |

#### 데드락 방지 전략
- 동일한 순서로 테이블 접근
- 짧은 트랜잭션 유지
- 불필요한 SELECT FOR UPDATE 제거
---
### 5. Architecture
| 주제 | 핵심 포인트 |
| -- | -- |
| Replication     | Lag 발생 → Read 분산 주의    |
| Sharding        | PK 기반 분할               |
| Connection Pool | Too Many Connection 방지 |
| Buffer Pool     | DB 메모리 튜닝 핵심           |

---
### Summary
- Explain은 습관이다.
- Transaction은 최대한 짧게 가져간다.
- Index는 쓰기 성능을 깎아먹으니 꼭 필요한 것만 만든다.
---