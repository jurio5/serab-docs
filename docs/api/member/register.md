# OAuth2 기반 회원가입(Register) + 자동 로그인 처리

OAuth2 로그인 이후,  
아직 사이트 회원이 아닌 사용자가 추가 정보를 입력하여 회원가입을 완료하고,  
서버가 자동 로그인 처리를 수행하여 최종적으로 JWT 쿠키를 발급하는 전체 흐름 설명입니다.

---

## 1. 회원가입(Register)의 전체 개요

OAuth2 로그인 성공 후 서버는 사용자가 기존 회원인지 여부를 판단

### 1) 기존 회원(Member Exists)
- DB에 Member + OAuth 매핑 존재
- 즉시 JWT 발급 → 로그인 완료 처리
- 클라이언트로 바로 이동 (`status=SUCCESS`)

### 2) 신규 회원(Member Not Exists)
- 아직 DB에 Member 없음
- provider / oauthId / email 정보를 서버가 HttpOnly 쿠키에 저장
- 프론트엔드 회원가입 페이지로 이동 (`status=REGISTER`)
- 프론트는 사용자에게 NICKNAME(nickname) 입력만 요구
- 이후 `POST /auth/register` 호출하여 회원가입 진행

---

## 2. 회원가입(Register) 단계에서 사용되는 임시 OAuth 쿠키

회원가입 페이지에 오기 전에 서버에서 미리 OAuth 데이터를 저장

임시 쿠키 목록(모두 HttpOnly + Secure):

- `oauth_provider` : OAuthProvider 문자열 ("KAKAO", "GOOGLE")
- `oauth_uid` : Provider가 제공하는 고유 oauthId
- `oauth_email` : Provider 계정 이메일

**특징**
- 사용자에게 노출되지 않음 (HttpOnly)
- 회원가입 시에만 사용
- 회원가입 완료 후 즉시 삭제됨

---

## 3. 회원가입 요청(`/auth/register`) 흐름

클라이언트 예시 요청:

```
POST /auth/register
Content-Type: application/json

{
  "nickname": "홍길동"
}
```

서버 처리 과정:

### 1) RegisterService.register()
1. 임시 OAuth 쿠키 읽기
    - provider
    - oauthId
    - email

2. 검증
    - 이메일 중복 검사
    - provider + oauthId 중복 검사

3. Member + OAuth 엔티티 생성 후 저장

4. 저장된 Member 객체 반환

### 2) AuthTokenService.login()
회원가입 완료된 Member를 이용해 즉시 로그인 처리:

- SecurityUser 생성
- AccessToken, RefreshToken 발급
- RefreshToken → Redis 저장
- JWT 쿠키 저장
- role 쿠키 저장

### 3) AuthTokenService.removeOAuthTempCookies()
- 임시 OAuth 쿠키 3개 삭제 (`oauth_provider`, `oauth_uid`, `oauth_email`)

### 4) 클라이언트로 성공 응답 반환
- 회원가입 + 자동 로그인 완료됨

---

## 4. 전체 흐름 다이어그램

```
[사용자] 
   ↓ OAuth 로그인
[OAuth Provider 인증]
   ↓ redirect
[서버 SuccessHandler]
   ├─ 기존 회원 → JWT 발급 → 바로 로그인 완료
   └─ 신규 회원 → 임시 OAuth 쿠키 저장 → 프론트 회원가입 페이지 이동

[프론트 회원가입 페이지]
   ↓ nickname 입력
POST /auth/register
   ↓
RegisterService.register()
   ↓ Member 생성
AuthTokenService.login()
   ↓ JWT 쿠키 발급
removeOAuthTempCookies()
   ↓
[회원가입 + 자동 로그인 완료]
```

---

## 5. 주요 코드 구조

### RegisterService

```java
public Member register(RegisterRequest request, HttpServletRequest req) {
    String provider = CookieUtil.getCookie(req, "oauth_provider")
            .map(Cookie::getValue)
            .orElseThrow(OAUTH_COOKIE_PROVIDER_MISSING::toException);

    String oauthId = CookieUtil.getCookie(req, "oauth_uid")
            .map(Cookie::getValue)
            .orElseThrow(OAUTH_COOKIE_OAUTHID_MISSING::toException);

    String email = CookieUtil.getCookie(req, "oauth_email")
            .map(Cookie::getValue)
            .orElseThrow(OAUTH_COOKIE_EMAIL_MISSING::toException);

    if (memberRepository.existsByEmail(email)) throw EMAIL_ALREADY_EXISTS.toException();
    if (memberRepository.existsByProviderAndOauthId(from(provider), oauthId)) throw OAUTH_ALREADY_EXISTS.toException();

    Member member = Member.create(request.nickname(), email, MEMBER);
    OAuth.create(member, from(provider), oauthId);

    return memberRepository.save(member);
}
```

---

### AuthTokenService

```java
public void login(Member member, HttpServletResponse response) {
    SecurityUser user = SecurityUser.createAuthUser(
            member.getId(),
            member.getNickname(),
            member.getEmail(),
            MEMBER.getAuthority()
    );

    String access = jwtTokenProvider.generateToken(jwtClaimsMapper.from(user, ACCESS));
    String refresh = jwtTokenProvider.generateToken(jwtClaimsMapper.from(user, REFRESH));

    refreshTokenService.saveRefreshToken(member.getId(), refresh);

    CookieUtil.addCookie(response, ACCESS_TOKEN_COOKIE, access, (int) jwtConfig.accessTokenExpiration(), true, true);
    CookieUtil.addCookie(response, REFRESH_TOKEN_COOKIE, refresh, (int) jwtConfig.refreshTokenExpiration(), true, true);

    addRoleCookie(response, member);
}

public void removeOAuthTempCookies(HttpServletResponse response) {
    CookieUtil.deleteCookie(response, "oauth_provider");
    CookieUtil.deleteCookie(response, "oauth_uid");
    CookieUtil.deleteCookie(response, "oauth_email");
}
```

---

## 6. 컨트롤러 흐름

```java
@PostMapping("/register")
public ResponseData<?> register(@RequestBody RegisterRequest request,
                                HttpServletRequest req,
                                HttpServletResponse response) {

    Member member = registerService.register(request, req);

    authTokenService.login(member, response);
    authTokenService.removeOAuthTempCookies(response);

    return ResponseData.success();
}
```

---

## 7. 회원가입 후 결과

- JWT 자동 발급 (access / refresh)
- Refresh Token은 Redis 보관
- role 쿠키 저장
- 임시 OAuth 쿠키 삭제
- 즉시 로그인된 상태로 클라이언트 진행

---

## 8. 보안 고려 사항

- 임시 OAuth 쿠키는 HttpOnly + Secure → 사용자가 볼 수 없음
- 회원가입 완료 후 즉시 삭제
- Refresh Token Rotation 적용
- 모든 JWT는 쿠키 기반이므로 XSS/CSRF 대비 필수 옵션 적용
- 서버에서는 세션을 사용하지 않음 (stateless)

---

