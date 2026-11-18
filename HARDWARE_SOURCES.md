# 공식 근거 / 확인 기록

확인일: 2026-11-18. 가격 견적이나 구매 가능 보장이 아니라 설계용 확인이다. 지역 배송, 재고 예약, 실물 리비전은 확인하지 않았다. 이 작업은 공식 기술 문서를 우선한 제작 과제이므로 추천 리뷰/커뮤니티 글을 전기적 근거로 사용하지 않았다.

| 출처 / URL | 사용한 사실 / 판단 |
|---|---|
| Raspberry Pi Pico 데이터시트, Raspberry Pi, PDF: https://pip-assets.raspberrypi.com/categories/610-raspberry-pi-pico/documents/RP-008307-DS-2-pico-datasheet.pdf | 원형 RP2040, 2MB flash, 51×21×1 PCB, 3.3V GPIO, 물리 핀, 4×Ø2.1 홀, 전원/USB, 소비전류 예시는 최대 보장 아님. 기계 도면 p7(인쇄 페이지6) 확인 |
| 공식 문서 목록: https://pip.raspberrypi.com/categories/610-raspberry-pi-pico | 원형 Pico 문서 구분, 2026-07-15 데이터시트 갱신 표기 |
| Pico 제품 재고 페이지: https://www.adafruit.com/product/4864 | Pico without Headers 변형 판매/재고 표기 확인. 전기 핀 근거는 판매 설명 대신 Raspberry Pi 원문 사용 |
| Waveshare 제품 페이지: https://www.waveshare.com/pico-oled-1.3.htm | 고정 모델명, SH1107, 52×25 외곽, 29.42×14.70 active area, 기본 SPI, 주문 가능/InStock 구조화 표기 |
| Waveshare wiki: https://www.waveshare.com/wiki/Pico-OLED-1.3 | 40mA, VSYS 입력, GP8/9/10/11/12, SPI 기본 설정 |
| Waveshare 기계 도면 이미지: https://www.waveshare.com/img/devkit/LCD/Pico-OLED-1.3/Pico-OLED-1.3-details-size.jpg | 52×25 PCB, 29.42×14.70 표시 영역. 모듈 두께·부품 높이·정확한 화면 원점은 미기재라 가정으로 분리 |
| Waveshare 공식 회로도 PDF: https://files.waveshare.com/upload/1/1b/Pico-OLED-1.3_SchDoc.pdf | VSYS→RT9193-33 LDO, R1/enable, R3 NC 우회, socket 물리 핀, 두 온보드 버튼 GP15/GP17 |
| Waveshare 공식 실행 예제: https://github.com/waveshare/Pico_code/blob/main/Python/Pico-OLED-1.3/Pico-OLED-1.3(spi).py | SH1107 초기화와 행/열 매핑, SPI1 mode0. wiki의 mode3 설명과 상충해 실제 예제 쪽 선택. 파일을 실행하지 않고 드라이버 설정만 참고 |
| Adafruit 1445 제품: https://www.adafruit.com/product/1445 | 비조명 적색 NO 패널 스위치, 최대18.06×18×29.4, 재고 표기 |
| Adafruit 기술 도면: https://cdn-shop.adafruit.com/datasheets/R13diagram.jpg | Ø16 패널/Ø13 버튼, 너트 구조. 본 케이스는 여유 Ø16.4 원형 구멍, 실제 회전 방지는 배선 고정과 너트로 처리 |
| Kingbright WP7113ID 제품: https://www.kingbrightusa.com/product.asp?catalog_name=LED&product_id=WP7113ID | 5mm 적색 diffused LED, 제조사 유통 재고 표시 |
| Kingbright 데이터시트: https://www.kingbrightusa.com/images/catalog/SPEC/WP7113ID.pdf | 패키지/극성/Vf/정격 확인. 데이터시트의 허용차 때문에 Ø5.3은 시제품 실측 필요 |
| Pico SDK 2.1.1 릴리스: https://github.com/raspberrypi/pico-sdk/releases/tag/2.1.1 | C SDK와 RP2040 타깃. 실제 사용 커밋은 dependencies.lock.json에 기록. 기본 USB descriptor의 250mA를 소스에서 직접 확인 |
| Arm 툴체인 안내: https://learn.arm.com/install-guides/gcc/arm-gnu/ | Cortex-M용 arm-none-eabi 툴체인 선택. 이 작업은 14.2.rel1을 고정해 실제 다운로드·실행했으며 안내 페이지의 최신 버전과 혼동하지 않음 |
| 실제 사용 Arm 아카이브: https://developer.arm.com/-/media/Files/downloads/gnu/14.2.rel1/binrel/arm-gnu-toolchain-14.2.rel1-x86_64-arm-none-eabi.tar.xz | 다운로드 SHA-256 고정. 현재 시스템 컴파일러가 아니라 이 ARM 크로스 컴파일러로 빌드 |
| OBS WebSocket v5.7.4 공식 프로토콜: https://github.com/obsproject/obs-websocket/blob/5.7.4/docs/generated/protocol.md | Hello/Identify 인증, op6/7 요청 응답, outputs64/scenes4 구독, GetRecordStatus/StartRecord/StopRecord/GetSceneList/SetCurrentProgramScene |
| websockets15.0.1 문서: https://websockets.readthedocs.io/en/15.0.1/reference/asyncio/client.html | asyncio connect API. 실제 로컬 WebSocket 시험으로 사용 방식 검증 |

OBS 서버 포함 버전/기본 포트는 공식 개요도 확인했다: https://github.com/obsproject/obs-websocket (OBS 28.0.0+ 포함, 기본4455).

## 모순과 남은 불확실성
- OLED 판매 페이지의 3.3V 표기와 wiki의 VSYS/2.6~5.5V 표기는 공식 회로도의 모듈 입력 LDO로 해석했다. 최종 설계는 VSYS 핀을 사용하며 공장 기본 저항 상태만 지원한다.
- SPI mode 설명은 wiki(mode3)와 공식 실행 예제(mode0)가 다르다. 실제 예제 mode0 + 낮춘 2MHz로 구현했다. 실물 화면 출력·방향 검증은 남았다.
- Adafruit 버튼 페이지의 max 높이29.4와 도면의24.8 표기가 다르다. 짧은 쪽을 사실화하지 않고 30mm 하부 keep-out을 두었다.
- 제조사 부품 외형만으로 배선 휨, 사출 케이블 몰드, OLED 하부 리비전, 실제 출력 수축을 보장할 수 없다. 이 항목은 실물 확인 후 파라미터를 조정한다.
- 부품 재고는 확인 순간의 웹 표시다. 주문·결제·배송 가능성 확인이나 대체품 승인으로 이어지지 않는다.
