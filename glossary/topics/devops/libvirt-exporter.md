# Libvirt Exporter

Libvirt Exporter는 libvirt가 관리하는 VM(도메인)의 CPU, 메모리, 디스크 I/O, 네트워크 트래픽 같은 메트릭을 Prometheus가 스크랩할 수 있는 형식으로 노출하는 익스포터다. Node Exporter가 호스트 OS 레벨 메트릭을 노출하듯, Libvirt Exporter는 그 위에서 도는 VM 단위 메트릭을 노출한다. 참고: qemu.md

```mermaid
graph LR
    Libvirtd["libvirtd (VM 관리)"]
    Exporter["Libvirt Exporter"]
    Prom["Prometheus"]
    Grafana["Grafana"]
    Libvirtd -->|libvirt API로 조회| Exporter
    Exporter -->|/metrics 노출| Prom
    Prom -->|시각화| Grafana
```

## 왜 필요한가

libvirt/KVM 자체는 `virsh` 같은 CLI로 개별 VM 상태를 조회할 수 있지만, 시계열로 쌓아서 추이를 보거나 임계치를 넘으면 알림을 보내는 체계는 없다. Libvirt Exporter는 libvirtd가 가진 정보를 주기적으로 읽어 HTTP 엔드포인트(보통 `/metrics`)로 노출하고, Prometheus가 이를 스크랩해 시계열로 저장한다. 이후 Grafana로 대시보드를 구성하거나 Alertmanager로 알림을 걸 수 있다.

## 수집 대상

VM 단위의 CPU 사용 시간, 메모리 사용량, 디스크 read/write, 네트워크 송수신 바이트 등을 수집한다. 호스트 자체의 메트릭이 아니라 libvirt가 관리하는 각 도메인(VM)의 메트릭이라는 점이 Node Exporter와의 차이다. 순수 KVM 호스팅 환경이나 KubeVirt처럼 libvirt를 통해 VM을 다루는 환경에서 VM 단위 모니터링에 쓰인다.

참고: qemu.md
