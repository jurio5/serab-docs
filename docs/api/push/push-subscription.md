# Push Subscription API

## 구독 등록

FCM 토큰을 서버에 등록하여 푸시 알림을 받을 수 있도록 한다.

- **URL:** `POST /push/subscribe`
- **인증:** 필요 (로그인 유저만)

### Request Body
```json
{
  "fcmToken": "dXh1bGFy...긴 FCM 토큰",
  "deviceType": "WEB"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|:--:|------|
| `fcmToken` | String | O  | Firebase에서 발급받은 토큰 |
| `deviceType` | String | O  | `WEB` 또는 `PWA` |

### Response
```json
{
  "code": "OK",
  "msg": "성공",
  "data": null
}
```

> 이미 등록된 토큰이면 중복 저장하지 않고 정상 응답 반환

---

## 구독 해제

등록된 FCM 토큰을 제거하여 푸시 알림 수신을 중단한다.

- **URL:** `DELETE /push/unsubscribe`
- **인증:** 필요 (로그인 유저만)

### Request Body
```json
{
  "fcmToken": "dXh1bGFy...해제할 FCM 토큰"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|:--:|------|
| `fcmToken` | String | O  | 해제할 FCM 토큰 |

### Response
```json
{
  "code": "OK",
  "msg": "성공",
  "data": null
}
```

---

## 클라이언트 동작

### Android PWA 권한

- 설치형 PWA라도 실제 웹 푸시는 브라우저의 사이트 알림 권한을 함께 사용합니다.
- Android에서는 앱 정보의 알림 허용과 Chrome 사이트 알림 허용이 모두 필요할 수 있습니다.
- `Notification.permission = denied` 상태가 되면 앱 안에서 시스템 권한 팝업을 다시 열 수 없고,
  브라우저의 사이트 설정에서 직접 허용해야 합니다.

### 길드전 채팅 딥링크

- room 채팅 푸시는 `/rooms/{roomId}` 링크를 포함할 수 있습니다.
- 알림 클릭 시 서비스워커가 기존 창을 해당 room URL로 이동시키거나,
  열린 창이 없으면 새 창으로 엽니다.
