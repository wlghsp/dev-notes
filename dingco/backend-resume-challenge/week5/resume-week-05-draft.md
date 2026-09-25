# Week 5 resume.md 문장 6 초안

대상 저장소의 `resume/resume.md`에는 이미 문장 1~5까지 채워져 있다("최종 문장은 세 개에서 다섯 개만 남긴다" 규칙상, 5주차 반영 시 가장 약한 문장 하나를 빼야 할 수 있다 — 이 부분은 지호님이 직접 5개 중 순위를 정한다). 5주차 내용은 **문장 6**으로 추가한다.

```markdown
### 문장 6

- 문장: 댓글 목록 API에서 작성자를 지연 로딩으로 조회하며 발생하는 N+1(댓글 30건 기준 쿼리 31건)을 Hibernate Statistics로 측정해 확인한 뒤 fetch join을 적용해, 같은 조건에서 쿼리 수를 1건(약 31배), 응답 시간을 178ms에서 13.7ms(약 13배)로 줄였다.
- 본인 행동: 측정 컴포넌트로 개선 전/후 쿼리 수를 실측하고, `@BatchSize`·fetch join·`@EntityGraph`를 비교해 fetch join을 적용했다. 응답 계약 동일성과 쿼리 수 상한을 검증하는 회귀 테스트를 작성했고, 선택 확장으로 Grafana 대시보드에 쿼리 수·응답 시간 패널을 구성했다.
- 저장소 안 근거: `CommentRepository`/`StudyQueryService`(개선 코드), `CommentQueryTest.java`(회귀 테스트), `evidence/week-05__weekly-pr.md`(측정 결과·Grafana 캡처)
- 한계: 시드 데이터 기준 댓글 30건 단일 스터디에서의 측정이며, 댓글 수가 훨씬 많아지는 경우(페이징 도입 시점)는 이번 측정 범위 밖이다.
```
