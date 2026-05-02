# 쿠폰 적용

입력된 `PID`와 쿠폰 코드를 이용해 외부 쿠폰 서버에 적용을 시도합니다.

- **URL:** `POST /coupons/redeem`
- **인증:** 불필요

---

## Request Body

```json
{
  "pid": "F0C235149434B95BBC93938C6636D5B...",
  "couponCode": "SKRE300BIRTH"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|:--:|------|
| `pid` | String | O | 게임 내 회원번호(PID) |
| `couponCode` | String | O | 적용할 쿠폰 코드 |

---

## Success Response

```json
{
  "code": "OK",
  "msg": "성공",
  "data": {
    "couponCode": "SKRE300BIRTH",
    "status": "SUCCESS",
    "message": "쿠폰이 등록되었습니다.",
    "rewards": [
      {
        "name": "루비",
        "quantity": 300
      }
    ]
  }
}
```

### `status` Values

| 값 | 설명 |
|----|------|
| `SUCCESS` | 적용 성공 |
| `ALREADY_USED` | 이미 사용했거나 만료된 쿠폰 |
| `FAILED` | 적용 실패 |

### Reward Fields

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | String | 보상 이름 |
| `quantity` | Integer | 보상 수량 |

---

## Error Responses

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| `COUPON_PID_REQUIRED` | 400 | PID 누락 |
| `INTERNAL_SERVER_ERROR` | 500 | 외부 쿠폰 서버 호출 실패 또는 응답 파싱 실패 |

---

## Notes

- 이 API는 외부 쿠폰 서버를 호출하는 프록시 역할을 합니다.
- `PID`는 요청 처리용으로만 사용하고 서버에 저장하지 않고 로컬 스토리지에 저장합니다.
- `rewardSummary`는 관리자 입력 메타데이터이고, 실제 적용 결과 보상은 이 API 응답의 `rewards`를 기준으로 봅니다.
