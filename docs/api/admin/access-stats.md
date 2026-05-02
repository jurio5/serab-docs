# 접속 통계 API

접속 유형별 통계를 기록하고 조회하는 API입니다.  
프론트엔드에서 페이지 로드 시 자동으로 호출되며, 관리자 대시보드에서 집계 데이터를 확인할 수 있습니다.

---

## 1. 접속 기록

POST /access/record

### [설명]

현재 접속 유형(PWA / 브라우저 / 인앱)을 기록합니다.  
인메모리 카운터(실시간)와 DB(히스토리) 양쪽에 동시 저장됩니다.

### [Request]

```json
{
  "type": "standalone"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|:--:|------|
| type | String | O  | 접속 유형 (대소문자 무관) |

### 접속 유형 (`AccessType`)

| 값 | 설명 | 판별 기준 |
|----|------|-----------|
| STANDALONE | PWA 앱 | `display-mode: standalone` |
| BROWSER | 일반 브라우저 | 기본값 |
| IN_APP_BROWSER | 인앱 브라우저 | 딥링크 경유 |

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

### [실패 응답]

| ErrorCode | HTTP Status | 설명 |
|-----------|-------------|------|
| (IllegalArgumentException) | 500 | 유효하지 않은 접속 유형 |

---

## 2. PWA 설치 기록

POST /access/pwa-install

### [설명]

PWA 설치 이벤트를 기록합니다.  
`beforeinstallprompt` → 설치 완료 시 프론트에서 호출합니다.  
인메모리에만 저장됩니다 (서버 재시작 시 초기화).

### [Request]

별도 파라미터 없음

### [성공 응답]

```json
HTTP 200
{
  "code": "OK",
  "data": null
}
```

---

## [내부 처리 흐름]

```
프론트 (useAccessTracker.ts)
  ↓ 페이지 로드 시 display-mode 판별
  ↓ POST /access/record { type: "standalone" | "browser" | "in_app_browser" }
  ↓
AccessStatsController → AccessStatsService
  ↓
recordAccess(typeStr)
  ├─ AccessType.valueOf()      ← 유효성 검증 (실패 시 GlobalExceptionHandler)
  ├─ incrementMemoryCount()    ← ConcurrentHashMap 카운터 +1
  └─ upsertDailyStat()         ← DB: 있으면 increment, 없으면 create
```

### 통계 조회 흐름 (관리자 대시보드)

```
AdminController → AdminDashboardService → AccessStatsService.getTodayStats()
  ↓
인메모리 데이터 있음? → buildFromMemory()  (서버 재시작 전)
인메모리 비어있음?   → buildFromDb()       (서버 재시작 후)
  ↓
AdminDashboardResponse에 포함되어 반환
```

---

## [데이터 저장 구조]

### 인메모리 (실시간)

```
Map<LocalDate, Map<AccessType, AtomicInteger>>  ← 접속 유형별 일별 카운트
Map<LocalDate, AtomicInteger>                   ← PWA 설치 일별 카운트
```

- 서버 재시작 시 초기화
- 실시간 대시보드 표시용

### DB (`access_daily_stats` 테이블)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| id | BIGINT (PK) | 자동 생성 |
| access_date | DATE | 날짜 |
| access_type | VARCHAR(20) | STANDALONE / BROWSER / IN_APP_BROWSER |
| count | INT | 일별 집계 카운트 |

- UNIQUE 제약조건: `(access_date, access_type)`
- 하루 최대 3행 (접속 유형 3개)
- 서버 재시작 후에도 히스토리 유지

---

## [프론트 연동]

### `useAccessTracker.ts`

페이지 최초 마운트 시 1회 호출:

```typescript
const type = window.matchMedia('(display-mode: standalone)').matches
  ? 'standalone'
  : isInAppBrowser()
    ? 'in_app_browser'
    : 'browser';

fetch('/access/record', { method: 'POST', body: JSON.stringify({ type }) });
```

### `usePWAInstall.ts`

PWA 설치 완료 시 호출:

```typescript
fetch('/access/pwa-install', { method: 'POST' });
```

---

## [테스트]

| 테스트 클래스 | 케이스 수 | 주요 검증 |
|--------------|:---------:|----------|
| `AccessStatsServiceTest` | 7 | 신규/기존 기록, 잘못된 타입, 대소문자, 인메모리/DB 폴백, PWA |
| `AccessStatsControllerTest` | 2 | 서비스 위임 확인 |
