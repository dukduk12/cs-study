## 📖 Session vs Token 인증 (+Cookie)
> `인증`(Authentication) 설계의 핵심은 
> - ① "상태를 어디에 둘 것인가(`Stateful` vs `Stateless`)"와 
> - ② "어떤 통로로 운반할 것인가(`Cookie` vs `Header`)"를 결정하는 것

<p align="center">
  <img src="../../../00_Docs/가_Images/session-cookie-jwt.jpg" alt="Authentication overview" width="600">
</p>

<p align="center">
  <em>Figure 1. Authentication</em>
</p>

---
### 1. Authentication 의 두 축
#### 1.1. 상태 관리 (State)
|구분|Stateful (Session)|Stateless (Token/JWT)|
|---|---|---|
|상태 위치|서버 (Memory/Redis/DB)|클라이언트|
|서버 책임|“이 사용자가 누구인지 기억”|“서명만 검증”|
|확장성|세션 공유 필요|수평 확장 매우 용이|
|강제 차단|즉시 가능|불가능 (만료까지 유효)|

> 🔑 **JWT를 서버에 저장하는 순간, 구조적으로 Session과 동일해짐**  
> → “Stateless JWT”의 핵심은 *서버가 사용자 상태를 저장하지 않는 것*

---
#### 1.2. 운반 수단 (Transport)
|구분|Cookie|Header|
|---|---|---|
|전송 방식|브라우저 자동 포함|클라이언트가 직접 설정|
|CSRF 위험|존재|없음|
|XSS 위험|HttpOnly로 방어 가능|JS 접근 필수 → 위험|
|주 사용처|웹 브라우저|모바일 앱, 서버-서버|

> 👉 **Session / Token ≠ Cookie / Header**  
> (상태 관리 전략과 전송 경로는 독립적으로 조합됨)
---
### 2. 동작 메커니즘 
#### 2.1. 세션 기반 인증 (Stateful)
> 서버가 "누가 로그인했는지"를 기억하고 있는 방식

**동작 흐름**
1. 로그인 요청 (ID/PW)
2. 서버가 세션 생성 (Redis/DB)
3. Session ID 발급
4. `Set-Cookie: SESSION_ID=abc...`
5. 이후 요청마다 Cookie 자동 전송
6. 서버는 세션 저장소 조회 후 인증

**특징**
- 서버가 단일 진실 원천 (Single Source of Truth)
- 로그아웃, 권한 변경 즉시 반영 가능
- Redis 장애 = 인증 전체 장애

#### 2.2. 토근 기반 인증 (Stateless)
> 서버는 기억하지 않고, 클라이언트의 JWT만 확인하는 방식
**동작 흐름**
1. 로그인 요청
2. 서버가 JWT 발급 (서명)
3. 서버는 토큰 저장하지 않음
4. 클라이언트가 토큰 보관
5. 요청 시 `Authorization: Bearer <JWT>`
6. 서버는 Signature만 검증

**특징**
- 서버 무상태 → 확장성 매우 우수
- 토큰 탈취 시 만료 전까지 무력화 불가
- Refresh Token 전략이 사실상 필수
  
---
### 3. Cookie vs Header
#### 3.1 Cookie
> 쿠키는 단순히 `저장소`일 뿐이다. 
```http
Set-Cookie: refresh_token=xxx;
  HttpOnly;
  Secure;
  SameSite=Lax;
```
- 브라우저가 자동 전송
- CSRF 공격 벡터 존재
- HttpOnly로 XSS 방어 가능
---
#### 3.2 Authorization Header
> 개발자가 직접 인증 정보를 HTTP 요청 메시지의 Header 영역에 삽입
```http
Authorization: Bearer eyJhbGciOi...
```
- CSRF 원천 차단
- JS 접근 필수 → XSS에 취약
- 모바일/서버 통신에 최적
---
### 4. 조합
| 토큰 종류 | 저장 위치 | 이유 |
| -- | -- | -- |
| Access Token  | JS 메모리 | 짧은 수명, 노출 최소화 |
| Refresh Token | HttpOnly Cookie | XSS 방어, 자동 전송 |

> ❗ LocalStorage에 JWT 저장은 거의 선택되지 않음
> 
> → XSS 한 번에 전체 계정 탈취 가능
---
### 5. Cookie 보안 옵션
| 옵션 | 역할 | 방어 대상 |
| -------- | ----------- | ----- |
| HttpOnly | JS 접근 차단    | XSS   |
| Secure   | HTTPS 강제    | MITM  |
| SameSite | 타 사이트 전송 제한 | CSRF  |

**SameSite 전략**
- `Strict`: 보안 최강, UX 제한 큼
- `Lax`: 대부분의 웹 서비스 권장
- `None`: 반드시 Secure 필요 (OAuth, 외부 연동)
---
### 6. 공격 모델
| 공격 유형 | 정의 | 공격 조건 | 주 공격 대상 | 대표 예시 | 주요 방어 전략 |
| -- | -- | -- | -- | --| -- |
| CSRF<br>(Cross-Site Request Forgery) | 사용자가 로그인된 상태를 악용하여 의도치 않은 요청을 서버로 전송하게 만드는 공격 | Cookie 자동 전송 + 서버의 요청 출처 검증 부재 | Session ID, Refresh Token (Cookie)| 공격자가 만든 페이지 접속 시 송금/비밀번호 변경 요청 전송 | SameSite Cookie, CSRF Token, Origin/Referer 검증 |
| XSS<br>(Cross-Site Scripting)| 악성 JavaScript를 삽입해 사용자의 브라우저에서 실행시키는 공격       | 입력값 검증 미흡, CSP 부재| LocalStorage JWT, JS 변수의 Access Token | `localStorage.getItem("jwt")` 탈취  | HttpOnly Cookie, CSP, 입력값 인코딩/검증 |
| Token Theft| 네트워크/로그/프론트엔드 취약점을 통한 토큰 탈취 | 토큰 평문 노출 | Access/Refresh Token | 콘솔 로그에 토큰 출력 | HTTPS, 토큰 최소 노출|
| Replay Attack | 탈취한 요청을 그대로 재전송 | 요청 무결성 검증 부재| Bearer Token| 이전 요청 재사용  | 짧은 만료 시간, nonce|
| Session Fixation  | 공격자가 미리 정한 세션 ID를 사용하게 만드는 공격 | 로그인 전 세션 유지| Session ID | 로그인 전 발급 세션 재사용  | 로그인 시 세션 재발급 |
> 🔑 **핵심**
> - CSRF는 브라우저의 자동성을 노리고,
> - XSS는 JS 실행 권한을 노린다.
> → 방어 전략이 절대 겹치지 않는다.
---
### 7. 쿠키에는 실제로 무엇을 저장하는가?
| 분류 | 예시 | 설명 |
| -- | -- | -- |
| 인증 | Session ID, Refresh Token | 사용자 식별 키 |
| 상태 | 언어, 테마, 다크모드              | UX 유지    |
| 추적 | User ID, Campaign ID      | 행동 분석    |
| 광고 | Tracking ID               | 광고 타겟팅   |
> 💡 쿠키에는 “데이터”가 아니라 “참조용 ID”만 저장하는 것이 원칙
---
### 8. 웹 서버 권장 아키텍처
| 항목    | 전통적 Session | 하이브리드 JWT               |
| ----- | ----------- | ----------------------- |
| 인증 매체 | Session ID  | Access + Refresh Token  |
| 운반 수단 | Cookie      | Memory + Cookie         |
| 상태 관리 | Stateful    | AT Stateless / RT 최소 상태 |
| 확장성   | 낮음          | 높음                      |
| 권장 상황 | 내부 시스템, 관리자 | 일반 웹 + 모바일              |
---
### 9. Summary
- 단순성 및 즉시 제어 -> Session
- 확장성 및 분산 제어 -> Token
- 보안은 **상태 + 전송 + 만료 전략**의 합
---