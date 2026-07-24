# virt-launcher

virt-launcher는 KubeVirt가 VM 하나당 만드는 특수한 Pod다. 이름 그대로 "virt(virtualization)를 launch(실행)하는 것"이라는 뜻이다. 참고: kubevirt.md

## 생명주기: VM과 1:1로 존재한다

virt-launcher Pod는 상시 떠 있는 인프라가 아니라, VM의 실행 상태를 그대로 따라간다. VM을 켜는(실행 요청하는) 시점에 KubeVirt 컨트롤러가 kubelet에게 지시해서 그 시점에 새로 생성되고, VM이 켜져 있는 동안 계속 떠 있는다. VM을 끄면 virt-launcher Pod도 같이 사라지고, 나중에 다시 켜면 새 Pod가 또 생성된다.

그래서 virt-launcher는 "VM이 살아있는 동안만 존재하는, VM 전용 실행 그릇"이다. 일반 컨테이너 Pod가 디플로이먼트마다 매번 새로 뜨는 것과 같은 개념으로, VM 생명주기와 Pod 존재 여부가 1:1로 묶여 있다.

## 내부 구성

이 Pod 안에는 `libvirtd`(가상화 관리 데몬)와 QEMU 프로세스가 함께 들어있고, 이 QEMU가 `/dev/kvm`을 통해 KVM에 접근해 실제 VM을 하드웨어 가속으로 실행한다. 참고: kvm.md, qemu.md

## 컨테이너 포장지로서의 virt-launcher

virt-launcher는 "VM을 감싸는 컨테이너 포장지"라고 보면 된다. VM 자체가 컨테이너 기술로 바뀌는 게 아니라, 기존 QEMU/KVM 방식 그대로 VM을 돌리되 그 QEMU 프로세스를 쿠버네티스가 인식할 수 있는 Pod 틀 안에 가둬놓은 것이다. 그래서 쿠버네티스 입장에서는 "Pod 하나가 떠 있다"로만 보이고 일반 Pod처럼 스케줄링·재시작 대상이 되지만, 내부적으로는 그 Pod가 VM을 돌리고 있다는 특수성이 있다.

참고: kubevirt.md, kvm.md, qemu.md, pod.md, container-vs-vm.md
