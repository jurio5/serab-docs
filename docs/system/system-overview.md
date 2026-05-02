# 시스템 관리 문서 목차

관리자용 시스템 모니터링 및 설정 관련 문서입니다.

---

## 기능 목록

| 기능 | 설명 |
|-----|------|
| Health Check | DB, Redis, WebSocket 연결 상태 확인 |
| 스케줄러 상태 | 등록된 스케줄러 실행 상태 조회/실행 |
| 비동기 처리 | WebSocket 이벤트 비동기 실행 설정 |

---

## Health Check (Actuator)

Spring Boot Actuator를 통해 서비스 상태를 확인합니다.

### 확인 대상

| 컴포넌트 | Indicator | 상세 |
|---------|-----------|------|
| MySQL | `DataSourceHealthIndicator` (자동) | JDBC 연결 확인 |
| Redis | `RedisHealthIndicator` (자동) | Redis 연결 확인 |
| WebSocket | `WebSocketHealthIndicator` (커스텀) | 접속자 수 표시 |

### 커스텀 Health Indicator

`WebSocketHealthIndicator`는 현재 접속 중인 WebSocket 유저 수를 표시합니다:
- 항상 UP 상태 반환
- `details`에 `connectedUsers` 값 포함

### 보안

| 엔드포인트 | 접근 권한 |
|-----------|---------|
| `/actuator/health` | `ROLE_ADMIN` |
| `/actuator/**` | `ROLE_ADMIN` |

---

## 비동기 처리 (AsyncConfig)

WebSocket 이벤트 처리를 위한 `ThreadPoolTaskExecutor` 설정:

| 설정 | 값 | 설명 |
|-----|---|------|
| `corePoolSize` | 5 | 기본 스레드 수 |
| `maxPoolSize` | 20 | 최대 스레드 수 |
| `queueCapacity` | 100 | 큐 용량 |
| `threadNamePrefix` | `async-` | 스레드 이름 접두사 |
| `waitForTasksToCompleteOnShutdown` | true | 종료 시 작업 완료 대기 |
| `awaitTerminationSeconds` | 30 | 종료 대기 시간 |

---

## API 엔드포인트

### 시스템 상태 조회
```
GET /api/admin/system/health
```
응답: DB, Redis, WebSocket 상태 + 스케줄러 상태 목록

### 스케줄러 수동 실행
```
POST /api/admin/system/scheduler/{name}/run
```
등록된 스케줄러를 즉시 실행합니다.

현재 수동 실행 가능한 이름은 다음과 같습니다.

- `room-cleanup`
- `room-purge`
- `stats-batch`
- `member-purge`
- `ws-presence-reconcile` : stale `ACTIVE` 참여자 오프라인 보정

---

## 관련 클래스

| 클래스 | 역할 |
|-------|------|
| `AdminSystemService` | 시스템 상태 조회 + 스케줄러 실행 |
| `SchedulerStatusTracker` | 스케줄러 상태 중앙 관리 |
| `AsyncConfig` | 비동기 처리 설정 |
| `WebSocketHealthIndicator` | WebSocket 커스텀 헬스 체크 |
| `SystemHealthResponse` | 시스템 상태 응답 DTO |
