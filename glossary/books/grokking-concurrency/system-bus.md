# System Bus

CPU와 다른 하드웨어 컴포넌트들을 연결하는 통신 경로다. 단일 버스가 아니라, 역할에 따라 여러 종류의 버스가 나뉜다.

## 왜 필요한가

CPU는 연산만 한다. 실제 데이터를 가져오거나 내보내려면 RAM, SSD, NIC 같은 장치와 통신해야 한다. 이 통신 경로가 버스다. CPU가 각 장치에 직접 전선을 꽂을 수 없으니, 표준화된 인터페이스로 묶은 것이다.

## 종류

버스는 연결 대상에 따라 다르다.

- 메모리 버스 (FSB / QPI / Infinity Fabric) — CPU ↔ RAM. 현대 CPU는 메모리 컨트롤러를 CPU 내부에 내장해서, 별도 칩 없이 이 버스로 RAM에 직접 접근한다.
- PCIe — CPU ↔ GPU, NVMe SSD, NIC. 현재 가장 범용적인 고속 버스.
- SATA — CPU ↔ HDD/SSD. PCIe보다 느리고, 구형 장치에 쓰인다.
- USB — CPU ↔ 외부 장치. 키보드, 마우스, 외장 드라이브 등.

## OS → Driver → 버스 → 하드웨어 흐름

OS는 하드웨어 세부사항을 모른다. Samsung NVMe SSD인지 WD SATA SSD인지, 어떤 명령어 셋을 써야 하는지 — 이건 드라이버가 안다.

```
OS (커널)
  → Device Driver (커널 내부 or 커널 모듈)
    → 시스템 버스 (PCIe, SATA, USB 등)
      → 하드웨어 (SSD, NIC, GPU 등)
```

- OS는 드라이버에게 추상화된 요청을 보낸다: "이 주소에서 4KB 읽어와"
- 드라이버는 그 요청을 해당 하드웨어 전용 명령어로 변환해서 버스에 전송한다
- 하드웨어가 응답하면 드라이버가 받아 OS로 전달한다

드라이버는 OS와 하드웨어 사이의 번역 레이어다. OS가 하드웨어 종류에 상관없이 동일한 인터페이스로 요청할 수 있게 해준다.

## RAM은 특수하다

RAM은 드라이버 없이 CPU가 메모리 컨트롤러를 통해 직접 접근한다. load/store 같은 CPU 명령어가 바로 메모리 버스로 이어진다. 다른 장치와 달리 OS가 드라이버를 거치지 않는다.

## 참고

- operating-system.md — User Space / Kernel Space, syscall 흐름
