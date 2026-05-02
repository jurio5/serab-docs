# Suspension Gate Filter 구조

정지된 사용자의 쓰기 요청을 필터 레벨에서 차단하는 구조를 설명합니다.

---

## 1. 필터 체인에서의 위치

```
요청 →
JwtExceptionFilter →
JwtAuthFilter →
SuspensionGateFilter →        ← POST/PUT/PATCH/DELETE 요청에만 동작
AdminGateFilter →              ← /admin/** 요청에만 동작
UsernamePasswordAuthenticationFilter →
Spring Security 인증/인가 처리
```

JwtAuthFilter 이후에 실행되므로,  
이미 SecurityContextHolder에 인증 정보(SecurityUser)가 세팅된 상태입니다.

---

## 2. 동작 조건

| HTTP Method | 필터 동작 여부 | 설명 |
|-------------|---------|------|
| GET |  스킵 | 읽기 요청 허용 |
| HEAD |  스킵 | 읽기 요청 허용 |
| OPTIONS |  스킵 | CORS preflight 허용 |
| POST |  동작 | 쓰기 요청 차단 대상 |
| PUT |  동작 | 쓰기 요청 차단 대상 |
| PATCH |  동작 | 쓰기 요청 차단 대상 |
| DELETE |  동작 | 쓰기 요청 차단 대상 |

---

## 3. 차단 판단 흐름

```
1. SecurityContextHolder에서 Authentication 추출
2. Principal이 SecurityUser가 아님 → 필터 통과 (비인증 요청)
3. RequestContext.getActorOrNull()로 Member 조회
4. Member가 null → 필터 통과
5. Member.isSuspended() 확인
   └── suspendedUntil != null && suspendedUntil.isAfter(now)
6. 정지 상태 → 403 MEMBER_SUSPENDED 응답 반환
7. 정상 상태 → filterChain.doFilter() (요청 진행)
```

---

## 4. RequestContext 연동

SuspensionGateFilter는 `MemberRepository`를 직접 의존하지 않고,  
`RequestContext`에 위임하여 Member를 조회합니다.

```
SuspensionGateFilter → RequestContext.getActorOrNull() → Member 반환
                              ↓ (캐싱)
Service 계층 → RequestContext.getActor() → 캐시된 Member 재사용
```

- `RequestContext`는 `@RequestScope` 빈 (요청마다 새 인스턴스)
- 필터에서 조회한 Member가 같은 요청 내 서비스 계층에서 재사용됨
- DB 조회가 요청당 1회로 최적화

---

## 5. 즉시 반영 구조

```
관리자가 정지 API 호출  →  Member.suspend(until, reason)
                          →  더티 체킹 → 트랜잭션 커밋 시 flush → DB 반영
                                ↓
정지된 유저의 다음 요청  →  SuspensionGateFilter
                          →  RequestContext.getActorOrNull() → DB 조회 (최신 상태)
                          →  isSuspended() = true → 403 차단
```

---

## 6. 에러 응답 형식

```json
HTTP 403
{
  "code": "MEMBER_SUSPENDED",
  "message": "활동이 정지된 계정입니다.",
  "data": null
}
```

---

## 7. 설계 결정 사항

- **필터 레벨에서 차단**: 서비스마다 개별 체크 불필요, 새 API 추가 시 자동 적용
- **DB 조회 방식**: JWT claims 대신 DB를 조회하여 정지 즉시 반영 보장
- **읽기 허용**: 정지된 유저도 프로필, 통계, 댓글 목록 등 읽기 가능

---
