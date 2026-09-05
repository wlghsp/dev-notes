# 2주차 실행 가이드

week2/prep-questions.md로 학습을 마친 뒤 실제로 진행하는 순서. 대상 저장소는 dingco/challenge-backend-resume-2026-08-wlghsp-r17(studypass), 1주차에 고른 API(GET /api/studies) 그대로 이어감.

각 단계마다 실행 명령과 결과를 그대로 붙여넣는 빈 칸을 뒀다. 결과는 코드블록 안에 그대로 붙여넣으면 된다.

---

## 1단계: 브랜치 생성

홈페이지에서 `submit/week-02__weekly-pr` 형태의 브랜치를 생성한다.

- [ ] 생성 완료

## 2단계: docker compose로 Prometheus·Grafana 띄우기

```bash
docker compose up -d   # db, redis, prometheus, grafana 전부
gradle bootRun
```

### 2-1. 컨테이너 상태 (`docker compose ps`)

```
jihochoi@Jiho-MacBook-Pro challenge-backend-resume-2026-08-wlghsp-r17 % docker ps
CONTAINER ID   IMAGE                            COMMAND                  CREATED        STATUS                PORTS                                                    NAMES
a0b0f399033a   grafana/grafana:11.1.0           "/run.sh"                6 days ago     Up 6 days             0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp              challenge-backend-resume-2026-08-wlghsp-r17-grafana-1
572c1bdde79b   redis:7                          "docker-entrypoint.s…"   6 days ago     Up 6 days             0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp              challenge-backend-resume-2026-08-wlghsp-r17-redis-1
15d62d0f4b07   prom/prometheus:v2.53.0          "/bin/prometheus --c…"   6 days ago     Up 6 days             0.0.0.0:9090->9090/tcp, [::]:9090->9090/tcp              challenge-backend-resume-2026-08-wlghsp-r17-prometheus-1
0e612a8c7238   mysql:8.0                        "docker-entrypoint.s…"   6 days ago     Up 6 days (healthy)   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp, 33060/tcp   challenge-backend-resume-2026-08-wlghsp-r17-db-1
54769e4ebfc1   ghcr.io/k3d-io/k3d-proxy:5.9.0   "/bin/sh -c nginx-pr…"   2 months ago   Up 2 weeks            80/tcp, 127.0.0.1:6443->6443/tcp                         k3d-etude-serverlb
3569b41d3f52   rancher/k3s:v1.35.5-k3s1         "/bin/k3d-entrypoint…"   2 months ago   Up 2 weeks                                                                     k3d-etude-server-0
```

### 2-2. Prometheus 타겟 상태 (`http://localhost:9090/targets`에서 studypass job이 UP인지)

```
UP
```

### 2-3. `/actuator/prometheus` 지표 확인

```bash
curl -s http://localhost:8080/actuator/prometheus | grep -E "^(system_cpu_usage|process_cpu_usage|jvm_memory_used_bytes|jvm_memory_max_bytes)\{"
```

```
jvm_memory_max_bytes{application="studypass",area="heap",id="G1 Eden Space"} -1.0
jvm_memory_max_bytes{application="studypass",area="heap",id="G1 Old Gen"} 4.294967296E9
jvm_memory_max_bytes{application="studypass",area="heap",id="G1 Survivor Space"} -1.0
jvm_memory_max_bytes{application="studypass",area="nonheap",id="CodeCache"} 5.0331648E7
jvm_memory_max_bytes{application="studypass",area="nonheap",id="Compressed Class Space"} 1.073741824E9
jvm_memory_max_bytes{application="studypass",area="nonheap",id="Metaspace"} -1.0
jvm_memory_used_bytes{application="studypass",area="heap",id="G1 Eden Space"} 1.048576E7
jvm_memory_used_bytes{application="studypass",area="heap",id="G1 Old Gen"} 4.1678336E7
jvm_memory_used_bytes{application="studypass",area="heap",id="G1 Survivor Space"} 6524320.0
jvm_memory_used_bytes{application="studypass",area="nonheap",id="CodeCache"} 1.4516352E7
jvm_memory_used_bytes{application="studypass",area="nonheap",id="Compressed Class Space"} 1.4099664E7
jvm_memory_used_bytes{application="studypass",area="nonheap",id="Metaspace"} 9.5221672E7
process_cpu_usage{application="studypass"} 8.195862267121522E-4
system_cpu_usage{application="studypass"} 0.2
```

### 2-4. Grafana에서 그래프 확인

```
JVM Statistics 대시보드에서 부하 없는 상태 (Uptime 8.7s) 기준값 확인:
- Heap Used: 1.7%
- Non-Heap Used: 11.0%
- System CPU Usage: Mean 0.343, Last 0.300 (Max 0.594 / Min 0.190)
- Process CPU Usage: Mean 0.00949, Last 0.00357 (Max 0.121 / Min 0.000854)
- Load Average [1m]: Mean 5.81 Last 5.71 (Max 10.7 / 3.32)
- CPU Core Size: 8
```

### 2-5. 부하 전 기준값 (4단계 비교용)

- system_cpu_usage: 0.343 (mean) / 0.300 (last)
- process_cpu_usage: 0.00949 (mean) / 0.00357 (last)
- jvm_memory_used_bytes (heap):1.7%
- jvm_memory_max_bytes (heap): 4.294967296E9 (G1 Old Gen 기준, Eden/Survivor는 동적 크기라 -1)

## 3단계: k6로 기준선 측정

```bash
k6 run k6-scripts/baseline.js
```

### 3-1. 측정 조건

- 시드 규모(1주차와 동일해야 비교 가능): members 2000 / studies 300 / enrollments 200000 / commentsPerStudy 30 (1주차와 동일, 확인 필요 시 재검증)
- 실행 시각: 2026-09-03 14:16:44 KST

### 3-2. 실행 결과 (요약 표 전체)

```
         /\      Grafana   /‾‾/
    /\  /  \     |\  __   /  /
   /  \/    \    | |/ /  /   ‾‾\
  /          \   |   (  |  (‾)  |
 / __________ \  |_|\_\  \_____/

     execution: local
        script: k6-scripts/baseline.js
        output: -

     scenarios: (100.00%) 1 scenario, 30 max VUs, 2m30s max duration (incl. graceful stop):
              * default: Up to 30 looping VUs for 2m0s over 3 stages (gracefulRampDown: 30s, gracefulStop: 30s)

  █ THRESHOLDS

    http_req_failed
    ✓ 'rate<0.05' rate=0.00%

  █ TOTAL RESULTS

    checks_total.......: 3502    29.175457/s
    checks_succeeded...: 100.00% 3502 out of 3502
    checks_failed......: 0.00%   0 out of 3502

    ✓ studies 200
    ✓ popular 200

    HTTP
    http_req_duration..............: avg=14.09ms min=751µs med=8.11ms max=396.06ms p(90)=30.17ms p(95)=36.56ms
      { expected_response:true }...: avg=14.09ms min=751µs med=8.11ms max=396.06ms p(90)=30.17ms p(95)=36.56ms
    http_req_failed................: 0.00%  0 out of 3502
    http_reqs......................: 3502   29.175457/s

    EXECUTION
    iteration_duration.............: avg=1.02s   min=1s    med=1.02s  max=1.4s     p(90)=1.04s   p(95)=1.06s
    iterations.....................: 1751   14.587729/s
    vus............................: 1      min=1         max=30
    vus_max........................: 30     min=30        max=30

    NETWORK
    data_received..................: 5.9 MB 49 kB/s
    data_sent......................: 324 kB 2.7 kB/s

running (2m00.0s), 00/30 VUs, 1751 complete and 0 interrupted iterations
default ✓ [======================================] 00/30 VUs  2m0s
```

### 3-3. 핵심 수치만 정리

- TPS (`http_reqs`): 29.18/s(총 3502건/ 2분)
- 응답 시간 평균: 14.09ms
- 응답 시간 p90: 30.17ms
- 응답 시간 p95: 36.56ms
- 응답 시간 p99: 55.35ms
- `http_req_failed` 비율: 0.0%

p99까지 필요하면 아래 명령으로 재실행:

```bash
k6 run --summary-trend-stats="avg,min,med,max,p(90),p(95),p(99)" k6-scripts/baseline.js
```

## 4단계: 병목 자원 판단

### 4-1. 부하 중/후 지표 (Grafana에서 관찰, 2-5의 부하 전 기준값과 비교)

- system_cpu_usage (부하 중): Max 0.655 (부하 전 mean 대비 0.343 대비 약 1.9배 상승)
- process_cpu_usage (부하 중): Max 0.189 (부하 전 mean 대비 0.00949 대비 약 20배 상승)
- jvm_memory_used_bytes (부하 중): 1.4% (부하 전 1.7%와 큰 차이 없음)

### 4-2. MySQL 커넥션 풀 상태

```bash
# MySQL 컨테이너 접속 후
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

```
mysql> SHOW STATUS LIKE 'Threads_connected';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_connected | 11    |
+-------------------+-------+
1 row in set (0.05 sec)

mysql> SHOW STATUS LIKE 'Max_used_connections';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| Max_used_connections | 12    |
+----------------------+-------+
1 row in set (0.00 sec)
```

### 4-3. 병목 판단

- 어느 자원이 먼저 한계에 닿았는가: CPU가 가장 먼저 반응했다. 메모리와 MySQL 커넥션 풀은 이번 부하 수준(최대 30 VU)에서 한계에 닿지 않았다.
- 판단 근거(수치 비교):
  - CPU: 부하 전 mean 0.343 → 부하 중 max 0.655로 약 1.9배 상승 (Process CPU Usage는 mean 0.00949 → max 0.189로 약 20배 상승, 더 뚜렷한 변화)
  - 메모리(heap): 부하 전 1.7% → 부하 중 1.4%로 오히려 변화 없음(오차 범위) - 메모리는 이번 부하로는 압박받지 않음
  - 커넥션 풀: Threads_connected 11, Max_used_connections 12로 여유 있음 - 풀 고갈 징후 없음
- 1주차에 예측한 병목(count 쿼리 풀스캔 / OFFSET 페이징)과 실제로 일치하는가:
  - 부분적으로 일치한다. count 쿼리와 OFFSET 페이징 둘 다 조건절 없이 테이블을 훑는 연산이라 CPU를 많이 쓰는데, 실제로 CPU 사용률이 가장 먼저 상승한 것으로 이 예측이 방향은 맞다는 게 확인됐다. 다만 이번 부하(최대 30 VU)로는 CPU가 완전히 포화(100%)되지는 않았고, 정확히 어느 쿼리가 CPU 상승의 원인인지까지는 이번 측정만으로는 특정하지 못했다 — 이는 실행계획(EXPLAIN)까지 확인해야 하는 3주차 영역과 연결된다.

## 5단계: 개선 적용

- 선택한 개선 방법(1주차 예측 병목과 연결): OFFSET 페이징 → 커서 기반 페이징 전환. `GET /api/studies`가 `LIMIT ?, ?`로 앞부분을 읽고 버리던 구조를 `WHERE id > ? ORDER BY id LIMIT ?`로 바꿔 PK 인덱스를 직접 활용하도록 함(자세한 코드는 improvement-guide.md).
- 변경한 파일: `src/main/` — `StudyRepository`에 `findByIdGreaterThanOrderByIdAsc` 추가, `StudyController.list()`를 `Page<Study>` 대신 `cursor`/`nextCursor` 기반으로 변경
- 변경한 파일: `src/test/` — `listReturnsOkWithExpectedFields`를 `nextCursor` 검증으로 수정, `cursorReturnsNextPageAfterGivenId` 신규 추가
- 기존 회귀 테스트(`StudyControllerTest`) 통과 확인:

```
BUILD SUCCESSFUL in 13s
5 actionable tasks: 1 executed, 4 up-to-date
```

## 6단계: 같은 조건으로 재측정

```bash
k6 run k6-scripts/baseline.js
```

### 6-1. 실행 결과 (요약 표 전체)

```
     execution: local
        script: k6-scripts/baseline.js
        output: -

     scenarios: (100.00%) 1 scenario, 30 max VUs, 2m30s max duration (incl. graceful stop):
              * default: Up to 30 looping VUs for 2m0s over 3 stages (gracefulRampDown: 30s, gracefulStop: 30s)



  █ THRESHOLDS 

    http_req_failed
    ✓ 'rate<0.05' rate=0.00%


  █ TOTAL RESULTS 

    checks_total.......: 3556    29.427522/s
    checks_succeeded...: 100.00% 3556 out of 3556
    checks_failed......: 0.00%   0 out of 3556

    ✓ studies 200
    ✓ popular 200

    HTTP
    http_req_duration..............: avg=5.97ms min=1.21ms med=5.56ms max=73.62ms p(90)=9.53ms p(95)=11.34ms p(99)=14.97ms
      { expected_response:true }...: avg=5.97ms min=1.21ms med=5.56ms max=73.62ms p(90)=9.53ms p(95)=11.34ms p(99)=14.97ms
    http_req_failed................: 0.00%  0 out of 3556
    http_reqs......................: 3556   29.427522/s

    EXECUTION
    iteration_duration.............: avg=1.01s  min=1s     med=1.01s  max=1.08s   p(90)=1.01s  p(95)=1.02s   p(99)=1.02s  
    iterations.....................: 1778   14.713761/s
    vus............................: 1      min=1         max=30
    vus_max........................: 30     min=30        max=30

    NETWORK
    data_received..................: 5.9 MB 49 kB/s
    data_sent......................: 331 kB 2.7 kB/s




running (2m00.8s), 00/30 VUs, 1778 complete and 0 interrupted iterations
default ✓ [======================================] 00/30 VUs  2m0s
```

### 6-2. 개선 전후 비교

| 지표 | 개선 전 | 개선 후 |
|---|---|---|
| TPS | 29.18/s | 29.43/s |
| 응답 시간 평균 | 14.09ms | 5.97ms |
| 응답 시간 p95 | 36.56ms | 11.34ms |
| 응답 시간 p99 | 55.35ms | 14.97ms |

참고: 6단계를 두 번 실행했는데 첫 실행은 max 540ms/p99 59.85ms로 편차가 컸고, 재실행은 max 73.62ms/p99 14.97ms로 안정적이었다. 서버를 막 띄운 직후(cold start)라 JIT 컴파일과 HikariCP 커넥션 풀 워밍업 비용이 첫 실행에 섞여 들어간 것으로 추정 — 위 표는 재실행(워밍업 이후) 값을 사용함.

## 7단계: resume.md 업데이트

- [ ] 병목 판단 근거를 resume.md에 기록
- [ ] 1주차 문장 1의 "한계"(동시 요청 부하 응답 시간 확인 불가)를 이번 주차 실측값으로 교체
- [ ] 새 문장(문장 2)으로 이번 주차 개선 결과를 수치화 패턴으로 작성

## 8단계: evidence 파일 작성

`evidence/week-02__weekly-pr.md`에 missions/README.md 형식대로 작성. 위 1~7단계에서 채운 내용을 그대로 옮기면 됨.

- [ ] Prometheus·Grafana 수집 지표 수치
- [ ] k6 기준선 TPS·응답 시간 분포
- [ ] 개선 전후 비교(같은 스크립트·같은 조건)
- [ ] 병목 판단 근거
- [ ] 근거형 질문 1~3 답변

## 9단계: PR 생성 및 리뷰 반영 (1주차/JPA week3에서 놓쳤던 부분!)

- [ ] resume.md와 src/main, src/test를 같은 PR에 함께 포함
- [ ] **PR 제출 후 자동 리뷰(또는 self-check)가 오면 즉시 evidence의 "## 리뷰 반영"에 기록** — 지적 없으면 "지적 없음", 있으면 무엇을 어떻게 고쳤는지
- [ ] self-check 실패 시 `gh run view <run-id> --job <job-id> --log`로 원인 확인. "채점 Workflow/challenge.json 변경 금지" 에러면 `git merge origin/main` 후 재push

---

## 근거형 질문 3개 — 답변 방향 메모

1. **사용자 400명 이상 증가 시 문제**: 이번 주차 k6에서 VU를 어디까지 올렸는지, 그 시점에 먼저 한계에 닿은 자원이 무엇이었는지 기준. 400이라는 숫자 자체보다 "측정한 임계점 대비 몇 배인가"로 추론
2. **추가 가능한 기능과 필요한 변화**: studypass가 실제로 뭘 하는 서비스인지 정리 후, 이번 주차 겪은 변화 패턴(모니터링 추가, src/main+src/test 동시 변경)을 참고해 새 기능이 어떤 계층에 영향 주는지 짚기
3. **예상보다 많은 사용자 유입 시 공부할 것**: 이번 주차 병목 판단에서 드러난 한계를 먼저 정리하고, 그 한계 해결에 필요한 지식 영역을 매칭(커넥션 풀/스레드 모델, 인덱싱/쿼리 최적화, 캐싱 전략 등)
