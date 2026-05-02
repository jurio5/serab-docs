# Member Withdraw (회원 탈퇴) 처리

**OAuth 기반 인증 환경**에서 구현된
회원 탈퇴(Withdraw) 로직을 정리한 문서 입니다.

---

## 1. 회원 탈퇴 설계 개요

회원 탈퇴는 단순한 데이터 삭제가 아닌,
**유예 기간을 둔 Soft Delete + 자동 정리 방식**으로 설계되었습니다.

### 설계 목적

* 사용자 실수로 인한 탈퇴 복구 가능성 확보
* 법적/정책적 요구사항을 고려한 개인정보 보관
* 삭제 이력과 원본 데이터의 책임 분리
* 데이터 무결성과 트랜잭션 안정성 보장

---

## 2. 전체 처리 흐름

### 회원 탈퇴 시 데이터 이동

```
Member        → MemberDeleted (30일 보관)
OAuth         → OAuthDeleted (30일 보관)
```

### 처리 순서 요약

1. 회원(Member) 조회
2. 회원의 OAuth 정보 조회
3. 삭제 전 데이터 백업 (삭제 테이블로 이동)
4. 원본 회원 데이터 삭제
5. OAuth 데이터는 Aggregate Root 정책에 따라 함께 제거

---

## 3. 도메인 설계

### MemberDeleted

* 탈퇴된 회원의 핵심 식별 정보 보관
* 복구 및 만료 기준 시점을 함께 관리

주요 필드:

* `id` : 원본 Member의 PK 유지
* `nickname` : 탈퇴 시점 닉네임
* `email` : 탈퇴 시점 이메일
* `deletedAt` : 탈퇴 시각
* `expireAt` : 영구 삭제 예정 시각 (deletedAt + 30일)

```java
public static MemberDeleted from(Member member) {
    MemberDeleted deleted = new MemberDeleted();
    deleted.id = member.getId();
    deleted.nickname = member.getNickname();
    deleted.email = member.getEmail();
    deleted.deletedAt = LocalDateTime.now();
    deleted.expireAt = deleted.deletedAt.plusDays(30);
    return deleted;
}
```

---

### OAuthDeleted

* OAuth 인증 정보를 별도 테이블에 보관
* Member와 생명주기를 분리하되, 복구 가능성을 고려해 유지

주요 필드:

* `id` : 원본 OAuth PK
* `memberId` : 탈퇴된 회원 ID
* `provider` : OAuth 제공자
* `oauthId` : OAuth 고유 식별자

```java
public static OAuthDeleted from(OAuth oAuth) {
    OAuthDeleted od = new OAuthDeleted();
    od.id = oAuth.getId();
    od.memberId = oAuth.getMember().getId();
    od.provider = oAuth.getProvider();
    od.oauthId = oAuth.getOauthId();
    return od;
}
```

---

## 4. 회원 탈퇴 처리 로직

회원 탈퇴는 **단일 트랜잭션**으로 처리되며,
중간 단계에서 예외가 발생할 경우 전체가 롤백됩니다.

```java
@Transactional
public void withdraw(Long memberId) {
    Member member = memberRepository.findById(memberId)
            .orElseThrow(MEMBER_NOT_FOUND::toException);

    OAuth oauth = member.getOAuth();

    // 1. 삭제 테이블에 백업
    memberDeletedRepository.save(MemberDeleted.from(member));
    oAuthDeletedRepository.save(OAuthDeleted.from(oauth));

    // 2. 원본 회원 삭제
    memberRepository.delete(member);
}
```

### 처리 포인트

* **삭제 전 백업 선행**
* 원본 데이터 삭제는 항상 마지막에 수행
* Member가 Aggregate Root이므로 OAuth는 함께 관리

---

## 5. 자동 영구 삭제 (Purge)

탈퇴 후 **30일이 경과한 데이터**는
스케줄러를 통해 자동으로 영구 삭제됩니다.

### 스케줄러 실행 정책

* 실행 주기: 매일 03:00
* 기준: `expireAt < 현재 시각`

```java
@Component
@Transactional
public class MemberPurgeScheduler {

    @Scheduled(cron = "0 0 3 * * *")
    public void purgeExpiredMembers() {
        List<MemberDeleted> expired = memberDeletedRepository
                .findAllByExpireAtBefore(LocalDateTime.now());

        if (expired.isEmpty()) {
            return;
        }

        List<Long> memberIds = expired.stream()
                .map(MemberDeleted::getId)
                .toList();

        oAuthDeletedRepository.deleteAllByMemberIdIn(memberIds);
        memberDeletedRepository.deleteAllByIdInBatch(memberIds);
    }
}
```

---

## 6. 트랜잭션 & 무결성 보장

* 회원 탈퇴 로직은 하나의 트랜잭션으로 관리
* 백업 실패 시 원본 삭제 불가
* 스케줄러 또한 트랜잭션 기반으로 동작
* 부분 삭제로 인한 데이터 불일치 방지

---

## 7. 보안 및 개인정보 보호 고려

* 탈퇴 회원 데이터는 30일 후 자동 제거
* 불필요한 개인정보 장기 보관 방지
* OAuth 식별자도 동일 정책 적용
* 관리자 개입 없이 자동 정리

---

## 8. 정리

본 회원 탈퇴 설계는 다음 특징을 가집니다:

* **복구 가능성을 고려한 Soft Delete 구조**
* **Aggregate Root 중심의 도메인 책임 분리**
* **스케줄러 기반 자동 데이터 정리**
* **트랜잭션 기반 안전한 탈퇴 처리**
