# declarative-model (선언형 모델)
참고: kubernetes.md, control-plane.md

---

"어떻게 해라"가 아니라 "이런 상태여야 한다"고 기술하는 방식이다. Kubernetes가 애플리케이션을 배포하는 방식이 바로 이것이다.

## 명령형 vs 선언형

명령형(imperative)은 절차를 직접 지시한다.
- "컨테이너 3개를 서버 A, B, C에 각각 실행해라"

선언형(declarative)은 원하는 상태를 기술한다.
- "이 애플리케이션은 3개의 인스턴스가 항상 실행 중이어야 한다"

Kubernetes에서는 YAML 또는 JSON으로 작성한 manifest 파일로 원하는 상태를 기술하고, API Server에 제출한다.

## Kubernetes가 선언형인 이유

원하는 상태를 제출하면 Kubernetes가 알아서 현재 상태와 비교해서 필요한 작업을 수행한다. 인스턴스 하나가 죽으면 자동으로 다시 띄운다. 노드가 장애나면 다른 노드에 재배치한다. 개발자가 직접 개입하지 않아도 된다.

선언형 모델의 핵심은 **desired state(원하는 상태)와 current state(현재 상태)의 지속적인 reconciliation(조정)**이다. Controllers가 이 역할을 담당한다. 참고: control-plane.md
