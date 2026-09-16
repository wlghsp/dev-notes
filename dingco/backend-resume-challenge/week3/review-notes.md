# 3주차 PR 자동 리뷰 기록

PR #3 "신청 검색 API를 실행 계획으로 개선하기"에 대한 자동 리뷰 결과.

- 점수: 88/100
- 자동 판정 신뢰도: 87%
- 결과: 통과

## 잘한 점

- search·stats 두 API 모두 EXPLAIN 전후 비교와 수치가 동일 데이터 조건에서 evidence에 기록됨
- 복합 인덱스 `(status, enrolled_at DESC, fee)` 컬럼 순서를 본인 쿼리 패턴 근거(등치 조건 → 범위·정렬 조건 → 후속 필터)로 설명하고, 버린 대안과 그 대안이 유리해지는 조건까지 서술함
- `EnrollmentControllerTest`에 search 경계값·정렬·필터와 stats 회원별 count·totalFee·lastEnrolledAt을 검증하는 회귀 테스트가 추가되고 통과 확인됨

## 보완할 점과 대응 계획

- stats HTTP API 개선 전 응답 시간이 기록되지 않아, 전후 비교가 DB `EXPLAIN ANALYZE`로만 대체됐다. 본인도 evidence에 한계로 명시함.
  - 대응 계획: 다음에 이런 상황이 생기면 개선 코드를 적용하기 전에 HTTP 레벨 측정부터 먼저 남긴다. 이번엔 집계 테이블 전환 전 HTTP 측정을 놓쳤다.
- `member_enrollment_fee_stats`의 `ORDER BY enrollment_count DESC` 정렬 비용이 evidence에 언급됐으나, 이를 줄이는 추가 인덱스 검토가 없었다.
  - 대응 계획: `(fee, enrollment_count DESC, member_id)` 인덱스를 검토하고 EXPLAIN으로 Sort 제거 여부를 확인하는 것을 다음 측정 과제로 남긴다.
- 신청 취소·수정 시 집계 테이블 보정 로직이 질문 4 답변에서만 언급되고 코드 수준 흔적(TODO 등)이 없다.
  - 대응: 현재 취소·수정 API 자체가 없어 범위 밖 판단. 해당 API가 생기면 보정 로직을 함께 구현한다.
- 선택 확장(인덱스 적용 후 INSERT·UPDATE 트레이드오프 측정)은 선택 항목이라 미수행, 감점 없음.

## 참고

- 근거 인용, 상세 리뷰 링크는 PR #3 코멘트 원문 참고 (dingco.net 챌린지 화면)
- evidence 원본 파일(`evidence/week-03__weekly-pr.md`)의 `## 리뷰 반영`은 과제 제출 계약에 따라 지호님이 직접 간단히 채운다. 이 파일은 리뷰 내용을 dev-notes 쪽에서 따로 남겨두는 기록용이다.
