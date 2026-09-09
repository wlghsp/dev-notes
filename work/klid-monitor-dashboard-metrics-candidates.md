# 신규 대시보드 표출 항목 후보

대상 레포: klid_monitor
작성일: 2026-09-09
관련 문서: klid-monitor-realtime-dashboard-proposal.md (전체 기획안)

이 문서는 기획안의 "표출 항목" 섹션을 확장하기 위해 klid_monitor 코드베이스를 다시 조사한 결과만 모았다. 이미 기획안에 있는 컴퓨트/K8s/네트워크/스토리지 4개 카테고리는 다루지 않고, 지금까지 언급되지 않은 새 후보만 정리한다.

각 항목은 근거 확실성에 따라 세 등급으로 나눴다.
- 즉시 표출 가능: 지금 있는 API/테이블을 그대로 조회해서 화면에 올릴 수 있는 항목
- 추가 개발 필요: 데이터는 있지만 조회 API가 없거나 추가 가공이 필요한 항목
- 근거 부족: 코드에 일부 흔적은 있지만 확정하기엔 이르다고 판단한 항목

## 즉시 표출 가능

### 관리자 활동 현황 (Audit)
로그인/로그아웃, 메뉴 이동, 생성/수정/삭제, 엑셀 다운로드 이벤트를 구분코드(SE)별로 집계해서 시간대별 활동량 추이나 사용자별 활동 랭킹으로 보여줄 수 있다. 운영 대시보드에 "지금 누가 무엇을 하고 있는가"를 더하는 항목이다.
- 근거: `DashboardAuditLogServiceImpl.java` (SE_CREATE/UPDATE/DELETE/LOGIN/MENU/EXCEL 구분), `DashboardAuditRestController.java`, DB 테이블 `TADP_AUDIT_LOG`(SE, PRJCT_NM, TEN_NM, CN, USER_ID, CREAT_DT)
- 표현 형태: 시간대별 활동량은 막대그래프, 사용자별 활동 랭킹은 순위 리스트(막대 길이로 상대량 표시)

### 로그인 세션 통계
활성 세션 수, 최종 접속 시각 분포를 보여주는 위젯. 동시 접속자 수 추이는 실시간 대시보드의 상단 KPI로 붙이기 좋다.
- 근거: DB 테이블 `TADP_LOGIN_TOKEN_MGNT`(USER_ID, SESN_ID, CREAT_DT), `TADP_USER_CONEC_INFO`(LAST_CONEC_BEGIN_DT, LAST_CONEC_END_DT), 컨트롤러 `DashboardLoginRestController.java`
- 표현 형태: 숫자 하나를 크게 보여주는 스탯 카드(현재 접속자 수), 그 아래 최근 접속 이력은 시각 순 리스트

### K8s 이벤트 타임라인 (CrashLoopBackOff 외 전체)
기존 기획안엔 CrashLoopBackOff만 있었는데, 오브젝트 구분(OBJECT_SE)·유형(TY)별로 전체 K8s 이벤트를 필터링하면 클러스터 전체 이벤트 발생 빈도 추이로 확장할 수 있다.
- 근거: DB 테이블 `SCE_K8S_EV`(CLUSTER_ID, OBJECT_SE, OBJECT_NM, TY, SE, SMRY, DC, CREAT_DT)
- 표현 형태: 시간축을 따라 이벤트 점을 찍는 타임라인(유형별 색상 구분), 클릭 시 SMRY 텍스트 상세 표시

### 자원 사용량 히스토리 (일/시간/월)
지금 기획안의 컴퓨트 지표는 실시간 추이 중심인데, 별도로 이미 일/시간/월 단위 집계 테이블이 존재한다. "최근 1시간" 그래프 옆에 "최근 30일 추이"를 나란히 붙이는 장기 추세 화면에 바로 쓸 수 있다.
- 근거: DB 테이블 `TADP_STATS_VM_USGQTY_DAILY`, `TADP_STATS_VM_USGQTY_HBH`(시간별), `TADP_STATS_VM_USGQTY_MTM`(월별), `TADP_VM_RESOURCE_HISTORY`, `TDAP_VM_HISTORY_DAY/HOUR/MONTH`
- 표현 형태: 실시간 라인차트 옆에 나란히 배치하는 별도 라인차트(기간 선택 탭: 일/주/월)

### 인터페이스(외부 연동) 처리 이력
송신/수신 시각차 기반 처리 지연, 처리 결과별(성공/실패) 건수 추이. 대시보드가 다루는 범위를 "인프라 상태"에서 "연동 상태"까지 넓히는 항목이다.
- 근거: DB 테이블 `TADP_IF_HIST`(MSG_ID, IF_ID, SEND_DT, RECV_DT, PROCESS_RESULT_CN)
- 표현 형태: 연동 채널별 성공/실패 건수는 스택 막대그래프, 처리 지연(RECV_DT - SEND_DT)은 추이 라인차트

### 보안 정책 현황 요약
등록된 접속 허용 IP 개수, 사용 여부(USG_AT) 분포를 요약 카드로 보여줄 수 있다.
- 근거: `SecurityPolicyRestController.java`, DB 테이블 `TADP_SETUP_CONEC_ALLOW_IP`(IP, USG_AT, DEL_AT)
- 표현 형태: 등록 IP 수·활성 비율을 보여주는 요약 스탯 카드 1~2개 (그래프 불필요)

### 고객지원 처리 현황
미답변 FAQ/QnA 건수, 티켓 처리 상태 분포. 인프라 지표는 아니지만 "운영 현황판"으로서 대시보드의 성격을 넓힐 수 있는 항목이다.
- 근거: `FaqRestController.java`, `QnARestController.java`, `TicketController.java`, DB 테이블 `TADP_FAQ`, `TADP_QNA`, `GTP_TICKET_MGMT`/`GTP_TICKET_MGMT_HIST`
- 표현 형태: 미답변 건수는 강조색 숫자 배지, 처리 상태 분포는 도넛차트(접수/처리중/완료)

### 서비스 카탈로그/브로커 상태
등록된 PaaS 서비스 브로커·카탈로그 개수 및 상태 요약.
- 근거: `ServiceBrokerRestController.java`, `ServiceCatgalogRestController.java`
- 표현 형태: 브로커/카탈로그 목록 테이블 대신 상태별(정상/오류) 카드 그리드

### 자원 분류(Categorization) 현황
프론트엔드 라우터에 이미 활성화되어 있는데 대시보드 기획 문서에는 빠져 있던 화면이다. 플랫폼/조직별 자원 분류 체계를 그대로 신규 대시보드에 편입할 수 있다.
- 근거: `router.js`의 `platformCategorization`/`organizationCategorization` (활성 라우트), `CategorizationRestController.java`, DB 테이블 `SCE_ASSET_CL`
- 표현 형태: 분류 체계이므로 트리 구조 또는 계층형 리스트

## 추가 개발 필요

### Quartz 스케줄러 실행 현황
작업별 성공/실패(RESULT_CODE), 처리 시간(PROCESS_TIME) 추이, 최근 에러 메시지를 노출하는 화면. 배치·알람 스케줄러가 정상 동작하는지 한눈에 보여줄 수 있어서 "운영 안정성" 관점에서 발표 포인트가 된다.
- 데이터는 있음: DB 테이블 `TADP_SCHEDULER_HISTORY`(SCHEDULER_NAME, STEP, RESULT_CODE, ERROR_MSG, PROCESS_TIME), `TADP_SCHEDULER_INFO`
- 막힌 지점: `SchedulerRestController.java`는 현재 시작/일시정지/재개/삭제 같은 실행 제어 API만 있고, 이력을 조회하는 API가 없다. 조회 API를 새로 만들어야 화면에 올릴 수 있다.
- 표현 형태: 작업별 최근 실행 결과를 성공/실패 색상 점으로 나열한 상태 그리드, 처리 시간은 추이 라인차트

## 근거 부족 — 보류

### 시스템/컨테이너 로그 발생량 추이
`sym-log-syslog`, `sym-log-container` 인덱스에서 시간대별 로그 건수(호스트별/파드별) 카운트 자체는 가능하다. 다만 로그 모델(`Log.java`)에 `file.path`만 있고 severity/level 필드가 없어서, "에러율"까지 보여주려면 메시지 텍스트 패턴 매칭을 새로 붙여야 한다. 지금 코드 근거로 확정할 수 있는 건 "로그 발생 빈도"까지이고, "에러율"은 별도 검증이 필요하다.
- 근거: `SyslogVo.java`, `ContainerLogVo.java`, `LogMetricVo.java`, `PodLogMetricVo.java` (인덱스명 `sym-log-syslog`/`sym-log-container`)
- 표현 형태(발생 빈도만): 시간대별 로그 건수 막대그래프. 에러율 배지는 패턴 매칭 검증 전까지 보류

## 조사 후 제외한 항목

- **RabbitMQ 큐 상태**: 내부 메시지 버스로만 쓰이고, 큐 상태를 노출하는 API나 모델이 코드에 없다.
- **WebSocket 연결 수**: `WebSocketController.java`에 테스트용 엔드포인트 하나만 있고, 실제 연결 수를 집계하는 근거가 없다.
- **Platform/Organization Statistics 화면**: 라우터에 남아있지만 주석 처리되어 비활성 상태라 후보에서 제외한다.

---

참고: klid_monitor 코드베이스 재조사 기반. AdminApi, DashboardApi, _dbScheme, DashboardApp/frontend/src/config/router 참조.
