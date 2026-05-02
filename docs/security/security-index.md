# Security 문서 목차

본 문서는  
프로젝트의 인증 / 보안 관련 설계를 한 곳에서 관리하기 위한 목차입니다.

JWT, OAuth, 필터 체인, 로그아웃 처리 등  
보안과 관련된 모든 문서를 아래에서 확인할 수 있습니다.

---

## [인증 흐름 문서]

- [OAuth 로그인 흐름](./auth/oauth-login-flow.md)
- [JWT 인증 필터 및 예외 처리 흐름](./auth/jwt-auth-filter.md)
- [Refresh Token Rotation 플로우](./auth/jwt-refresh-rotation.md)
- [로그아웃 처리 흐름](./auth/logout-flow.md)
- [Security Filter Chain 구조](./auth/filter-chain.md)
- [Admin Gate Filter 구조](./admin/admin-gate-filter.md)
- [Suspension Gate Filter 구조](./suspension-gate-filter.md)

---

## [보안 설계 요약]

본 프로젝트의 인증 / 보안 설계 특징:

- OAuth2 기반 소셜 로그인 (Kakao, Google)
- Access Token / Refresh Token 기반 인증
- HttpOnly + Secure Cookie 방식 토큰 전달
- Refresh Token Rotation 적용
- Redis 기반 Refresh Token 관리
- Access Token Blacklist 기반 즉시 로그아웃 처리
- Stateless 인증 구조
- JWT 기반 사용자 인증

---

## [문서 구조]

docs/security/
- security-index.md

docs/security/auth/
- oauth-login-flow.md
- jwt-auth-filter.md
- jwt-refresh-rotation.md
- logout-flow.md
- filter-chain.md


---

Security 관련 문서가 추가될 경우  
해당 문서를 작성한 뒤 이 목차에 링크를 추가하여 관리합니다.
