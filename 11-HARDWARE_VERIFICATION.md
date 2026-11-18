# 검증 기록

2026-11-18. 이 문서의 PASS는 아래 디지털 시험 범위에만 해당한다. 실물 제작/납땜/전류 측정/USB 플래시/실제 OBS 접속을 했다는 뜻이 아니다. 구매·제조 발주·유료 생성·외부 배포·저장소 생성·커밋은 하지 않았다.

## 실행 환경
Python 3.10.12, CMake 3.22.1, 네이티브 GCC11.4.0, GNU Arm14.2.rel1, Pico SDK2.1.1, CadQuery2.6.1/OpenCascade7.8, pyserial3.5, websockets15.0.1, Playwright1.56.0/Chromium. 정확한 SDK/TinyUSB/picotool 커밋과 툴체인 SHA-256은 `dependencies.lock.json`, `firmware/build.sh`에 고정했다. 최종 ZIP에는 다운로드한 툴체인/SDK/패키지/브라우저를 넣지 않았다.

## 수행 결과

| 검사 | 실제 결과 | 근거 파일 |
|---|---|---|
| RP2040 전체 펌웨어 ARM 빌드 | PASS. C/C++/ASM 링크, UF2/ELF/BIN 생성. 프로젝트 C는 `-Wall -Wextra -Werror` | `verification/repro-build.log` |
| 재현 빌드 경로 | PASS. SDK의 develop 자동 취득 대신 커밋 고정 picotool을 별도로 빌드해 새 `firmware/repro-build`에서 수행 | 동일 로그, `firmware/build.sh` |
| 펌웨어 크기 | text 40,628 B / data0 / bss5,408 B. BIN40,632 B. RP2040 Pico의 2MB flash 범위 안 | 로그 마지막 size, `verification/binary-report.json` |
| UF2 파일 구조 | PASS. 159×512B 블록, RP2040 family0xe48bff56, flash 시작0x10000000, 블록 순번·주소·magic·payload 확인. BIN과 byte 일치 | `verification/binary.log` |
| USB 전류 구성 | PASS(정적 확인). 실제 ELF의 usbd_desc_cfg를 찾아 bMaxPower125×2=250mA 확인. 전류 실측은 아님 | `verification/binary-report.json` |
| 버튼 C 로직 | PASS. 바운스/30ms 안정/길게 누름/해제/재누름/부팅 시 눌림 | `verification/debounce.log` |
| 상태 프레임 C 로직 | PASS. 정상/일시정지/버전/범위/음수/양수기호/초과수/잔여문자 거부 + 1,750ms freshness 경계, 18개 확인 | `verification/protocol.log` |
| 호스트 브리지 | PASS. 13개 자동시험. 실제 loopback WebSocket, 모의 OBS와 모의 USB transport 사용 | `verification/host-tests.log` |
| Python 소스/CLI | PASS. py_compile, --help 실행 | `verification/host-help.txt` |
| CAD 솔리드 | PASS. base/lid/clip 각각 유효 단일 솔리드. 정의한 부품 외형과 base/lid 사이 양의 겹침 체적0. 부품끼리 겹침0 | `verification/cad.log`, `enclosure/clearance_report.json` |
| 독립 STL 검사 | PASS. base8144 / lid5412 / clip528 삼각형. 모서리 공유 횟수 모두2, 연결 성분1, 양의 체적 | `verification/mesh-report.json` |
| 로컬 미리보기 | PASS. 조립/분해 슬라이더·재생·REC·SCENE·LINK LOST·재연결 상태 확인. 브라우저 JS 오류0 | `verification/preview.log`, `preview-tests.json` |
| 픽셀 확인 | PASS. 1920×1080 조립/분해, 배선도와 390px 모바일 전체 화면을 실제 렌더해 확인. 한글 폰트와 모바일 넘침/가림 수정 후 재검사 | `verification/preview-assembled.png`, `preview-exploded.png`, `preview-mobile.png`, `wiring.png` |

호스트 13개 시험은 인증 성공/실패, 실제 녹화 조회, 시작/중지, 중복 seq 억제, 500ms 제한, 장면 순환, 없는 장면 거부, 외부 녹화/일시정지, 불량 프레임, OBS 거부, 연결 상실/재조회, 녹화가 시작됐지만 응답이 사라진 경우의 재전송 방지, 조회 중 이벤트 발생에 따른 스냅샷 폐기, USB 재접속/큐 폐기를 포괄한다. 항목 일부를 한 시험에서 함께 확인했다. 테스트의 local-test 문자열은 가짜 서버용 값이며 실제 인증정보가 아니다.

첫 ARM 빌드에서 printf 정수형 경고와 재구성 시 SDK 경로 사용 오류를 발견해 수정했다. 최종 고정 의존성 경로를 새 빌드 폴더에서 다시 통과시켰다. 파일의 존재만 확인한 것이 아니라 ARM 실행 파일과 UF2 내부를 검사했다. 다만 바이너리 구조 검사도 실물 부팅을 대신하지 못한다.

## 재실행

프로젝트 루트, Python 가상환경 기준. 아래 명령은 프로젝트 안에서만 출력 파일을 만든다. CMake와 네이티브 GCC도 필요하다.

```sh
python -m pip install -r requirements-dev.txt
bash firmware/build.sh > verification/local-build.log 2>&1
python -m unittest discover -s tests -p test_bridge.py -v
cc -Wall -Wextra -Werror tests/test_debounce.c -o verification/test_debounce
./verification/test_debounce
cc -Wall -Wextra -Werror tests/test_protocol.c -o verification/test_protocol
./verification/test_protocol
python enclosure/design.py
python tests/check_mesh.py
python tests/check_binary.py
# 브라우저 검증을 원하면 (브라우저 다운로드 포함)
PLAYWRIGHT_BROWSERS_PATH="$PWD/tools/browsers" python -m playwright install chromium
PLAYWRIGHT_BROWSERS_PATH="$PWD/tools/browsers" python tests/test_preview.py
```

Windows에서 환경변수/네이티브 빌드 명령은 PowerShell 또는 WSL 문법으로 조정한다. 배포용 자동 플래시/자동 OBS 설정 변경은 없다.

## 확인하지 못한 것 / 남는 위험
- 실제 Pico/SH1107에서 USB 열거, 화면 주소·방향·SPI mode0, 버튼 접점, LED 가시성 및 실제 전압은 미검증.
- **실제 OBS와 연결한 적 없음.** 공식 v5 프로토콜 모의 서버로만 시험했다. 설치한 OBS 버전, 녹화 파일 생성, 인코더 오류, 디스크 부족, 다른 플러그인/외부 조작과의 경합은 실제 환경 시험 필요.
- 연결 상실 표시 설계는 마지막 상태 후 약1.75초, 통신 루프 지연을 감안해 약2초 목표다. 실제 시각적 지연·OS USB 버퍼는 측정하지 않았다.
- **USB suspend 저전력 적합성 미완료.** 전류 예산151.53mA는 가정과 제조사 예시를 합친 설계 배정이며 보장 최대가 아니다. 동작/초기접속/suspend/resume 실측 없이 제품화하지 않는다.
- PCB/헤더 두께, OLED 뒤 부품, 플렉스, 플러그 몰드, 나사 머리/나사산, 전선 휨과 납땜 높이는 단순 keep-out 검사 밖이다. 유리 압박과 배선 끼임을 실물에서 확인한다.
- FDM 수축/나사 체결력/버튼 반복 하중/낙하/난연/ESD/EMC/USB 인증 미검증. PCB 제조 파일은 없으며, 배선도를 전기 CAD라고 부르지 않는다.
- 장면 이름은 OLED에 표시하지 않는다. OLED는 연결 상태/녹화 상태/경과 시간을, 실제 다음 장면은 OBS 화면을 통해 확인한다.
- 데스크 컨트롤러에 별도 AI 추론은 구현하지 않았다. 이 과제의 실질 요구인 촬영 조작·상태 피드백을 구현했다.

## 실제 기기 인수 시험 (전부 미수행)

### 전기 / 초기 부팅
- [ ] BOM/리비전/기본 SPI 저항과 LDO 경로를 사진으로 기록. Pico W/2 또는 변형 OLED가 아님 확인.
- [ ] USB 제거 상태에서 VSYS-GND 단락 없음, 모든 핀-핀 연속성, 스위치 NO, LED A/K/330Ω 실측.
- [ ] USB 전력계/전원 분석기로 부팅·구성 전·전체 표시·LED 점등 전류 측정. 정상운전 목표200mA 이하, 250mA 초과 시 중단.
- [ ] Pico 3.3V와 모듈 내부3.3V, VSYS 전압·리플·온도 확인. GPIO에5V 없음.
- [ ] USB suspend/resume 전류와 복귀 시험. 허용 규격에 못 미치면 저전력 설계 수정 후 재시험.
- [ ] BOOTSEL 플래시, 재부팅 후 올바른 CDC 포트, OLED 가독성과 mode0/주소/방향 확인.

### OBS와 사용자 조작
- [ ] 실제 OBS 인증 성공/실패, READY, 호스트 비실행 시 LINK LOST. 잘못된 포트 장치를 제어하지 않음 확인.
- [ ] REC 시작/중지 각20회와 2시간 녹화: 실제 파일 생성/재생, 화면 시간과 OBS 시간 오차 기록.
- [ ] 버튼 바운스/길게 누름/빠른 연타/동시 버튼: 중복 명령이나 임의 시작 없음 확인.
- [ ] OBS 화면에서 시작/중지/일시정지/재개: LED와 OLED가 실제 상태를 따라감 확인.
- [ ] 외부 장면 변경 후 SCENE, Studio Mode, 장면 삭제/이름 변경/없는 --scene 처리 확인.
- [ ] 녹화 중 브리지 종료/USB 분리/OBS 서버 중지/인증 변경: 약2초 내 상태 미확인, 재연결 뒤 실제 상태 복구, 과거 입력 재생 없음 확인.
- [ ] 녹화 명령 응답만 소실시켜도 명령 자동 재시도/연속 토글 없음 확인. OBS에서 직접 상태를 확인.
- [ ] 디스크 부족/인코더 오류/권한 오류 때 녹화 성공으로 표시하지 않음 확인.

### 기구 / 장기 사용
- [ ] USB 실제 플러그/몰드 완전 삽입, 케이블 당김 때 PCB/포트 손상 없음, 버튼 너트/LED 고정 확인.
- [ ] 출력 공차 시험편, 나사 pilot/머리/길이, Pico 홀, OLED 두께/헤더/플렉스/클립 간섭 확인.
- [ ] 유리 압박·솔더 노출·배선 끼임 없이 뚜껑 닫힘. 각 버튼1000회 반복 후 균열·헐거움 확인.
- [ ] 8시간 전원/녹화와 재연결 반복, 온도·전류·디스플레이 번인/상태 갱신 확인.

시험 담당자 / 날짜 / 부품 리비전 / 프린터 / 실제 전류 / OBS 버전 / 결과를 별도로 기록한 뒤 제작 승인 여부를 결정한다.
