# JWT Refresh Token Rotation

본 문서는 **Access Token / Refresh Token 구조**와  
**Refresh Token Rotation 정책**을 다룹니다.

---

## 1. 토큰 종류

### Access Token
- 만료 시간: 1시간
- Stateless 방식 인증
- 만료 전까지는 그대로 사용
- Blacklist 등록 시 즉시 무효화

### Refresh Token
- 만료 시간: 1일 (Rotation으로 사용 시 계속 갱신)
- Redis에 영속 저장 (TTL 자동 만료)
- Access Token 재발급에 사용
- Rotation 정책 적용
  → 요청 시마다 새로운 Refresh Token 발급  
  → 이전 Refresh Token은 30초 유예 후 폐기

---

## 2. Claims 구조

```json
{
  "id": 1,
  "email": "user@example.com",
  "nickname": "사용자",
  "role": "ROLE_USER",
  "type": "access" // or "refresh"
}
```

---

## 3. Refresh Token Rotation Flow

### 기본 흐름
1. Access Token 만료
2. Refresh Token 쿠키 확인
3. Refresh Token JWT 파싱
4. Redis 저장 값과 비교
5. 같으면 → **Access + Refresh Token 재발급**
6. Redis: 새 토큰 저장 + 이전 토큰 30초 유예기간 설정
7. 클라이언트 쿠키 업데이트 (HttpOnly + Secure)

### 동시 요청 처리 (락 기반)

```
[요청 A] refreshToken=old → 락 획득 → 새 토큰 발급
[요청 B] refreshToken=old → 락 대기/실패 → 현재 토큰 반환
```

```java
// 토큰 로테이션 시 락 획득
if (!refreshTokenService.tryAcquireLock(memberId)) {
    // 다른 요청이 로테이션 중 → 현재 토큰 반환
    String currentRefreshToken = refreshTokenService.getCurrentRefreshToken(memberId);
    return new TokenPair(newAccessToken, currentRefreshToken);
}
```

### 이전 토큰 유예기간 (30초)

```
[0초] Rotation 발생 → newToken 발급, oldToken 30초 유예
[5초] oldToken으로 요청 → PREVIOUS 상태 → 새 access 발급 (refresh 미발급)
[30초] oldToken 만료 → 더 이상 사용 불가
```

```java
// 토큰 상태 확인
TokenValidationResult result = refreshTokenService.validate(memberId, token);

switch (result) {
    case VALID -> { /* 정상 토큰 */ }
    case PREVIOUS -> { /* 이전 토큰 - access만 재발급 */ }
    case INVALID -> { /* 무효 토큰 - 재인증 필요 */ }
}
```

---

## 4. Redis 저장 구조

```
REFRESH:{memberId}         = refresh_token (TTL = 1일)
PREV_REFRESH:{memberId}    = previous_token (TTL = 30초)
LOCK:refresh:{memberId}    = "locked" (TTL = 5초)
BL:{accessToken}           = "logout" (TTL = 1시간)
```

| 키 | 용도 |
|----|------|
| `REFRESH:` | 현재 유효한 Refresh Token |
| `PREV_REFRESH:` | 이전 토큰 (30초 유예기간) |
| `LOCK:refresh:` | 토큰 로테이션 동시성 제어 |
| `BL:` | Blacklist (무효 Access Token) |

---

## 5. 로그아웃 처리

- Refresh Token 삭제 (Redis)
- Previous Token 삭제 (Redis)
- Access Token → Blacklist 등록
- 모든 인증 쿠키 삭제
    - `access_token`
    - `refresh_token`
    - `role`

---

## 6. 보안 효과

| 위협 | 방어 |
|------|------|
| Refresh Token 탈취 | Rotation으로 즉시 무효화 |
| Access Token 도난 | Blacklist로 재사용 차단 |
| 동시 요청 공격 | 락으로 중복 발급 방지 |
| 네트워크 지연 | 30초 유예기간으로 정상 요청 보호 |

---

## 7. 관련 클래스

| 클래스 | 역할 |
|--------|------|
| `JwtService` | 토큰 재발급 로직 |
| `RefreshTokenService` | Redis 토큰 관리 (저장/검증/락) |
| `JwtTokenProvider` | JWT 생성/파싱 |
| `JwtAuthFilter` | 요청별 인증 처리 |
