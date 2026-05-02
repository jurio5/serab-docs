# 닉네임 변경 API

PATCH /members/me

로그인된 사용자의 닉네임을 변경하는 API 입니다.

---

## [설명]

로그인된 사용자의 닉네임을 변경합니다.

닉네임 변경 시 다음 검증을 수행합니다.

- 기본 유효성 검증 (길이, 형식)
- 현재 닉네임과 동일한 경우 변경 차단
- 다른 사용자가 사용 중인 닉네임 변경 차단

---

## [Request]

```json
Headers:
Cookie: ACCESS_TOKEN=...

Body:
{
  "nickname": "새닉네임"
}
```
---

## [닉네임 유효성 규칙]

- 길이: 2자 이상 20자 이하
- 허용 문자: 영문, 숫자, 한글
- 공백: 앞/뒤 공백 자동 제거, 중간 공백 불가
- 예약어: 현재 정책에선 비활성화 상태 (추후 적용 가능)

---

## [비즈니스 정책]

| 조건 | 결과 |
|------|------|
| 현재 닉네임과 동일 | NICKNAME_SAME_AS_CURRENT |
| 다른 사용자가 사용 중 | NICKNAME_ALREADY_EXISTS |
| 기본 유효성 실패 | NICKNAME_INVALID 계열 |


---

## [성공 응답]

```json
HTTP 200
{
"success": true
}
```


---

## [실패 응답]

가능한 ErrorCode:

- NICKNAME_SAME_AS_CURRENT
- NICKNAME_ALREADY_EXISTS
- NICKNAME_INVALID
- NICKNAME_INVALID_LENGTH
- NICKNAME_INVALID_FORMAT
- NICKNAME_RESERVED
- UNAUTHORIZED

예시:

```json
{
  "success": false,
  "errorCode": "NICKNAME_ALREADY_EXISTS",
  "message": "이미 존재하는 닉네임입니다."
}
```
---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. NICKNAME VO 생성 → 길이/형식/공백 검증
3. 현재 닉네임과 동일한지 확인
4. DB에서 동일 닉네임 존재 여부 확인
5. 이상 없으면 변경 로직 수행
6. 성공 응답 반환
