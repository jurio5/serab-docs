# 활성 쿠폰 목록 조회

현재 활성화된 공용 쿠폰 목록을 조회합니다.

- **URL:** `GET /coupons`
- **인증:** 불필요

---

## Response

```json
{
  "code": "OK",
  "msg": "성공",
  "data": [
    {
      "id": 1,
      "code": "SKRE300BIRTH",
      "name": "300일 기념 쿠폰",
      "rewardSummary": "루비 300개, 소환권 10장",
      "active": true,
      "createdAt": "2026-03-30T10:00:00",
      "updatedAt": "2026-03-30T10:00:00"
    }
  ]
}
```

### Response Fields

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | Long | 쿠폰 ID |
| `code` | String | 실제 쿠폰 코드 |
| `name` | String | 사용자 화면에 보여줄 쿠폰 제목 |
| `rewardSummary` | String | 관리자가 입력한 보상 요약 |
| `active` | boolean | 활성 여부 |
| `createdAt` | LocalDateTime | 생성 시각 |
| `updatedAt` | LocalDateTime | 수정 시각 |

---

## Notes

- 비활성 쿠폰은 조회되지 않습니다.
- 프론트에서는 이 목록과 사용자가 직접 추가한 로컬 쿠폰 목록을 합쳐서 사용합니다.
