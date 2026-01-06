## 📖 Naming & Formatting
>**가독성은 취향이 아니라 비용 문제다**

### 1. Naming 의 본질
- **이름은 곧 설계다.** : 이름이 짓기 어렵다면, 그 함수나 클래스가 너무 많은 일을 하고 있을 가능성이 높음. **intent**가 잘 들어나야 함.
- **검색 가능한 이름** : `d`보다는 `daysSinceCreation`와 같이 의도가 분명하며 검색이 쉽게 설정
  
> ❗ 이름이 길어지는 문제의 원인은 길이 자체가 아니라
**서로 다른 추상화 수준의 개념이 섞여 있는 설계**다.
---

### 2. Naming Convention
|구분|규칙|나쁜 예 (Bad)|좋은 예 (Good)|
|--|--|--|--|
|변수|명사 중심, 의도 반영|`data`, `temp`, `value`|`userCount`, `maxRetryLimit`|
|Boolean|의문문 접두사 활용|`valid`, `login`|`isValid`, `isLoggedIn`, `hasToken`|
|함수|동사로 시작, 1책임|`process()`, `handle()`|`calculateTotalPrice()`, `validateToken()`|
|컬렉션|단수/복수 구분|`user_list`|`users`(리스트), `user_map`(딕셔너리)|

#### ✅ Boolean 접두어 가이드
|접두어|설명|예시|
|--|--|--|
|is|상태/속성 확인|`isVisible`|
|has|소유 여부 확인|`hasRole`|
|can|가능 여부 확인|`canEdit`|
|should|수행 필요성 확인|`shouldRefresh`|

- Boolean 이름에는 **부정형(not, isNot)** 을 피함.
- 조건문 해석 비용을 최소화하는 것이 목적


#### ✅ 함수명 동사 가이드
```python
# CRUD 작업
get_user()      # 조회
create_order()  # 생성
update_status() # 수정
delete_cache()  # 삭제

# 변환/계산
calculate_total()
convert_to_json()
parse_response()

# 검증
validate_token()
check_permission()
verify_signature()

# 제어/실행
start_server()
stop_process()
trigger_event()
execute_command()
```
---
### 3. Stack Specifics Naming Rules
|스택|변수/함수|클래스/컴포넌트|상수 (Constant)|주요 도구|
|--|--|--|--|--|
|Python|`snake_case`|`PascalCase`|`UPPER_SNAKE`|Black, Flake8|
|Node.js|`camelCase`|`PascalCase`|`UPPER_SNAKE`|ESLint, Prettier|
|React|`camelCase`|`PascalCase`|`UPPER_SNAKE`|Prettier|
|Java/Spring|`camelCase`|`PascalCase`|`UPPER_SNAKE`|Checkstyle|

> Naming Convention은 개인 취향이 아니라 **Formatter와 Linter가 강제하는 팀 규칙**이어야 함.
---
### 4. Refactoring 원칙
#### ① Magic Number 제거
> 매직 넘버는 값의 문제가 아니라 **의미가 코드 밖에 존재한다는 설계 문제**
- ❌ **Before** : 
    ```javascript
    if (status === 8) { ... }
    ```
- ✅ **After** : 
    ```javascript
    const STATUS_PUBLISHED = 8; if (status === STATUS_PUBLISHED) { ... }
    ```
#### ② Noise Words 제거
- 의미 없는 접미어는 책임을 흐림
    | Bad         | Good                         |
    | ----------- | ---------------------------- |
    | UserInfo    | User                         |
    | UserManager | UserService / UserRepository |
    | ProcessData | normalize_features           |
- 이름이 일반적일수록 책임은 불명확해짐

#### ③ Guard Clause (Early Return)
> 중첩된 `if`문을 줄여 논리적 깊이를 최소화 (최대 3단계 미만 권장)
- 중첩은 사고 깊이를 증가시키며, **조건 실패를 먼저 반환** 할 것.
- ❌ **Before** : 
    ```javascript
    if cond:
        if cond2:
            do_something()
    ```
- ✅ **After** : 
    ```javascript
    if not cond: return
    if not cond2: return
    do_something()
    ```

#### ④ 추상화 레벨 일관성
- 하나의 함수/클래스 내부에서는 **하나의 추상화 레벨만 유지**
- ❌ **Before** : 
    ```python
    validate_user()
    hashed = bcrypt.hashpw()
    cursor.execute()
    ```
- ✅ **After** : 
    ```python
    validate_user()
    persist_user()
    ```
---
### 5. Formatting
- **일관성이 최우선**: 팀 내 규칙(Style Guide)을 정하고 반드시 준수 
- **들여쓰기 = 사고 구조**: 들여쓰기는 코드의 논리적 깊이를 나타내며, 깊이가 너무 깊어지지 않도록 관리 
- **한 줄에 하나의 개념**: 가독성을 위해 한 줄에 너무 많은 로직을 넣지 않기
---
### Abstraction
> `How`(구현 세부)를 숨기고, `What`(의모, 목적)만 드러내게 하는 것
#### ① Abstraction Level
- [Def] : 코드를 읽을 때 생각해야 하는 사고의 높이
    | 레벨        | 설명          | 예시                |
    | --------- | ----------- | ----------------- |
    | 높음 (High) | 도메인/비즈니스 의도 | `save_user()`     |
    | 중간 (Mid)  | 알고리즘/로직     | `hash_password()` |
    | 낮음 (Low)  | 라이브러리/인프라   | `bcrypt.hashpw()` |
- ❌ **Before** : 
    ```python
    def process_user_signup(user_data):
        # High Level: 사용자 검증 (의도 중심)
        validate_user(user_data) 
        
        # Low Level: 갑자기 데이터베이스 명령어가 나옴 (구현 중심)
        db_cursor = conn.cursor()
        db_cursor.execute("INSERT INTO users ...") 
        
        # Low Level: 갑자기 암호화 알고리즘이 나옴
        hashed_pw = bcrypt.hashpw(user_data.pw, salt)
    ```
- ✅ **After** : 
    ```python
    def process_user_signup(user_data):
        validate_user(user_data)    # 수준: 높음 (비즈니스)
        encrypt_password(user_data) # 수준: 높음 (비즈니스)
        save_user_to_db(user_data)  # 수준: 높음 (비즈니스)
    ```

#### ② Good Abstraction
- **SLAP**(단일 추상화 수준 원칙) : 하나의 함수 내에 있는 모든 문장은 동일한 추상화 수준 가짐
- **의도의 명확성** : 하위 레벨의 구현(How)을 상위 레벨의 이름(What)으로 캡슐화하여, 읽는 사람이 비즈니스 로직에만 집중하게 함
---