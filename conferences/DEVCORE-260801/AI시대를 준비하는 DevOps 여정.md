# AI시대를 준비하는 DevOps 여정


인프라 계층 전체의 복제가 필요

IaC (Infrastructure as Code)
terraform - 

Pulumi - 실제 프로그래밍 언어로 인프라를 선언하는 IaC 도구 (참고: pulumi.md)

why Pulumi

선언형 언어가 아니라서 IDE 활용, 추상화된 설계 가능

가장 훌륭한 문서 품질

(TS) ESLint/Prettier 모노 레포 등으로 높은 품질 관리
(TS) Jest를 활용한 단위 테스트

쿠버네티스 

GitOps + Argo CD (참고: gitops.md, argocd.md)

원하는 상태를 Git에 쓰면 시스템이 거기에 맞춘다
- Git이 유일한 진실

GitOps
- 원하는 운영 상태를 선언하는 곳
- merge가 배포, revert가 롤백
- 시스템이 실제를 Git에 맞춘다.

Argo CD
- 클러스터가 Git을 보고 당긴다(pull)
- 선언과 실제를 비교, 다르면 교정
- 배포 기록이 곧 Git 히스토리

### 쿠키 기반 라우팅

- 기존 도메인 그대로
- 앱 수정 없이 인프라에서 분기

Linkerd (서비스 메시)

### PR을 열면, 그 변경만 반영된 환경 생성


Qodana 정적분석

### 모든 비즈니스 데이터를 빅쿼리로


Claude Code -> envoy Gateway(토큰 단위 집계) -> Anthropic API -> 429시 BedRock

AI Token Usage 