# SCV — 다중 AMR 자율점검 시스템

> 두산로보틱스 ROKEY 9기 최종 프로젝트 · 2026.10 · **8인**(4인 팀 2개 통합)
> 팀 프로젝트이며, 아래는 제(김도윤) 담당 범위입니다.
>
> **담당: PM · 시스템 통합** — 5개 파트 일정·인터페이스 총괄, 팀 간 통신 계약서 작성·개정,
> 로봇 임무 실행 / 관제 작업 관리 / 관제 웹–ROS 브리지 등 **파트 사이를 잇는 부분 직접 구현**

TurtleBot4 두 대가 물류창고 모형을 구역별로 나눠 순찰하며 **Wi-Fi 음영 지역을 지도로 만들고**,
같은 순찰에서 **소화기 · 방화셔터 · 유도등을 점검**하는 다중 AMR 시스템입니다.

자동화 물류센터에서 AMR 이 신호가 약한 구역에 들어가면 통신이 지연되고, 산업용 안전 프로토콜은
이를 충돌 위험으로 보아 비상정지를 겁니다. 통로 한가운데 멈춘 한 대가 뒤따르는 로봇까지 세우고
결국 사람이 들어가 풀어야 합니다. 그래서 **음영 구역을 사고 전에 미리 찾는 것**을 주 기능으로,
소방·안전 설비 점검을 부가 기능으로 잡았습니다.

`ROS 2 Jazzy` · `Nav2` · `SLAM` · `FastDDS` · `TurtleBot4` · `OAK-D` · `YOLO` ·
`FastAPI` · `PostgreSQL` · `Python`

## 시연과 자료

| | |
|---|---|
| 시연 영상 (55초) | [두 대 동시 순찰 — 챕터 자막 포함](https://raw.githubusercontent.com/tnduaehowl12/portfolio-assets/main/scv/scv_demo_two_robots.mp4) |
| 전체 구조 | [scv_overview.png](https://raw.githubusercontent.com/tnduaehowl12/portfolio-assets/main/scv/scv_overview.png) |
| 시스템 배치 | [scv_system_layout.png](https://raw.githubusercontent.com/tnduaehowl12/portfolio-assets/main/scv/scv_system_layout.png) |
| 핵심 흐름 | [scv_core_flow.png](https://raw.githubusercontent.com/tnduaehowl12/portfolio-assets/main/scv/scv_core_flow.png) |
| 모듈별 설계도 8장 | [`docs/deliverables/system-diagram`](docs/deliverables/system-diagram) |

![순찰 결과 — 계획 경로와 실제 궤적, 0.5 m 칸마다의 RSSI 히트맵(−85 → −35 dBm), 점검 설비 8곳, 도달 실패 구간](https://raw.githubusercontent.com/tnduaehowl12/portfolio-assets/main/scv/rssi_heatmap.png)

## 설계에서 신경 쓴 것

- **없는 데이터를 숫자로 채우지 않았습니다.** 신호가 안 잡힌 측정을 −100 dBm 으로 적지 않고
  '미검출' 로 따로 표시했습니다. 임의 값을 넣으면 평균과 히트맵이 실제보다 나쁘게 왜곡됩니다
- **음영 판정** — 0.5 m 칸마다 검출값 중앙값이 −72 dBm 미만이거나 미검출 비율이 절반 이상이면 음영
- **예외 처리** — 순찰은 최대한 이어 가되, 확인하지 못한 것은 정상으로 처리하지 않고 '미확인'
  으로 기록 (통로 막힘 · 배터리 부족 · 상태 끊김 · 위치 불확실 · DB 저장 실패 등 8가지)

## 결과

- 실기 통합 시험에서 Wi-Fi 채널을 고정한 뒤 **두 회차 연속으로 측정 22/24구간 + 자동 도킹 완주**
  (나머지 2구간은 실물 장애물로 출발 전 자동 제외)
- 두 대 동시 순찰: Wi-Fi 측정 → 설비 8곳 점검 → 도킹까지 약 10분
- 시연용 연출이 아니라 **실제로 압력이 낮았던 소화기를 비전 점검이 먼저 찾아냈습니다**

약 3.2 × 6.5 m 테스트베드에서의 결과입니다.

---

## 빌드

```bash
deactivate 2>/dev/null   # 가상환경(venv)이 켜져 있으면 끈다
cd ros2_ws
colcon build --symlink-install
source install/setup.bash
```

가상환경이 켜진 채 빌드하면 `scv_msgs` 가 `No module named 'em'` 으로 실패합니다.
인터페이스 규격은 `docs/interfaces.md` 에 있습니다.

## 폴더 구조

```text
.
├── README.md
├── docs/            문서 (일지 가이드 docs/DAILY_GUIDE.md, 일지 docs/daily/, 아이디어 docs/idea/)
├── scripts/         도구 (scripts/daily.sh)
└── ros2_ws/         ROS 2 워크스페이스 — 빌드는 여기서
    └── src/         ROS 패키지
```

빌드는 `ros2_ws` 안에서 합니다. `build/`, `install/`, `log/` 가 그 안에만 생기고 git 에는
올라가지 않습니다.

## 브랜치 (프로젝트 기간 중 팀 운영 규칙)

팀마다 브랜치 하나를 썼습니다. **모든 브랜치에 패키지 전체가 들어 있고**(`main` 과 같은 구조),
팀은 자기 패키지만 고쳤습니다. `main` 이 통합본입니다.

| 팀 | 브랜치 | 이 팀이 고치는 패키지 |
| --- | --- | --- |
| 비전·물리 인프라 | `feature/vision` | `scv_detection` |
| 주행·작업 실행 | `feature/driving` | `scv_nav` |
| 관제·작업 관리 | `feature/control` | `scv_manager`, `scv_monitor` |
| 통신 인프라·RSSI | `feature/survey` | `scv_survey` |
| PM + 통합 | `feature/pm` | `scv_bringup`, `scv_testbed`, `scv_msgs`, 여러 팀이 보는 문서 |
| 통합본 | `main` | (PM 이 팀 브랜치를 합침) |

- 자기 팀 브랜치에서만 커밋합니다. `main` 에는 PM 이 통합합니다
- 로봇 PC·관제 PC·RPi 는 `main` 으로 돌립니다 (통합 시험도 `main`)
- 처음 한 번: `git config pull.rebase false`
- 다른 팀 패키지와 `scv_msgs`, 여러 팀이 보는 문서(`README.md`, `docs/interfaces.md`,
  `docs/DAILY_GUIDE.md`)는 팀 브랜치에서 고치지 않습니다
- `main` 이 바뀌면 PM 이 팀 브랜치에 합쳐 넣습니다. 팀원은 `git pull` 만 하면 됩니다
