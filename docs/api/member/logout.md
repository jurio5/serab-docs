# Logout(로그아웃) 처리 문서

JWT 기반 인증 환경에서 로그아웃 시 수행되는  
쿠키 삭제, Refresh Token 삭제, Access Token 블랙리스트 등록 등  
전체 흐름을 설명합니다.


# 1. 로그아웃 처리 개요

JWT 기반 환경에서는 서버가 세션을 저장하지 않으므로  
"로그아웃"은 아래와 같은 작업으로 구성됩니다:

1. AccessToken 삭제
2. RefreshToken 삭제 및 무효화
3. AccessToken을 블랙리스트에 등록 (만료 전까지 차단)
4. role 쿠키 삭제
5. 응답 성공 반환

즉, "로그아웃"은 토큰 기반 인증 정보를 제거하는 과정입니다.

# 2. 로그아웃 요청 흐름

### 클라이언트 요청
```
POST /auth/logout
```

### 서버 처리 흐름

1. 요청 쿠키에서 access_token / refresh_token 추출
2. 응답 쿠키에서 두 토큰 삭제
3. AccessToken 블랙리스트 등록
4. RefreshToken은 Redis에서 제거
5. role 쿠키 삭제
6. 클라이언트에 성공 응답 반환

# 3. 전체 흐름 다이어그램

```
[사용자]
   ↓ logout 요청
POST /auth/logout
   ↓
LogoutService.logout()
   ├─ 요청 쿠키에서 access & refresh 토큰 읽기
   ├─ access_token 쿠키 삭제
   ├─ refresh_token 쿠키 삭제
   ├─ role 쿠키 삭제
   ├─ AccessToken → 블랙리스트 등록
   ├─ RefreshToken → Redis 저장소에서 제거
   ↓
[로그아웃 완료]
```

# 4. 핵심 LogoutService 코드

```java
@Service
@RequiredArgsConstructor
public class LogoutService {

    private final AccessTokenBlacklistService blacklistService;
    private final RefreshTokenService refreshTokenService;
    private final JwtTokenProvider jwtTokenProvider;

    public void logout(HttpServletRequest request, HttpServletResponse response) {

        String accessToken = CookieUtil.getCookie(request, ACCESS_TOKEN_COOKIE)
                .map(Cookie::getValue)
                .orElse(null);

        String refreshToken = CookieUtil.getCookie(request, REFRESH_TOKEN_COOKIE)
                .map(Cookie::getValue)
                .orElse(null);

        // 1. 쿠키 삭제
        CookieUtil.deleteCookie(response, ACCESS_TOKEN_COOKIE);
        CookieUtil.deleteCookie(response, REFRESH_TOKEN_COOKIE);
        CookieUtil.deleteCookie(response, "role");

        // 2. AccessToken 블랙리스트 등록
        if (StringUtils.hasText(accessToken)) {
            blacklistService.blacklist(accessToken);
        }

        // 3. RefreshToken 제거
        if (StringUtils.hasText(refreshToken)) {
            Claims claims = jwtTokenProvider.getClaims(refreshToken);
            long memberId = Long.parseLong(claims.getSubject());
            refreshTokenService.deleteRefreshToken(memberId);
        }
    }
}
```

# 5. 클라이언트 → 서버 Logout API

```java
@PostMapping("/logout")
public ResponseData<?> logout(HttpServletRequest request,
                              HttpServletResponse response) {
    logoutService.logout(request, response);
    return ResponseData.success();
}
```

# 6. 보안 고려 사항

- AccessToken은 만료될 때까지 유효하므로  
  반드시 **블랙리스트 등록**해야 완전 차단됨
- RefreshToken은 Redis 등 저장소에서 확실히 삭제해야 함
- 쿠키는 `HttpOnly + Secure + SameSite=Lax` 옵션 적용  
  → XSS, CSRF 방어
- 서버는 세션을 사용하지 않기 때문에  
  "로그아웃 = 토큰 무효화 + 쿠키 삭제" 로 정의됨

# 7. 로그아웃 결과

- access_token 쿠키 삭제
- refresh_token 쿠키 삭제
- role 쿠키 삭제
- AccessToken 즉시 무효화 (블랙리스트)
- RefreshToken 폐기 (Redis 제거)
- 사용자는 완전히 로그아웃된 상태가 됨
