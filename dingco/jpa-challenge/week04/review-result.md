# Week 4 자동 리뷰 결과

PR: https://github.com/GaeChwiPpo/challenge-jpa-deep-dive-2026-08-wlghsp-r8/pull/4 (머지 완료, 2026-09-12)

- 점수: 82/100
- 자동 판정 신뢰도: 88%
- 판정: 자동 리뷰 통과

## 잘한 점

- SINGLE_TABLE·JOINED 두 전략을 완전히 분리된 클래스·테이블로 구현하고, DDL·INSERT·다형성 조회 SQL 로그를 evidence 파일에 재현 가능한 형태로 남김
- 테스트 5개가 INSERT 횟수, nullable 컬럼 존재, 조인 유무를 코드 레벨에서 직접 단언(assertion)해 다른 사람이 mvn test로 재확인 가능
- ADR에서 선택 전략(JOINED)·버린 전략(SINGLE_TABLE)의 근거와 재평가 조건을 명시하고, 근거형 질문 1~4 모두 본인 코드·로그를 인용해 답변

## 보완할 점 (다음 주차 이후 필요 시 참고)

- 실제 실행 시간(ms) 측정 없이 구조 확인에서 멈춤 — 정량 측정 또는 측정 불가 이유를 더 구체적으로 기록했어야 함
- evidence의 "리뷰 반영" 항목이 "자동 리뷰 수신 전"으로 비어 있어 리뷰 반영 증거 없음
- `BankTransferPaymentJoined`/`BankTransferPaymentSingleTable`에 `backCode` 오타(`bankCode`가 맞음), `PaymentJoined`에 getter 없음

## 처리 상태

PR이 이미 머지되어 같은 PR에는 반영 불가. 오타·getter 수정은 미반영 상태로 남겨둠 — 필요 시 별도 커밋으로 처리.
