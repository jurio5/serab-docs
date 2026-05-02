# WebSocket 티켓 기반 인증

WebSocket(STOMP) 연결 시 사용되는 일회용 티켓 인증 방식입니다.

---

## 배경

WebSocket 연결에서는 HTTP 쿠키 기반 JWT 인증을 직접 사용할 수 없습니다.
→ 일회용 티켓을 발급받아 STOMP CONNECT 시 전달하는 방식으로 해결

---

## 인증 흐름

```
[1] 클라이언트: POST /auth/ws-ticket (JWT 쿠키 자동 전송)
        ↓
[2] JwtAuthFilter: JWT 검증 → SecurityContext에 memberId 저장
        ↓
[3] WebSocketTicketController: requestContext.getActor().getId()
        ↓
[4] WebSocketTicketService.issueTicket():
    - UUID 티켓 생성
    - Redis 저장: ws:ticket:{UUID} → memberId:nickname (TTL 30초)
    - 티켓 반환
        ↓
[5] 클라이언트: STOMP CONNECT + ticket 헤더
        ↓
[6] TicketAuthInterceptor (ChannelInterceptor):
    - ticketService.validateAndConsume(ticket)
    - Redis에서 memberId:nickname 추출 + 티켓 삭제 (일회용)
    - accessor.setUser(new MemberPrincipal(memberId, nickname))
        ↓
[7] WebSocket 세션: Principal에 memberId + nickname 저장
        ↓
[... 세션 유지 ...]
        ↓
[8] DISCONNECT: Principal.getName() → memberId로 퇴장 처리
```

---

## Redis 저장 구조

```
ws:ticket:{UUID} = memberId:nickname (TTL 30초)
```

- **일회성**: validateAndConsume 시 즉시 삭제 (getAndDelete)
- **TTL**: 30초 내 미사용 시 자동 만료

---

## 주요 컴포넌트

| 컴포넌트 | 역할 |
|---------|------|
| `WebSocketTicketController` | POST /auth/ws-ticket 엔드포인트 |
| `WebSocketTicketService` | 티켓 발급/검증 (Redis 연동) |
| `TicketData` | 티켓 검증 결과 (memberId + nickname) |
| `TicketAuthInterceptor` | STOMP CONNECT 시 티켓 검증 |
| `MemberPrincipal` | WebSocket 세션의 Principal 구현체 (memberId + nickname) |

---

## 프론트엔드 처리

```typescript
// stompClient.ts
async function createStompClient() {
    const ticket = await fetchWsTicket();  // 티켓 발급
    
    return new Client({
        connectHeaders: { ticket },  // CONNECT 시 전달
        beforeConnect: async () => {
            // 재연결 시 새 티켓 발급
            const newTicket = await fetchWsTicket();
            client.connectHeaders = { ticket: newTicket };
        }
    });
}
```

---

## 보안 효과

| 항목 | 설명 |
|------|------|
| 서버 검증 | memberId는 서버에서만 추출 (위조 불가) |
| 일회용 | 사용 즉시 삭제 (재사용 차단) |
| TTL | 30초 만료 (탈취 시간 최소화) |
| JWT 연계 | 티켓 발급 시 JWT 인증 필수 |
