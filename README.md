# DeviceNet PIT Time 계산기

Hilscher CIF50-DNM 환경에서 **Explicit Message 안정화**를 위한 Poll Interval Time(PIT) 최적값을 산출하는 웹 기반 계산 도구입니다.

> 제작: TES CO., LTD. S/W1 Team

---

## 실행 방법

별도의 설치나 빌드 과정 없이 `DeviceNet_PIT_Calculator.html` 파일을 웹 브라우저에서 직접 열면 됩니다.

---

## 사용 방법

### ① 네트워크 기본 설정

| 항목 | 설명 | SyConDN 위치 |
|------|------|-------------|
| Baud Rate | 통신 속도 (500 / 250 / 125 kbps) | Master → Bus Parameters → Baud Rate |
| EPR | Expected Packet Rate (기본값: 75) | Slave → Connection → EPR |
| 안전 여유율 | 권장 250% | — |

### ② 노드 정보 입력

| 항목 | 설명 | SyConDN 위치 |
|------|------|-------------|
| MAC ID | 슬레이브 장치 주소 (1~63) | Slave → Device Configuration |
| Input (bytes) | 슬레이브 → 마스터 데이터 크기 | Slave → I/O → Input Size |
| Output (bytes) | 마스터 → 슬레이브 데이터 크기 | Slave → I/O → Output Size |

프리셋 버튼으로 자주 사용하는 장치를 빠르게 추가할 수 있습니다.

| 프리셋 | Input | Output |
|--------|-------|--------|
| MFC | 6 bytes | 6 bytes |
| EPC | 8 bytes | 8 bytes |
| I/O 모듈 | 4 bytes | 4 bytes |

### ③ 계산 결과 확인

노드를 추가하면 결과가 자동으로 계산됩니다. 산출된 **권장 PIT 설정값**을 SyConDN의 `Master → Bus Parameters → Poll Interval Time`에 입력하십시오.

---

## 판정 기준

| 판정 | 조건 | 조치 |
|------|------|------|
| 녹색 | 권장 PIT < EPR 타임아웃 × 70% | 안전. 그대로 적용 가능 |
| 노란색 | 권장 PIT < EPR 타임아웃 | EPR 상향 또는 노드 수·I/O 크기 축소 권장 |
| 빨간색 | 권장 PIT ≥ EPR 타임아웃 | 통신 속도 업그레이드 또는 노드 분산 필요 |
| 노란색 행 (표) | I/O 크기 > 8 bytes | 프래그멘테이션 발생. I/O 크기 최적화 권장 |

> EPR 타임아웃 상한선 = EPR × 4 ms

---

## 계산 원리

### CAN 프레임 전송 시간

```
프레임 비트 수 = 47 (고정 오버헤드) + bytes × 8
비트 스터핑 보정 = 프레임 비트 수 × 1.2
전송 시간(ms) = 보정 비트 수 / (Baud Rate × 1000) × 1000
```

- 고정 오버헤드 47비트: SOF, ID, RTR, IDE, DLC, CRC, ACK, EOF, IFS 등
- 비트 스터핑: 동일 극성 5비트 연속 시 반대 극성 1비트 삽입 → 최대 20% 증가

### 권장 PIT 산출

```
총 Poll 점유시간 = 모든 노드의 전송 시간 합산
권장 PIT = ceil(총 Poll 점유시간 × 안전 여유율)
```

### 프래그멘테이션

CAN 프레임의 최대 페이로드는 8 bytes입니다. I/O 크기가 8 bytes를 초과하면 여러 프레임으로 분할 전송되며, 각 분할 프레임에 1 byte의 프래그멘테이션 헤더가 추가되어 버스 부하가 증가합니다.

---

## 주의사항

- 계산 결과는 **이론값**입니다. 케이블 품질, 노이즈 등 실제 환경에 따라 차이가 발생할 수 있습니다.
- 설정 변경 후 최소 **24시간 이상** 모니터링을 권장합니다.

---

## 파일 구성

| 파일 | 설명 |
|------|------|
| `DeviceNet_PIT_Calculator.html` | 계산기 본체 (HTML + CSS + JavaScript 단일 파일) |
| `DeviceNet_PIT_Calculator_Manual.docx` | 사용자 매뉴얼 |
