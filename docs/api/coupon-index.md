# Coupon API Index

쿠폰 기능 관련 API 문서 모음입니다.

---

## Public API

- [활성 쿠폰 목록 조회](./coupon/list-active.md)
- [쿠폰 적용](./coupon/redeem.md)

## Admin API

- [관리자 쿠폰 관리](./admin/coupons.md)

---

## Notes

- 쿠폰 적용 API는 `PID`와 `couponCode`를 받아 외부 넷마블 쿠폰 서버를 호출합니다.
- `PID`는 서버에 저장하지 않는 전제를 기준으로 설계되어 있습니다.
