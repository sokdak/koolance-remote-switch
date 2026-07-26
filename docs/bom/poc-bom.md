# Koolance 무선 제어 베드 PoC 주문 목록

- 작성일: 2026-07-26
- 대상 장비: Koolance EXC-900 또는 VLX-2000-110V 중 한 모델 선택 지원
- 기준 설계: [2026-07-24-koolance-wireless-controller-design.md](../superpowers/specs/2026-07-24-koolance-wireless-controller-design.md)
- 견적 제출용 간략 BOM: [poc-rfq-bom.md](poc-rfq-bom.md) / [poc-rfq-bom.csv](poc-rfq-bom.csv)
- PoC 범위: 저전압 전체 기능, 실제 VLX 소프트 전원 버튼, EXC 12 V 더미 부하

## 1. 이번 PoC에서 확인할 것

이 주문 목록은 다음 경로를 실제 하드웨어로 검증하기 위한 것이다.

```text
PC PWR_BTN/PWR_LED
        │
        ▼
ESP32-S3 LCD 제어기 ))) 전용 2.4 GHz Wi-Fi ((( Pi Zero 2 W
        │                                           │
  버튼·LCD·버저                              USB 절연기 ─ KSM
                                                    │
                                             Pico 2 H Supervisor
                                              ├─ VLX PhotoMOS
                                              └─ EXC 12 V 더미 릴레이
```

확인 범위에는 LCD 상태 표시와 설정, 자동 페어링, KSM 읽기/쓰기, PC 기동 인터록, PWR_LED의 ON/OFF/절전 점멸 판정, 버저 경보, 후행 운전, VLX 소프트 버튼 병렬 구동이 포함된다. PC 상태는 PSU 출력이 아니라 `PWR_LED`로만 감지하며, PC 전원 명령은 메인보드 `PWR_SW`에 무전압 접점으로만 전달한다.

EXC의 실제 AC 입력과 모터 부하는 이번 PoC에 연결하지 않는다. EXC 프로파일의 논리와 Supervisor 출력은 5 V 릴레이와 12 V LED로 모사한다.

## 2. 바로 주문할 핵심 부품

아래 수량은 같은 부품을 전혀 보유하지 않았다고 가정한 구매 수량이다. `사용/예비` 열은 실제 장착 수량과 고장·배선 실수에 대비한 여분을 구분한다.

### 2.1 연산·표시·통신

| 주문 | 품목 및 검색어 | 구매 수량 | 사용/예비 | 반드시 확인할 조건 | 용도 |
|---|---|---:|---:|---|---|
| [ ] | `Waveshare ESP32-S3-Touch-LCD-3.5` | 1 | 1/0 | ESP32-S3R8, 8 MB PSRAM, 16 MB Flash, 3.5인치 320×480, 정전식 터치 모델 | PC 측 LCD 제어기 |
| [ ] | `Raspberry Pi Zero 2 W` | 1 | 1/0 | Zero W가 아닌 **Zero 2 W**, 2.4 GHz Wi-Fi 내장 | 칠러 측 게이트웨이 |
| [ ] | `Raspberry Pi Pico 2 H` | 1 | 1/0 | 헤더 납땜 완료된 **H** 모델 | PoC Supervisor MCU |
| [ ] | `High Endurance microSD 32GB` | 2 | 1/1 | 산업용 또는 블랙박스용 고내구성, A1 이상 | Pi OS와 복구 예비 카드 |
| [ ] | `USB microSD reader` | 1 | 1/0 | 보유 중이면 생략 | Pi 이미지 기록 |

> LCD 보드는 이름이 유사한 3.5인치 제품이 많다. 일반 ESP32나 ESP32-3248S035가 아니라 위의 Waveshare ESP32-S3 모델을 주문한다.

### 2.2 센서·사용자 입력·알림

| 주문 | 품목 및 검색어 | 구매 수량 | 사용/예비 | 반드시 확인할 조건 | 용도 |
|---|---|---:|---:|---|---|
| [ ] | `SHT45 I2C breakout 3.3V` | 1 | 1/0 | SHT40/가품 대신 SHT45, 3.3 V 동작, 풀업 저항 포함 모듈 | 주변 온도·습도와 이슬점 계산 |
| [ ] | `MCP23017 I2C GPIO expander module` | 1 | 1/0 | MCP23017, 3.3 V 사용 가능, 주소 점퍼 제공 | 버튼·엔코더 GPIO 확장 |
| [ ] | `EC11 rotary encoder push switch knob` | 2 | 2/0 | 푸시 스위치 포함, 손잡이 포함 | 메뉴 이동과 값 변경 |
| [ ] | `12mm momentary push button NO` | 6 | 6/0 | 자동 복귀, 순간동작, NO 접점 | PC POWER, BACK, MUTE, 메뉴 기능키 |
| [ ] | `5V active buzzer module low level trigger` | 2 | 2/0 | 발진 회로가 있는 **능동형**, 트랜지스터 입력 모듈 권장 | PC 측 경보 및 칠러 측 서비스 알림 |
| [ ] | `5mm LED red green yellow assortment` | 1세트 | 필요량/여분 | 개별 LED와 220 Ω~1 kΩ 저항 사용 | 전원·링크·경보 상태 표시 |

버튼 6개와 엔코더 푸시 2개를 함께 사용한다. 최종 조작 매핑은 펌웨어에서 정하므로 PoC 버튼은 색이 다른 동일 규격 제품으로 맞추는 편이 배선과 교체에 유리하다.

### 2.3 절연 I/O와 PC·칠러 인터페이스

| 주문 | 품목 및 검색어 | 구매 수량 | 사용/예비 | 반드시 확인할 조건 | 용도 |
|---|---|---:|---:|---|---|
| [ ] | `ADuM3160 USB isolator full speed 12Mbps isolated power` | 2 | 1/1 | 12 Mbps Full-Speed 지원, 절연 DC/DC 내장, host/device 방향 표시 | Pi와 KSM USB 사이 갈바닉 절연 |
| [ ] | `Raspberry Pi Zero OTG micro USB male to USB A female` | 2 | 1/1 | 충전 전용이 아닌 OTG 데이터 어댑터 | Pi USB Host 변환 |
| [ ] | `USB 2.0 A male to B male cable 1m` | 2 | 2/0 | 데이터 케이블, 가능하면 차폐형 | Pi OTG↔절연기 및 절연기↔칠러 |
| [ ] | `AQY212EH PhotoMOS DIP-4` | 5 | 2/3 | Panasonic AQY212EH 또는 동급, 기계식 릴레이/일반 옵토커플러와 혼동 금지 | PC PWR_SW와 VLX 버튼 무전압 병렬 접점 |
| [ ] | `DIP4 2.54mm breakout socket` | 5 | 2/3 | 브레드보드 장착 가능 | PhotoMOS 교체와 실험 배선 |
| [ ] | `LTV-814 AC input optocoupler DIP-4` | 4 | 1/3 | 역병렬 입력 LED형 LTV-814, LTV-817과 혼동 금지 | PWR_LED 극성 무관 감지 |
| [ ] | `2 pin PC front panel extension cable 2.54mm` | 4 | 4/0 | 암-수 조합, 개별 2핀 하우징 | PWR_BTN/PWR_LED 비파괴 경유 배선 |
| [ ] | `mini grabber test hook insulated` | 1세트 | 1/0 | 미세 피치용, 완전 절연 훅 | VLX 버튼 핀 식별과 임시 연결 |
| [ ] | `30AWG silicone wire wrapping wire` | 1세트 | 필요량/여분 | 여러 색상, 연선 권장 | VLX 저전압 버튼 하네스 |
| [ ] | `JST PH XH 1.25mm connector kit` | 각 1세트 | 필요량/여분 | PH 2.0 mm, XH 2.54 mm, 1.25 mm를 별도 구분 | 확인된 VLX 커넥터에 맞춘 분기 하네스 |

AQY212EH는 두 개만 실장한다. 한 개는 PC 메인보드 `PWR_SW`, 한 개는 VLX 전면 버튼에 쓴다. 나머지 세 개는 핀 식별 오류나 납땜 손상에 대비한 예비품이다. PhotoMOS 출력은 극성이 없지만 입력 LED에는 극성과 직렬 저항이 필요하다.

LTV-814의 입력 저항값은 메인보드별 `PWR_LED` 전압과 전류를 측정한 뒤 정한다. PoC에서 최초 측정 전 임의 저항으로 PC 헤더에 연결하지 않는다.

### 2.4 EXC 프로파일용 12 V 더미 부하

| 주문 | 품목 및 검색어 | 구매 수량 | 사용/예비 | 반드시 확인할 조건 | 용도 |
|---|---|---:|---:|---|---|
| [ ] | `Omron G5V-2 DC5 DPDT relay` | 2 | 1/1 | 코일 전압 **DC 5 V**, DPDT | EXC 컨택터 코일 동작 모사 |
| [ ] | `2N2222A TO-92 transistor` | 10 | 2/8 | 핀 배열은 제조사 데이터시트로 확인 | 릴레이·버저 로우사이드 구동 |
| [ ] | `1N4007 diode` | 10 | 2/8 | 정류 다이오드 | 릴레이 코일 플라이백 억제 |
| [ ] | `12V LED indicator module` | 2 | 1/1 | 직렬 저항 내장형 | 가상 EXC AC 부하 표시 |
| [ ] | `12V 1A DC adapter 5.5x2.1mm center positive` | 1 | 1/0 | 국내 안전인증 제품, 센터 플러스 | 더미 부하 독립 전원 |
| [ ] | `5.5x2.1mm DC jack screw terminal` | 1 | 1/0 | 어댑터 플러그와 규격 일치 | 납땜 없는 12 V 인입 |

릴레이 코일에는 플라이백 다이오드를 반드시 역방향 병렬 연결한다. Pico GPIO에서 코일을 직접 구동하지 않고 2N2222A와 베이스 저항을 사용한다.

### 2.5 전원·배선·제작 소모품

| 주문 | 품목 및 검색어 | 구매 수량 | 사용/예비 | 반드시 확인할 조건 | 용도 |
|---|---|---:|---:|---|---|
| [ ] | `5V 3A USB-C power adapter` + 데이터 USB-C 케이블 | 1세트 | 1/0 | LCD 보드 커넥터 확인, 안정적인 5 V 3 A | PC 측 제어기 독립 전원·개발 |
| [ ] | `Raspberry Pi 5V 3A micro USB power supply` | 1 | 1/0 | Pi Zero 전원 포트용, 스위치 없는 타입 권장 | 칠러 측 노드 독립 전원 |
| [ ] | `USB micro B data cable` | 1 | 1/0 | 충전 전용이 아닌 데이터 케이블 | Pico 2 H 펌웨어 기록 |
| [ ] | `830 point breadboard` | 3 | 3/0 | 전원 레일 분리형이면 표시 확인 | PC 측, 칠러 측, EXC 더미 분리 |
| [ ] | `Dupont jumper wire male female set` | M-M/M-F/F-F 각 1세트 | 필요량/여분 | 20 cm 전후 | 브레드보드 연결 |
| [ ] | `2.54mm pin header female header kit` | 1세트 | 필요량/여분 | 직선형 위주 | 모듈 장착 |
| [ ] | `2.54mm screw terminal breakout 2P 3P` | 각 5 | 필요량/여분 | 3.5 mm 단자와 혼동하지 않기 | 외부 버튼·LED·하네스 연결 |
| [ ] | `resistor assortment 1/4W 1%` | 1세트 | 필요량/여분 | 220 Ω, 330 Ω, 1 kΩ, 2.2 kΩ, 4.7 kΩ, 10 kΩ, 100 kΩ 포함 | LED, PhotoMOS, 옵토, 트랜지스터 |
| [ ] | `ceramic capacitor assortment` | 1세트 | 필요량/여분 | 100 nF와 1 µF 포함 | 디바운스·디커플링 |
| [ ] | `electrolytic capacitor assortment` | 1세트 | 필요량/여분 | 10 µF, 47 µF, 정격 16 V 이상 포함 | 전원 안정화 |
| [ ] | `22AWG silicone stranded wire` | 적·흑 각 5 m | 필요량/여분 | 연선, 주석도금이면 편리 | 5 V·12 V 저전압 전원 |
| [ ] | `heat shrink tube assortment` | 1세트 | 필요량/여분 | 1~6 mm | 납땜부 절연 |
| [ ] | `universal perfboard 2.54mm` | 3 | 3/0 | 단면 독립 패드형 | 브레드보드 검증 후 고정 |
| [ ] | `inline fuse holder 5x20mm` + `1A slow blow fuse` | 2세트 | 2/0 | 5 V/12 V 저전압 분기용 | PoC 배선 단락 보호 |

ESP32-S3 LCD와 Pi는 서로 다른 5 V 어댑터로 공급한다. 두 노드 사이에 전원선이나 신호 접지를 길게 연결하지 않고 Wi-Fi로만 통신한다.

## 3. 측정·조립 도구

이미 보유한 도구는 주문하지 않아도 된다.

| 주문 | 품목 및 검색어 | 권장 수량 | 선정 기준 | 주 사용처 |
|---|---|---:|---|---|
| [ ] | 디지털 멀티미터 | 1 | DC 전압, 저항, 연속성, 다이오드 모드 | VLX 핀 조사, 전원·극성 확인 |
| [ ] | `8 channel USB logic analyzer 24MHz` | 1 | 3.3 V 입력 호환, sigrok/PulseView 지원 | UART, 버튼 펄스, PWR_LED 파형 확인 |
| [ ] | 온도 조절 인두기·받침대 | 1 | 미세 팁, 60 W급 | 하네스·만능기판 제작 |
| [ ] | 무연 또는 유연 납·플럭스·흡입선 | 1세트 | 전자회로용 | 납땜과 수정 |
| [ ] | 와이어 스트리퍼·니퍼·핀셋 | 1세트 | 22~30 AWG 대응 | 배선 제작 |
| [ ] | JST 압착기 | 1 | PH/XH/소형 단자 대응 여부 확인 | VLX 비파괴 분기 하네스 |
| [ ] | `3.3V USB UART CP2102` | 1 | 전압 선택 가능, 5 V TTL로 사용 금지 | Pi/Pico 진단 콘솔 예비 경로 |
| [ ] | 절연 매트·보안경 | 각 1 | 전자 작업용 | 작업자 보호 |

USB-UART는 KSM 연결용이 아니다. KSM은 장비의 USB 인터페이스와 Pi의 Linux FTDI 드라이버를 사용하고, USB-UART는 개발 보드 진단에만 사용한다.

## 4. 보유품 확인 후 생략 가능한 항목

- USB-C 데이터 케이블, micro-USB 데이터 케이블, microSD 리더
- 브레드보드, 점퍼선, 저항·커패시터 키트
- 멀티미터, 인두기, 로직 애널라이저, USB-UART
- 5 V/3 A 어댑터 두 개와 12 V/1 A 어댑터
- 케이스 전면 패널 2핀 연장선과 각종 JST 하우징

다만 USB 절연기, PhotoMOS, LTV-814는 외형이 비슷한 대체품을 보유했더라도 정격과 내부 구조를 데이터시트로 확인하기 전에는 생략하지 않는다.

## 5. 이번에는 주문하지 않는 것

다음 품목은 실제 EXC 설치 또는 최종 제작 단계에서 장비 명판, 내부 배선, 설치 장소를 확인한 뒤 별도 선정한다.

- EXC용 AC-3 모터 정격 컨택터
- 컨택터 코일용 12/24 V 전원공급기
- 배선용 차단기, 누전 차단기, 퓨즈, 서지 보호기와 접지 자재
- AC 정격 케이블, 글랜드, 단자대와 난연 인클로저
- ATtiny1616과 커스텀 Supervisor HAT PCB
- 최종 제어기 케이스, 패널 가공물과 방수 커넥터

이번 PoC 주문에 가정용 스마트 플러그나 PCB 실장용 소형 릴레이를 EXC 본체 전원 차단 용도로 추가하지 않는다. EXC 실제 부하는 냉각기 모터의 기동 전류를 만족하는 인증된 외부 컨택터와 보호 장치가 필요하다.

## 6. 부품이 도착하면 확인할 사항

### 6.1 입고 검사

- [ ] LCD 보드 실크와 상품명이 `ESP32-S3-Touch-LCD-3.5`인지 확인한다.
- [ ] Pi가 `Zero 2 W`이고, 핀 헤더와 OTG 어댑터가 물리적으로 간섭하지 않는지 확인한다.
- [ ] Pico가 헤더 납땜된 `Pico 2 H`인지 확인한다.
- [ ] 두 microSD에 각각 식별 라벨을 붙인다.
- [ ] USB 절연기의 12 Mbps 설정과 host/device 방향, 절연 전원 내장 여부를 설명서에서 확인한다.
- [ ] AQY212EH, LTV-814, 2N2222A의 제조사별 핀 배열을 데이터시트와 대조한다.
- [ ] G5V-2 코일 표기가 `DC5V`인지 확인한다.
- [ ] 모든 전원 어댑터를 부하 연결 전에 멀티미터로 측정한다.

### 6.2 권장 조립·검증 순서

1. LCD 보드, Pi, Pico를 각각 독립 전원으로 부팅하고 펌웨어 기록 경로를 확인한다.
2. SHT45, MCP23017, 엔코더, 버튼과 버저를 LCD 보드에 연결해 로컬 UI 입출력을 검증한다.
3. 두 노드 사이의 전용 2.4 GHz Wi-Fi 자동 페어링과 재부팅 후 재연결을 검증한다.
4. Pi OTG → USB A-B → ADuM3160 → USB A-B → 칠러 순서로 연결하고, 먼저 KSM 읽기 전용 polling만 시험한다.
5. Pico 출력으로 G5V-2를 구동하여 EXC ON/OFF를 12 V LED로 모사한다.
6. 실제 PC 대신 버튼과 LED로 만든 저전압 지그에서 PWR_BTN 경유, PWR_SW 펄스, PWR_LED 연속·점멸·OFF 판정을 검증한다.
7. PC 전원이 분리된 상태에서 실제 메인보드 전면 패널 헤더에 연결하고, 원래 케이스 버튼이 계속 동작하는지 확인한다.
8. VLX의 AC 플러그를 뽑은 상태에서 전면 버튼 커넥터와 회로를 조사하고, 원래 버튼을 보존하는 Y 하네스를 만든다.
9. VLX를 전원 인가한 상태에서의 전압·극성 측정은 자격을 갖춘 작업자가 절연된 계측기로 수행한다. 단순 접점임이 확인된 뒤에만 AQY212EH를 병렬 연결한다.
10. 칠러 준비 조건 10초 유지 후 PC 기동, 절전 중 칠러 유지, 종료 후 기본 5분 후행 운전, 후행 중 재기동을 순서대로 검증한다.
11. 운전 중 냉각 이상, Wi-Fi 단절, KSM 오류를 주입해 버저와 LCD 경보만 발생하고 PC 전원에는 개입하지 않는지 확인한다.

## 7. PoC 완료 판정

- [ ] EXC-900/VLX-2000 프로파일 중 하나를 선택하면 해당 출력만 활성화된다.
- [ ] LCD에서 목표 온도, 펌프 단계, 인터록 기준과 후행 운전 시간을 조절하고 KSM 재조회 값으로 반영을 확인한다.
- [ ] 칠러 준비 조건이 10초간 유지되기 전에는 PC `PWR_SW` 펄스가 나오지 않는다.
- [ ] `PWR_LED` 연속 점등은 ON, 점멸은 절전, 8초 연속 무펄스는 OFF로 분류된다.
- [ ] 절전 중 칠러가 계속 운전한다.
- [ ] PC 종료 후 기본 5분 동안 후행 운전하며 LCD에서 0~30분으로 변경할 수 있다.
- [ ] 후행 운전 중 재기동 요청을 받으면 칠러 정지를 취소하고 준비 절차로 복귀한다.
- [ ] PC 운전 중 냉각 이상은 LCD·LED·버저 경보만 만들고 `PWR_SW`를 조작하지 않는다.
- [ ] VLX 원래 전원 버튼과 제어기 PhotoMOS 버튼이 각각 독립적으로 동작한다.
- [ ] EXC 더미 릴레이의 명령 상태와 피드백 불일치를 검출한다.
- [ ] Wi-Fi 또는 KSM 상태가 불명확할 때 임의의 칠러 OFF 명령을 만들지 않는다.

## 8. 예상 예산

판매처와 정품 여부에 따라 차이가 크므로 다음 금액은 배송비를 제외한 발주 검토용 범위다.

| 구분 | 예상 범위 |
|---|---:|
| LCD·Pi·Pico·microSD | 120,000~190,000원 |
| USB 절연·센서·I/O 부품 | 70,000~130,000원 |
| 전원·배선·제작 소모품 | 60,000~120,000원 |
| 측정·납땜 도구 | 보유 시 0원, 신규 80,000~250,000원 |
| **도구 제외 합계** | **250,000~440,000원** |

가격이 지나치게 낮은 SHT45, AQY212EH, 고내구성 microSD는 가품 가능성을 확인한다. 핵심 반도체는 데이터시트를 제공하고 반품이 가능한 판매처를 우선한다.
