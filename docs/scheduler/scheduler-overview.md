# 스케줄러 문서 목차

배치 작업 및 정기 스케줄러 관련 문서입니다.

---

## 운영 관리 대상 스케줄러

| 스케줄러 | 위치 | 실행 주기 | 역할 |
|---------|------|---------|------|
| `MemberPurgeScheduler` | `global/scheduler/` | 매일 03:00 | 탈퇴 후 30일 경과 회원 영구 삭제 |
| `StatsBatchScheduler` | `stats/scheduler/` | 화/목/일 03:00 | PENDING 격파 기록 → 매치업 통계 집계 |
| `RoomScheduler` | `room/scheduler/` | 화/목/일 03:00 | 모든 ACTIVE 방 종료 후 LOCKED 처리 |
| `RoomPurgeScheduler` | `room/scheduler/` | 매월 1일 04:00 | 3개월 이상 LOCKED 방 영구 삭제 |
| `WebSocketActiveMemberReconciliationScheduler` | `room/scheduler/` | 30초마다 | stale `ACTIVE` 참여자를 `OFFLINE`으로 보정 |

`WebSocketRoomPresenceScheduler`는 disconnect 이후 3초 유예를 처리하는 내부 스케줄러입니다.
이 스케줄러는 `SchedulerStatusTracker` 관리 대상이 아니며, 상세 동작은 `docs/websocket/websocket-disconnect-handling.md`에 정리합니다.

---

## 상태 추적 아키텍처

운영 관리 대상 스케줄러는 `SchedulerStatusTracker`를 통해 실행 상태를 중앙 관리합니다.

```
@PostConstruct → tracker.register(name, description, schedule)

@Scheduled 실행 시:
  tracker.recordStart(name)
  try {
      // 비즈니스 로직
      tracker.recordSuccess(name, message)
  } catch {
      tracker.recordFailure(name, error)
  }
```

### SchedulerStatus 필드

| 필드 | 타입 | 설명 |
|-----|------|------|
| `name` | String | 스케줄러 고유 이름 |
| `description` | String | 역할 설명 |
| `schedule` | String | 실행 주기 |
| `running` | boolean | 현재 실행 중 여부 |
| `lastStatus` | String | `IDLE` / `SUCCESS` / `FAILED` |
| `lastResult` | String | 마지막 실행 결과 메시지 |
| `lastRunAt` | LocalDateTime | 마지막 실행 시각 |

---

## StatsBatchScheduler 상세

### 처리 흐름

```
1. PENDING 상태 BattleRecord 조회
2. 각 기록에서 defenseCombo, attackCombo 추출
3. combo가 빈 값이면 → SKIPPED 처리 (집계 제외)
4. MatchupStat 조회 (없으면 새로 생성)
5. addResult(isWin) → 통계 반영
6. BattleRecord 상태 → ACTIVE
```

### BattleRecordStatus 전이

```
PENDING ───┬──→ ACTIVE   (정상 집계 완료)
           └──→ SKIPPED  (불완전 데이터)
```

### 예외 처리
- **개별 기록 예외**: 해당 기록만 건너뛰고 나머지 계속 처리
- **전체 예외**: `tracker.recordFailure()` 기록 후 예외 재전파

---

## RoomScheduler 상세

### 처리 흐름

```
1. closeAllActiveRooms() — 모든 ACTIVE 방 → CLOSED (closedAt 기록)
2. lockClosedRooms() — CLOSED 상태의 방 → LOCKED
```

### 예외 처리
- `tracker.recordFailure()` 기록 후 예외 재전파

---

## RoomPurgeScheduler 상세

### 처리 흐름

```
1. 3개월 이상 LOCKED 상태인 방 조회 (closedAt 기준)
2. 해당 방 영구 삭제 (CASCADE로 관련 데이터 포함)
```

### 예외 처리
- `tracker.recordFailure()` 기록 후 예외 재전파

---

## WebSocketActiveMemberReconciliationScheduler 상세

### 처리 흐름

```
1. DB에서 ACTIVE room_member의 distinct memberId 조회
2. SimpUserRegistry에서 현재 live WebSocket memberId 집합 수집
3. DB ACTIVE - live memberId 차집합 계산
4. stale memberId만 markRoomsOffline(memberId) 호출
```

### 목적
- disconnect 이벤트 유실
- 브라우저 / 앱 강제 종료
- 서버 재시작 후 남아 있는 stale `ACTIVE`

상황을 주기적으로 정리해 room 참여 상태를 복구합니다.

### 예외 처리
- `tracker.recordFailure()` 기록 후 예외 재전파

---

## 테스트

| 테스트 | 파일 | 케이스 수 |
|-------|-----|---------|
| `MemberPurgeSchedulerTest` | `test/.../global/scheduler/` | 2 |
| `StatsBatchSchedulerTest` | `test/.../stats/scheduler/` | 5 |
| `RoomSchedulerTest` | `test/.../room/scheduler/` | 2 |
| `RoomPurgeSchedulerTest` | `test/.../room/scheduler/` | 2 |
| `WebSocketActiveMemberReconciliationSchedulerTest` | `test/.../room/scheduler/` | 2 |
| `AdminSystemServiceTest` | `test/.../admin/service/` | 6 |
