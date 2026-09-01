# 1주차 실행 가이드

week1-prep-questions.md로 학습을 마친 뒤 실제로 진행하는 순서. 대상 저장소는 challenge-backend-resume-2026-08-wlghsp-r17(studypass), 대상 API는 GET /api/studies로 확정.

---

## 1단계: 브랜치 생성

홈페이지에서 `submit/week-01__weekly-pr` 형태의 브랜치를 생성한다 (missions/README.md 공통 제출 계약 1번).

## 2단계: 응답 시간 3회 이상 측정

`docker compose up -d`, `gradle bootRun`이 떠 있는 상태에서 같은 입력(`page=0&size=20`)으로 3회 이상 측정한다. 콜드 스타트(첫 요청)는 워밍업으로 별도 처리하고, 측정 대상에서 제외할지 포함할지 먼저 정한 뒤 그 판단을 evidence에 적는다.

```bash
# 워밍업 1회 (측정에서 제외)
curl -s -o /dev/null "http://localhost:8080/api/studies?page=0&size=20"

# 측정 3회 이상
for i in 1 2 3; do
  curl -s -o /dev/null -w "요청 %{time_total}s\n" "http://localhost:8080/api/studies?page=0&size=20"
done
```

기록해야 할 것: 각 회차의 응답 시간, page/size 값, 측정 시점의 시드 데이터 규모(`app.seed.*` 값), 실행 환경(로컬 스펙 정도).

```shell
jihochoi@Jiho-MacBook-Pro challenge-backend-resume-2026-08-wlghsp-r17 % for i in 1 2 3; do
  curl -s -o /dev/null -w "요청 %{time_total}s\n" "http://localhost:8080/api/studies?page=0&size=20"
done
요청 0.025574s
요청 0.012112s
요청 0.017677s
```

### 측정시점의 시드 데이터 규모
```
    members: 2000
    studies: 300
    enrollments: 200000
    commentsPerStudy: 30
```
### 실행 환경(로컬 스펙 정도)
- 맥북프로 M3 16GB

## 3단계: 문제 상황 예측 작성

지금까지 확인한 근거를 정리한다.

- `GET /api/studies`는 `WHERE` 없이 `LIMIT ?, ?` + 별도 `count(*)` 두 방을 날린다
- `study` 테이블은 PK 외 인덱스가 없다 (3주차 미션 대상이라 지금은 만들지 않음)
- 병목 후보 두 가지, 원인이 다름:
  - count 쿼리: `Page`가 총 개수를 요구해서 매 요청마다 전체 스캔
  - OFFSET 페이징: `LIMIT ?, ?`가 앞부분을 읽고 버려서 뒷페이지일수록 느려짐
- 이 둘은 독립적인 문제라 하나의 해결책으로 둘 다 안 풀린다 (커서 페이징은 OFFSET만 해결, Slice 전환은 count만 해결)

이 내용을 evidence의 "## 검증"과 resume.md의 근거 범위에 반영한다. 지금 단계에서는 구현하지 않고 예측·근거 제시만 한다.

- GET /api/studies?page=0&size=20은 WHERE절 없이 study 테이블 전체를 대상으로 LIMIT ?, ?와 별도의 count(s1_0.id) 쿼리 두 번을 날린다.
- 현재 study는 300건뿐이라 3회 측정 결과 12~26ms(0.012s/0.017s/0.025s)로 빠르다. 이건 테이블 크기가 작아서지, 쿼리 구조가 좋아서가 아니다.
- 데이터가 늘어나면(예: study가 수만 건으로 증가) count(*) 쿼리가 매 요청마다 테이블 전체를 스캔하므로 응답 시간이 테이블 크기에 비례해 늘어난다. 동시 요청이 늘어나면, 이 풀스캔 두 방이 요청마다 반복되므로 DB 부하가 요청 수만큼 배가된다.
- 뒷페이지로 갈수록 LIMI ?, ? 의 OFFSET이 커져 그만큼 더 많은 row를 읽고 버려야 하므로, 페이지 번호가 커질수록 응답 시간도 함께 늘어날 것으로 예측된다.
- study 테이블에 인덱스가 없지만, 지금 쿼리는 조건절이 없어 인덱스를 만들어도 실행계획이 달라지지 않는다. 인덱스 설계는 3주차 대상이므로 이번 주차는 개선 없이 예측과 근거만 남긴다.

## 4단계: 회귀 테스트 작성

`src/test/`에 `GET /api/studies` 대상 테스트를 추가한다. H2 기반이라 MySQL 전용 기능(예: 특정 함수, 스토리지 엔진 의존 동작)은 피한다.

고정해서 검증할 것:
- 상태 코드 200
- 응답 필드 존재 (`totalElements`, `content`, 그리고 `content` 안의 id/title/category/fee/capacity/enrolledCount)
- `size` 파라미터가 100을 넘겨도 100으로 캡되는 동작 (`Math.min(size, 100)`)
- 페이지네이션 동작 (`page`, `size` 반영)

## 5단계: 이력서 문장 작성

resume/resume.md의 "문장 1" 항목을 수치화 패턴으로 채운다.

- 문장: 수치 + 조건 + 결과 형태로 (예: "동시 요청 없는 로컬 환경에서 GET /api/studies 응답 시간을 3회 측정해 평균 Xms를 확인했다")
- 본인 행동: 무엇을 측정했고 무엇을 확인했는지
- 전후 조건과 결과: 개선 전이므로 "전"만 있고 "후"는 다음 주차로 명시
- 저장소 안 근거: evidence 파일 경로, 회귀 테스트 파일 경로를 텍스트로 명시
- 한계: 동시 요청 부하는 미측정, 실사용 트래픽 패턴과의 괴리 등 — 확인 못 한 값은 "확인 불가"로 표시

## 6단계: evidence 파일 작성

`evidence/week-01__weekly-pr.md`를 evidence/README.md 틀에 맞춰 작성한다.

- `## 변경`: 측정 대상(GET /api/studies), 변경한 이력서 문장, 개인정보 제거 확인
- `## 검증`: 실행 환경, 고정한 데이터/표본/명령(위 curl 명령과 시드 조건), 실행 결과(3회 측정값), 확인 불가 항목
- `## 선택 근거`: (선택 확장을 했다면) 비교한 대안과 선택 이유
- `## 근거형 질문`: 근거형 질문 1~4에 대해 최초 판단 → 연결한 코드/로그 → 검증 후 답변 순서로 작성
- `## 리뷰 반영`: 최초 제출은 "자동 리뷰 수신 전"으로 기록

## 7단계: PR 생성 및 자동 검사 확인

resume/resume.md와 src/(테스트 파일)를 **같은 PR에 함께** 포함해야 자동 검사를 통과한다. AI 리뷰에서 보완 요청이 오면 같은 PR에 수정 커밋을 push하고, 없으면 evidence의 "## 리뷰 반영"에 "지적 없음"을 기록한다.

---

## 근거형 질문 4개 — 답변 방향 메모

1. **병목 예측과 근거**: count 쿼리의 풀스캔 + OFFSET 페이징 비용, SQL 로그로 확인한 걸 근거로 제시
2. **측정의 대표성과 한계**: 단일 사용자·순차 요청 3회는 동시 요청 상황을 대표하지 못함 (2주차 k6로 보완 예정)
3. **팀 성과 vs 본인 기여 구분**: 1주차 시점엔 개인 프로젝트이므로 "팀 성과"를 분리할 근거 자체가 아직 없음 — 이 점을 그대로 명시
4. **검증 못한 주장과 보완 계획**: 동시 부하 상태의 응답 시간, 실사용 트래픽 패턴 대표성 — 2주차 k6 부하 테스트로 보완 예정
