# Week 4 resume.md 문장 5 초안

대상 저장소의 `resume/resume.md`에는 이미 문장 1~4까지 채워져 있다("최종 문장은 세 개에서 다섯 개만 남긴다" 규칙상 5개가 상한). 4주차 내용은 **문장 5**로 추가한다.

```markdown
### 문장 5

- 문장: 스터디 신청 API에 동시 요청 30건을 재현해 정원 10명인 스터디에서 신청 30건이 전부 성공·저장되고도 실제 정원(enrolledCount)은 4명으로 집계되는 Lost Update를 확인한 뒤 비관적 락(`SELECT ... FOR UPDATE`)을 적용해, 동일 조건 5회 반복 검증에서 성공 수를 매회 정확히 정원(10건)으로 맞췄다.
- 본인 행동: ExecutorService+CountDownLatch로 동일 studyId에 동시 요청을 보내는 재현 테스트를 작성해 정합성 붕괴를 실측했다. 비관적 락과 낙관적 락(@Version+재시도)을 같은 조건으로 비교해 정합성·성공 수·재시도 수 근거로 비관적 락을 최종 선택했고, 회원 조회를 락 획득 이전으로 옮겨 트랜잭션 범위를 점검했다.
- 전후 조건과 결과:
  - 조건: capacity=10, 동시 요청 30건, CountDownLatch로 동시 출발, 개선 후는 5회 반복 검증
  - 개선 전: successCount=30 / savedEnrollmentCount=30 / enrolledCount=4
  - 개선 후: 5회 반복 모두 successCount=enrolledCount=10, 실패 0건
- 저장소 안 근거:
  - 재현·검증 테스트: src/test/java/co/dingcodingco/studypass/enrollment/EnrollmentConcurrencyTest.java
  - 비관적 락 코드: src/main/java/co/dingcodingco/studypass/enrollment/EnrollmentService.java, src/main/java/co/dingcodingco/studypass/study/StudyRepository.java
  - 낙관적 락 비교 근거(정합성 5회 유지·매회 실패 20건·재시도 33~59회): evidence/week-04__weekly-pr.md
  - 측정 결과: evidence/week-04__weekly-pr.md
- 한계: 동시 요청 30건·capacity 10명이라는 로컬 조건에서의 측정이며, 실제 운영 트래픽의 순간 동시성 규모나 DB 커넥션 풀 한계는 이번 측정 범위 밖이다.
```
