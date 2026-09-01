# 1주차 준비: 알아야 할 것들

week1-problem-and-measurement.md를 실제로 하기 전에 먼저 답할 수 있어야 하는 질문들. studypass 저장소(challenge-backend-resume-2026-08-wlghsp-r17) 코드를 근거로 채운다.

---

## 1. 프로젝트 실행

- `docker compose up -d`로 뜨는 서비스는 몇 개이고 각각 무슨 역할인가? (db, redis, prometheus, grafana)

4개 
db는 데이터 저장 및 조회 등
redis는 캐시
prometheus는 메트릭 수집
grafana는 메트릭 시각화

- 이번 주차 범위에서 실제로 필요한 서비스는 무엇이고, 나머지는 왜 지금 안 써도 되는가?

: GET api/studies만 사용. 나머지는 남은 주차 미션에서 사용 

- `gradle bootRun`을 처음 실행하면 왜 몇 분이 걸리는가? (SeedRunner, BatchSource가 하는 일)

: SeedRunner, BatchSource를 통해서 초기에 시드 데이터를 넣는 작업을 합니다.  1회만 실행하고 2번째부터는 조회를 해서 데이터가 있으면 실행되지 않습니다.

- 시드 데이터 규모는 어디서 조절하는가? (`application.yml`의 `app.seed.*`)

: application.yml 의 app.seed.*

- 컴파일에 쓰이는 JDK 버전이 로컬에 설치된 Gradle/JDK 버전과 다를 수 있는데, 왜 바이트코드가 강의와 같게 나오는가? (toolchain)

```
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}
```

: 
위의 build.gradle이 있기 때문에 동일하게 컴파일을 실행하게 됩니다.
이게 하는 일은 Gradle을 실행하는 JVM과 코드를 컴파일하는 JVM을 분리하는 겁니다. 로컬에 JDK25가 깔려있어도, toolchain이 지정되어 있으면 Gradle이 컴파일 시점에만 JDK17을 찾아서(없으면 자동 다운로드) 그걸로 javac를 돌립니다.
그래서 내 로컬 JDK가 뭐든 컴파일 결과물(바이트코드)은 항상 JDK17 기준으로 동일하게 나옵니다. Gradle 자체 실행은 최신 JDK로 해도 상관없습니다

## 2. 대상 API 고르기

- 미션이 예시로 든 `GET /api/studies` 목록 조회는 실제 코드에서 무슨 쿼리를 실행하는가? (`StudyController.list`, `PageRequest`)

```
    select
        s1_0.id,
        s1_0.capacity,
        s1_0.category,
        s1_0.created_at,
        s1_0.enrolled_count,
        s1_0.fee,
        s1_0.opened_at,
        s1_0.status,
        s1_0.title 
    from
        study s1_0 
    limit
        ?, ?
2026-08-28T10:26:21.632+09:00 TRACE 850 --- [nio-8080-exec-1] org.hibernate.orm.jdbc.bind              : binding parameter (1:INTEGER) <- [1036]
2026-08-28T10:26:21.633+09:00 TRACE 850 --- [nio-8080-exec-1] org.hibernate.orm.jdbc.bind              : binding parameter (2:INTEGER) <- [74]
2026-08-28T10:26:21.648+09:00 DEBUG 850 --- [nio-8080-exec-1] org.hibernate.SQL                        : 
    select
        count(s1_0.id) 
    from
        study s1_0
```

- 같은 컨트롤러에 있는 `GET /api/studies/popular`는 왜 6주차 캐싱 대상으로 남겨져 있는가? 1주차에 이 API를 고르면 안 되는 이유가 있는가, 아니면 골라도 되는가?

6주차라 고르지 않는게 좋을듯 

- "사용자가 늘면 무엇이 먼저 깨질지"를 예측하려면, 지금 이 API의 구현에서 어떤 부분을 봐야 하는가? (`study` 테이블은 schema.sql상 PK 외 인덱스가 없다, `findAll`이 쓰는 OFFSET 기반 페이징의 비용)

위와 같이 쿼리가 나가는건 확인했는데 인덱스 생성 필요없음 

- `study_enrollment` 테이블이 가장 크다고 README에 적혀 있는데, `studies` 목록 조회는 이 테이블과 관계가 있는가 없는가?

없습니다.

## 3. 응답 시간 측정

- "같은 입력으로 3회 이상 측정"이라고 했을 때, 어떤 값을 고정해야 "같은 입력"이 되는가? (page, size, 서버 워밍업 상태)

page, size, 서버 워밍업 상태 3개를 고정해야 합니다. 

- 측정 도구로 뭘 쓸 수 있는가? (`curl -w`, k6, Postman 등 — 이번 주차는 아직 k6 스크립트를 요구하지 않음, 2주차부터 k6 사용)

측정 도구로는 curl -w나 IntelliJ의 http 파일로 실행합니다. Postman은 필요 없습니다.

- 응답 시간 측정 시 콜드 스타트(첫 요청)를 포함해야 하는가, 제외해야 하는가? 왜 이 판단이 "재현 가능성"과 연결되는가?

콜드 스타트는 제외해야함. JIT 워밍업 안됨, 첫 커넥션 풀 생성. 첫 쿼리 플랜 캐싱등 일회성 비용이 섞이기 때문입니다. 이 비용은 매 요청마다 반복되는게 아니라 딱 한번만 발생하니, 이걸 측정에 포함하면 "3회 평균"이 실제 정상 상태의 응답 시간을 대표하지 못하게 됩니다.

콜드 스타트가 일회성 비용을 섞어 넣어서 측정값의 변동성을 키우고, 그 변동성이 재현성을 해친다는 인과 관계입니다.

- 측정 조건(표본 수, 환경)을 안 적으면 왜 완료 기준을 못 채우는가?

: 측정 조건이 없으면 다른 사람이 재현할 수 없기때문입니다.


## 4. 이력서 수치화 패턴

- resume/resume.md의 "방어 가능한 성과 문장" 항목들(본인 행동, 전후 조건과 결과, 저장소 안 근거, 한계)은 각각 무엇을 채워야 하는가?
- "저장소 안 경로로 연결"한다는 게 구체적으로 무슨 뜻인가? (커밋 diff 링크가 아니라 파일 경로 + 근거)
- 아직 측정하지 못한 값을 "확인 불가"로 표시하라는 규칙은 왜 있는가? 추정치를 쓰면 안 되는 이유는?
- 이 시점(1주차)에 "팀의 성과와 본인의 기여를 구분"하라는 질문에 답하려면, 지금 갖고 있는 근거가 무엇이어야 하는가?

## 5. 회귀 테스트

- `src/test/`에 이미 있는 `StudypassSmokeTest`는 무엇을 검사하는가? 이번 주차에 추가할 테스트와 뭐가 다른가?
- 테스트가 MySQL 없이도 돌아야 한다는 제약(H2 사용) 때문에, 고른 API의 테스트를 짤 때 주의해야 할 게 있는가?
- "이후 주차의 개선이 기능을 깨지 않는지 확인"하는 회귀 테스트라면, 지금 시점에 무엇을 고정해서 검증해야 하는가? (응답 필드, 상태 코드, 페이지네이션 동작)

## 6. 제출 형식

- 이번 주차 evidence 파일명은 무엇이어야 하는가? (`evidence/<브랜치와 같은 이름>.md`)

//
- evidence/README.md의 틀에서 "## 변경", "## 검증", "## 선택 근거", "## 근거형 질문", "## 리뷰 반영" 각 섹션에 1주차 기준으로 뭘 채워야 하는가?
- resume/resume.md와 src/를 "함께" 바꿔야 자동 검사를 통과한다는 규칙은 왜 있는가?
