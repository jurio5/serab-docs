# Flyway baseline 가이드

기존 운영 DB를 기준점으로 편입한 뒤, 이후 변경점만 SQL 마이그레이션으로 관리하는 기준 문서입니다.

현재 프로젝트는 다음 기준으로 설정되어 있습니다.

- 운영 DB는 Flyway baseline version `1` 로 간주
- 운영 환경에서는 JPA가 스키마를 직접 변경하지 않음
- 운영 환경에서는 `ddl-auto: validate`
- 이후 스키마 변경은 `db/migration` 아래 SQL 파일로 직접 관리

---

## 기본 경로

Flyway 기본 경로:

- `src/main/resources/db/migration`

이 경로 아래에 SQL 파일을 두면 앱 시작 시 Flyway가 자동으로 읽습니다.

---

## 파일명 규칙

기본 형식:

- `V<version>__<description>.sql`

예:

- `V2__add_member_status_message.sql`
- `V3__create_room_member_index.sql`
- `V4__add_notice_expires_at.sql`

규칙:

- 앞에 `V`
- 버전 번호
- 언더바 두 개 `__`
- 설명
- 확장자 `.sql`

---

## 현재 프로젝트 기준

현재 운영 DB는 baseline version `1` 로 편입하는 방식입니다.

즉 앞으로는 다음처럼 관리합니다.

- 현재 운영 DB 상태 = `V1`
- 다음 변경 = `V2`
- 그다음 변경 = `V3`

예를 들어 `Member` 엔티티에 컬럼을 하나 추가했다면, Flyway가 엔티티를 읽고 자동으로 SQL을 생성하는 것이 아니라 개발자가 직접 SQL을 작성해야 합니다.

예:

```sql
ALTER TABLE member
ADD COLUMN status_message VARCHAR(255) NULL;
```

이 SQL은 다음과 같은 파일로 저장합니다.

- `src/main/resources/db/migration/V2__add_member_status_message.sql`

---

## 적용 흐름

1. 엔티티 변경
2. 변경점 SQL 작성
3. `db/migration` 경로에 버전 파일 추가
4. 배포
5. 앱 시작 시 Flyway가 미적용 버전 SQL 실행
6. 이후 JPA `validate` 수행

즉 운영 환경에서는:

- Flyway = 변경 SQL 실행
- JPA = 엔티티와 DB 구조 검증

역할로 나뉩니다.

---

## 왜 baseline 방식으로 가는가

현재 프로젝트는 이미 운영 DB가 존재합니다.

이 상태에서 전체 스키마를 처음부터 다시 만드는 것보다:

- 현재 운영 DB를 baseline 으로 편입하고
- 이후 변경점만 SQL로 관리하는 방식

이 더 안전합니다.

즉 신규 프로젝트처럼 `V1__init.sql` 로 모든 테이블을 처음부터 만드는 방식보다, 현재 운영 상태를 기준점으로 잡는 방식이 더 자연스럽습니다.

---

## 주의할 점

- Flyway는 엔티티를 보고 SQL을 자동 생성하지 않음
- 버전 번호는 직접 관리해야 함
- 운영 환경에서는 파괴적 변경을 바로 넣지 않는 것이 안전함
- 컬럼 추가 -> 배포 -> 코드 전환 -> 정리 순서를 우선 고려

---

## 첫 변경 예시

현재 baseline 이 `1` 이므로, 첫 실제 마이그레이션 파일은 보통 `V2__...sql` 로 시작합니다.

예:

- `V2__add_member_status_message.sql`
