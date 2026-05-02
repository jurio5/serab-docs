# 데이터 모델 설계

개발을 시작하기 전, 개발을 진행하며 추가된 도메인별 엔티티 구조와 필드 타입, 제약 조건, Enum 값을 정리한 문서입니다.

프론트엔드 개발 시 API 응답 타입을 맞추거나, 백엔드에서 엔티티 간 관계를 확인할 때 참고할 수 있도록 남겨뒀습니다.\
ERD 이미지만으로는 컬럼 타입이나 제약 조건, Enum 값까지 확인하기 어려워서 별도로 정리했습니다.

관련 ERD는 [핵심 비즈니스 ERD](assets/erd-business-core.png)와 [운영/관리 ERD](assets/erd-admin-operation.png)를 참고하면 됩니다.

---

## 설계 기준

- 모든 엔티티는 `BaseEntity`를 상속하며, 공통 감사 필드 4개가 자동으로 들어갑니다.
- Enum은 DB에 문자열로 저장했습니다. (`EnumType.STRING`) 숫자 인덱스로 저장하면 나중에 순서가 바뀌었을 때 데이터가 깨질 수 있어서 문자열을 선택했습니다.
- 연관관계는 기본적으로 `FetchType.LAZY`를 사용했고, 필요한 경우에만 fetch join으로 가져옵니다.
- 비즈니스 도메인(방, 전투, 통계)과 운영 도메인(공지, 문의 등)을 패키지 단위로 분리했습니다.

### BaseEntity 공통 필드

| 필드 | 타입 | 설명 |
|-----|------|------|
| createDate | `LocalDateTime` | 생성 시각 (자동, 수정 불가) |
| updateDate | `LocalDateTime` | 마지막 수정 시각 (자동) |
| createdBy | `Long` | 생성자 ID (자동, 수정 불가) |
| modifiedBy | `Long` | 수정자 ID (자동) |

---

## 핵심 비즈니스 도메인

### Member

사용자 계정 정보를 관리합니다. OAuth 로그인 후 최초 가입 시 생성되며, 닉네임 변경에는 24시간 쿨다운이 있습니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | member_id |
| nickname | `String(30)` | NOT NULL, UNIQUE | 서비스 내 닉네임 |
| email | `String` | NOT NULL, UNIQUE | OAuth에서 가져온 이메일 |
| role | `Role` | NOT NULL | GUEST / MEMBER / ADMIN |
| nicknameChangedAt | `LocalDateTime` | nullable | 마지막 닉네임 변경 시각 |
| suspendedUntil | `LocalDateTime` | nullable | 정지 해제 시각 |
| suspendReason | `String(200)` | nullable | 정지 사유 |

### OAuth

OAuth 로그인 연동 정보입니다. Member와 1:1 관계이고, provider + oauthId 조합으로 유니크 제약이 있습니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | oauth_id |
| member | `Member` | NOT NULL, 1:1 | 연결된 회원 |
| provider | `OAuthProvider` | NOT NULL | KAKAO / GOOGLE |
| oauthId | `String` | NOT NULL | provider별 고유 식별자 |

### Room

길드전 방 정보입니다. 생성 시 아군/적군 성이 자동으로 초기화되고, 방장이 함께 멤버로 등록됩니다.

| 필드 | 타입 | 제약 | 설명                           |
|-----|------|------|------------------------------|
| id | `Long` | PK, AUTO | room_id                      |
| title | `String(100)` | NOT NULL | 방 제목                         |
| guildWarDate | `LocalDate` | NOT NULL | 길드전 날짜                       |
| owner | `Member` | FK (nullable) | 현재 방장 (대기방 모드에서 방장이 없을 수 있음) |
| status | `RoomStatus` | NOT NULL | ACTIVE / CLOSED / LOCKED     |
| password | `String(100)` | nullable | 비밀번호 (없으면 공개방)               |
| waitingMode | `boolean` | NOT NULL | 대기방 모드 여부 (기본 true)          |
| closedAt | `LocalDateTime` | nullable | 종료 시각                        |
| members | `List<RoomMember>` | cascade ALL | 방 참여자 목록                     |
| castles | `List<Castle>` | cascade ALL | 성 목록 (생성 시 자동 초기화)           |

### RoomMember

방 참여자 상태를 관리합니다. room_id + member_id 복합 유니크 제약이 있고, 멤버 상태별 인덱스도 걸어뒀습니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | room_member_id |
| room | `Room` | NOT NULL, FK | 소속 방 |
| member | `Member` | NOT NULL, FK | 회원 정보 |
| role | `RoomRole` | NOT NULL | OWNER / MANAGER / USER / OBSERVER |
| status | `RoomMemberStatus` | NOT NULL | ACTIVE / OFFLINE / KICKED |

### Castle

방 안의 성 정보입니다. 방 생성 시 아군/적군 각각 외성 5개, 내성 3개, 본성 1개가 자동 생성됩니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | castle_id |
| room | `Room` | NOT NULL, FK | 소속 방 |
| side | `CastleSide` | NOT NULL | ALLY / ENEMY |
| type | `CastleType` | NOT NULL | OUTER / INNER / MAIN |
| number | `Integer` | NOT NULL | 성 번호 (같은 타입 내 순서) |
| defenseFormation | `Formation` | nullable | 방어 진형 |
| memo | `String(1000)` | nullable | 성 메모 |
| defenderSlots | `List<DefenderSlot>` | cascade ALL | 방어 슬롯 목록 |
| battleRecords | `List<BattleRecord>` | cascade ALL | 전투 기록 목록 |

### DefenderSlot

성 안의 방어 슬롯입니다. 각 슬롯에 방어팀 이름, 영웅, 펫, 스킬 순서 등이 저장됩니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | defender_slot_id |
| castle | `Castle` | NOT NULL, FK | 소속 성 |
| position | `Integer` | NOT NULL | 슬롯 위치 |
| playerName | `String` | nullable | 방어팀 이름 |
| formation | `Formation` | nullable | 방어 진형 |
| memo | `String(500)` | nullable | 메모 |
| petName | `String` | nullable | 펫 이름 |
| defenseSkillOrder | `String(200)` | nullable | 방어 스킬 순서 |
| result | `BattleResult` | nullable | 슬롯 결과 (WIN / LOSE) |
| reservedByMemberId | `Long` | nullable | 예약자 ID |
| reservedByNickname | `String` | nullable | 예약자 닉네임 |
| heroes | `List<DefenseHero>` | cascade ALL | 방어 영웅 목록 |

### BattleRecord

전투 기록 원본 데이터입니다. 통계 집계의 입력값이 되는 핵심 엔티티입니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | battle_record_id |
| defenderSlot | `DefenderSlot` | NOT NULL, FK | 대상 방어 슬롯 |
| castle | `Castle` | NOT NULL, FK | 소속 성 |
| result | `BattleResult` | NOT NULL | WIN / LOSE |
| status | `BattleRecordStatus` | NOT NULL | PENDING / ACTIVE / SKIPPED / DISABLED |
| attackFormation | `Formation` | nullable | 공격 진형 |
| attackerName | `String` | nullable | 공격자 이름 |
| attackPetName | `String` | nullable | 공격 펫 이름 |
| recordedBy | `Member` | NOT NULL, FK | 기록자 |
| defenseCombo | `String(200)` | NOT NULL | 방어 조합 (정렬된 영웅 이름, 콤마 구분) |
| attackCombo | `String(200)` | NOT NULL | 공격 조합 (정렬된 영웅 이름, 콤마 구분) |
| defenseSkillOrder | `String(200)` | nullable | 방어 스킬 순서 |
| attackSkillOrder | `String(200)` | nullable | 공격 스킬 순서 |
| attackTeam | `List<AttackHero>` | cascade ALL | 공격 영웅 목록 |

### AttackHero / DefenseHero

전투 기록이나 방어 슬롯에 소속된 개별 영웅 정보입니다. 구조는 동일하고 소속 관계만 다릅니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | attack_hero_id / defense_hero_id |
| heroName | `String(50)` | NOT NULL | 영웅 이름 |
| position | `Integer` | NOT NULL | 배치 위치 |

- `AttackHero`는 `BattleRecord`에 소속
- `DefenseHero`는 `DefenderSlot` 또는 `BattleRecord`에 소속 (둘 중 하나만 연결)

### ChatMessage

방 채팅 메시지입니다. Redis 버퍼에서 주기적으로 DB에 flush됩니다.

| 필드 | 타입 | 제약 | 설명 |
|-----|------|------|------|
| id | `Long` | PK, AUTO | - |
| roomId | `Long` | NOT NULL, INDEX | 방 ID |
| senderId | `Long` | nullable | 발신자 ID (시스템 메시지는 null) |
| senderNickname | `String(50)` | nullable | 발신자 닉네임 |
| role | `String(20)` | nullable | 발신자 역할 |
| type | `String(10)` | nullable | 메시지 타입 |
| content | `String(200)` | NOT NULL | 메시지 내용 |

세부 도메인은 [데이터 모델 상세](data-model-detail.md)에서 이어집니다.
