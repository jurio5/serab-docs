# 관리자 쿠폰 관리 API

관리자가 공용 쿠폰을 조회, 등록, 수정, 삭제하는 API입니다.

- **Base URL:** `/admin/coupons`
- **인증:** 필요 (`ROLE_ADMIN`)

---

## 1. 쿠폰 목록 조회

- **URL:** `GET /admin/coupons`

### Query Parameters

| 이름 | 타입 | 필수 | 설명 |
|------|------|:--:|------|
| `keyword` | String | X | `code` 또는 `name` 검색어 |
| `page` | int | X | 페이지 번호, 기본값 `0` |
| `size` | int | X | 페이지 크기, 기본값 `20` |

### Response

```json
{
  "code": "OK",
  "msg": "성공",
  "data": {
    "content": [
      {
        "id": 1,
        "code": "SKRE300BIRTH",
        "name": "300일 기념 쿠폰",
        "rewardSummary": "루비 300개, 소환권 10장",
        "active": true,
        "createdAt": "2026-03-30T10:00:00",
        "updatedAt": "2026-03-30T10:00:00"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  }
}
```

---

## 2. 쿠폰 등록

- **URL:** `POST /admin/coupons`

### Request Body

```json
{
  "code": "SKRE300BIRTH",
  "name": "300일 기념 쿠폰",
  "rewardSummary": "루비 300개, 소환권 10장",
  "active": true
}
```

---

## 3. 쿠폰 수정

- **URL:** `PUT /admin/coupons/{id}`

### Path Parameter

| 이름 | 타입 | 설명 |
|------|------|------|
| `id` | Long | 수정할 쿠폰 ID |

### Request Body

등록과 동일한 구조를 사용합니다.

---

## 4. 쿠폰 삭제

- **URL:** `DELETE /admin/coupons/{id}`

### Path Parameter

| 이름 | 타입 | 설명 |
|------|------|------|
| `id` | Long | 삭제할 쿠폰 ID |

---

## Error Responses

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| `COUPON_NOT_FOUND` | 404 | 존재하지 않는 쿠폰 |
| `COUPON_ALREADY_EXISTS` | 400 | 중복된 쿠폰 코드 |
| `COUPON_CODE_REQUIRED` | 400 | 쿠폰 코드 누락 |
| `FORBIDDEN_ACCESS` | 403 | 관리자 권한 없음 |

---

## Notes

- `code`는 대문자로 정규화되어 저장됩니다.
- `name`이 비어 있으면 `code`를 기본 표시명으로 사용합니다.
- `rewardSummary`는 실제 외부 응답이 아니라 관리용 보상 요약 메타데이터입니다.
