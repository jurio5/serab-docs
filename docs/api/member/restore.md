# Member Restore(회원 복구) 처리

OAuth 기반 인증 환경에서 탈퇴한 회원이 재로그인 시  
자동으로 복구되는 전체 흐름을 설명합니다.

---

## 1. 회원 복구 처리 개요

JWT 기반 환경에서 회원 탈퇴는 즉시 삭제가 아닌  
**30일 보관 후 자동 삭제(Soft Delete)** 방식으로 구현됩니다:

1. 회원 탈퇴 시 별도 테이블로 이동 (MemberDeleted, OAuthDeleted)
2. 30일 이내 OAuth 재로그인 시 자동 복구
3. 30일 경과 시 스케줄러가 자동 영구 삭제

즉, "회원 복구"는 삭제 테이블에서 원본 테이블로 데이터를 복원하는 과정입니다.

---

## 2. 회원 복구 흐름

### 탈퇴 시 처리
```
Member → MemberDeleted (30일 보관)
OAuth  → OAuthDeleted (30일 보관)
```

### 복구 처리
```
[사용자]
   ↓ OAuth 재로그인
CustomOAuth2AuthenticationSuccessHandler
   ├─ 1. 삭제된 회원인가? (MemberDeleted 조회)
   ├─ 2. Yes → 중복 검증 (이메일, 닉네임, OAuth)
   ├─ 3. 복구 진행
   │    ├─ MemberDeleted → Member
   │    └─ OAuthDeleted → OAuth
   ├─ 4. JWT 발급
   └─ 5. status=RESTORED 리다이렉트
   
[30일 경과 시]
스케줄러가 매일 03:00에 자동 영구 삭제
```

---

## 3. 전체 흐름 다이어그램

```
회원 탈퇴
  ↓
Member → MemberDeleted (30일 보관)
OAuth → OAuthDeleted (30일 보관)
  ↓
[30일 이내] OAuth 재로그인
  ↓
삭제된 회원 확인
  ├─ Yes → 중복 검증
  │         ├─ 성공 → 자동 복구 → status=RESTORED
  │         └─ 실패 → status=RESTORE_FAILED
  └─ No → 정상 로그인 또는 신규 가입
  
[30일 경과] 스케줄러가 자동 영구 삭제
```

---

## 4. 핵심 코드

### 회원 탈퇴 처리
```java
@Transactional
public void withdraw(Long memberId) {
    Member member = memberRepository.findById(memberId)
            .orElseThrow(MEMBER_NOT_FOUND::toException);
    
    OAuth oauth = member.getOAuth();
    
    // 삭제 테이블에 백업
    memberDeletedRepository.save(MemberDeleted.from(member));
    oAuthDeletedRepository.save(OAuthDeleted.from(oauth));
    
    // 원본 삭제 (Cascade로 OAuth도 함께 삭제)
    memberRepository.delete(member);
}
```

### OAuth 로그인 시 자동 복구
```java
@Override
public void onAuthenticationSuccess(...) {
    SecurityUser user = (SecurityUser) authentication.getPrincipal();
    OAuthProvider provider = OAuthProvider.from(user.getProvider());
    
    // 1. 삭제된 회원인지 먼저 확인
    Optional<MemberDeleted> deletedOpt = memberDeletedRepository
            .findByProviderAndOauthId(provider, user.getOauthId());
    
    if (deletedOpt.isPresent()) {
        handleDeletedMemberRestore(response, deletedOpt.get(), user);
        return;
    }
    
    // 2. 기존 로직 (신규 or 기존 회원)
    memberRepository.findByProviderAndOauthId(provider, user.getOauthId())
            .ifPresentOrElse(
                member -> handleExistingUserLogin(response, user, member),
                () -> handleNewUserRegistration(response, user)
            );
}
```

### 복구 처리 로직
```java
@Transactional
public Member restoreMember(Long memberId) {
    // 1. 이미 복구된 회원인지 확인
    if (memberRepository.existsById(memberId)) {
        throw MEMBER_ALREADY_RESTORED.toException();
    }
    
    // 2. 삭제된 데이터 조회
    MemberDeleted deleted = memberDeletedRepository.findById(memberId)
            .orElseThrow(MEMBER_DELETED_NOT_FOUND::toException);
    
    OAuthDeleted oauthDeleted = oAuthDeletedRepository.findById(memberId)
            .orElseThrow(OAUTH_DELETED_NOT_FOUND::toException);
    
    // 3. 중복 검증 (이메일, 닉네임, OAuth)
    validateDuplicates(deleted, oauthDeleted);
    
    // 4. 복구 진행
    Member member = Member.restore(deleted);
    OAuth.restore(member, oauthDeleted);
    
    memberRepository.save(member);
    oAuthDeletedRepository.delete(oauthDeleted);
    memberDeletedRepository.delete(deleted);
    
    return member;
}
```

### 자동 영구 삭제 스케줄러
```java
@Component
@Transactional
public class MemberPurgeScheduler {
    
    @Scheduled(cron = "0 0 3 * * *")  // 매일 03:00 실행
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
        
        log.debug("Purged {} expired members", memberIds.size());
    }
}
```

---

## 5. 클라이언트 연동

### 리다이렉트 URL 처리
```javascript
const params = new URLSearchParams(window.location.search);
const status = params.get('status');

switch(status) {
    case 'SUCCESS':
        // 일반 로그인 성공
        break;
    case 'REGISTER':
        // 신규 회원 가입
        break;
    case 'RESTORED':
        // 회원 복구 완료
        showNotification('계정이 복구되었습니다! 다시 만나서 반갑습니다.');
        break;
    case 'RESTORE_FAILED':
        // 복구 실패 (이메일/닉네임 중복 등)
        showError('계정 복구에 실패했습니다. 고객센터에 문의해주세요.');
        break;
}
```

---

## 6. 보안 고려 사항

### Aggregate Root 패턴
- **Member가 Aggregate Root**: OAuth는 Member의 일부로 취급
- **Cascade 설정**: Member 삭제/저장 시 OAuth도 자동 처리
- **생명주기 공유**: Member와 OAuth는 항상 함께 존재

### 중복 검증
- **이메일 중복**: 삭제 후 다른 사용자가 같은 이메일 사용 시 복구 실패
- **닉네임 중복**: 삭제 후 다른 사용자가 같은 닉네임 사용 시 복구 실패
- **OAuth 중복**: provider + oauthId 조합 중복 시 복구 실패

### 트랜잭션 관리
- 복구 과정은 하나의 트랜잭션으로 처리
- 중간에 예외 발생 시 전체 롤백
- 데이터 일관성 보장

### 개인정보 보호
- 30일 보관 정책으로 법적 요구사항 준수
- 스케줄러로 자동 영구 삭제
- 불필요한 개인정보 장기 보관 방지

---

## 7. 엣지 케이스 처리

### 이미 복구된 회원이 재로그인
→ 정상 로그인 처리 (status=SUCCESS)

### 복구 중 이메일 중복
→ EMAIL_ALREADY_EXISTS 예외 발생 (status=RESTORE_FAILED)

### 복구 중 닉네임 중복
→ NICKNAME_ALREADY_EXISTS 예외 발생 (status=RESTORE_FAILED)

### 30일 경과 후 로그인
→ 데이터 없음, 신규 회원으로 처리 (status=REGISTER)

---

## 8. 복구 결과

**정상 복구 시**
- MemberDeleted → Member 복원
- OAuthDeleted → OAuth 복원
- 삭제 테이블 데이터 제거
- JWT 발급 및 로그인 처리
- status=RESTORED 리다이렉트

**복구 실패 시**
- 중복 검증 실패 (이메일/닉네임/OAuth)
- 복구 진행하지 않음
- status=RESTORE_FAILED 리다이렉트
- 사용자에게 고객센터 안내