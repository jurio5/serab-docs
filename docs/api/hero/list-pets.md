# 펫 목록 조회 API

GET /pets

펫 목록을 조회하거나 검색하는 API입니다.

---

## [설명]

모든 펫 목록을 조회하거나, 이름으로 검색합니다.\
query 파라미터가 없으면 전체 펫 목록을 반환합니다.\
query 파라미터가 있으면 해당 문자열로 시작하는 펫만 반환합니다.

> **인증 불필요**: 비로그인 사용자도 조회 가능합니다.

---

## [Request]

```
Headers: (선택)
Cookie: ACCESS_TOKEN=...

Query Parameters:
- query: 검색어 (선택, 이름 prefix 검색)
```

### 예시

```
GET /pets              → 전체 펫 목록
GET /pets?query=유     → "유"로 시작하는 펫
```

---

## [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": [
    {
      "id": 1,
      "name": "유",
      "imageUrl": null
    },
    {
      "id": 2,
      "name": "연지",
      "imageUrl": null
    }
  ]
}
```

---

## [실패 응답]

이 API는 인증이 필요 없으므로 일반적인 실패 케이스가 없습니다.

---

## [내부 처리 흐름]

1. 클라이언트 요청 수신
2. query 파라미터 확인
   - null/blank → 전체 펫 조회
   - 값 존재 → 해당 prefix로 시작하는 펫 검색
3. PetResponse 리스트로 변환
4. 성공 응답 반환
