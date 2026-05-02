# ShedLock / 재연결 복구 메모

블루그린 배포를 고려한 스케줄러 락과 WebSocket 재연결 복구 방향을 정리한 문서입니다.

---

## 문서 개요

- 스케줄러 중복 실행 방지 목적
- Redis 기반 ShedLock 적용 이유
- WebSocket 재연결 후 상태 재조회 전략

---

## 왜 RabbitMQ를 바로 도입하지 않았는가

- 현재 프로젝트는 Spring MVC + STOMP + CSR 구조입니다.
- 방 상태를 다시 가져올 수 있는 조회 API가 이미 준비되어 있습니다.
- 방 상세, 채팅 히스토리, 전투 기록을 재조회하면 끊긴 동안 놓친 실시간 이벤트가 있어도 최종 상태를 다시 맞출 수 있습니다.
- 운영 복잡도를 바로 올리기보다, 현재 규모에 맞는 복구 전략을 먼저 완성하는 쪽이 더 적절하다고 판단했습니다.

관련 조회 API:

- `GET /rooms/{roomId}/detail`
- `GET /rooms/{roomId}/chats`
- `GET /battle-records/rooms/{roomId}`

---

## 재연결 복구 전략

1. 웹소켓 재연결 성공을 감지합니다.
2. 방 토픽을 다시 구독합니다.
3. 방 상세를 다시 조회합니다.
4. 채팅 히스토리를 다시 조회합니다.
5. 필요하면 전투 기록도 다시 조회합니다.

이 방식은 실시간 이벤트를 모두 재생하는 구조는 아니지만, 길드전 방의 최종 상태를 다시 맞추는 데는 충분합니다.

---

## ShedLock

### 왜 Redis provider를 선택했는가

- 이미 Redis를 운영 중입니다.
- 블루그린 목적은 스케줄러 중복 실행 방지이므로 별도 DB 락 테이블까지 만들 필요가 없습니다.
- 락 저장소를 Redis로 두면 추가 스키마 변경 없이 바로 분산 락을 걸 수 있습니다.

### 코드 적용 범위

- `src/main/java/sena/core/global/config/SchedulerLockConfig.java`
  - `@EnableSchedulerLock`
  - Redis 기반 `LockProvider` 등록

- `@SchedulerLock` 적용 스케줄러
  - `src/main/java/sena/core/content/room/scheduler/RoomScheduler.java`
  - `src/main/java/sena/core/content/stats/scheduler/StatsBatchScheduler.java`
  - `src/main/java/sena/core/content/member/scheduler/MemberPurgeScheduler.java`
  - `src/main/java/sena/core/content/push/scheduler/DailyGiftReminderScheduler.java`
  - `src/main/java/sena/core/content/room/scheduler/RoomPurgeScheduler.java`
  - `src/main/java/sena/core/content/room/scheduler/WebSocketRoomPresenceScheduler.java`
  - `src/main/java/sena/core/content/room/scheduler/WebSocketActiveMemberReconciliationScheduler.java`

### 락 시간 기준

- 일 단위 / 월 단위 배치
  - `lockAtMostFor` 는 넉넉하게 두고, `lockAtLeastFor` 는 짧게 설정합니다.
- 짧은 fixedDelay 스케줄러
  - 장애 시 락이 오래 남지 않게 `lockAtMostFor` 를 짧게 설정합니다.

---

## 참고 문서

- [ShedLock README](https://github.com/lukas-krecan/ShedLock)
