# UnderWorld — Macro Watcher & Consciousness Warning Engine

> **Hades ONLY measures. Hades NEVER acts.**

UnderWorld는 **행성 규모 룰 위반**을 감시하고, 그 결과를 **ConsciousnessSignal(의식 경고)**로만 내보내는 독립 패키지입니다.  
결정·처벌·전이(예: IMMORTAL → MORTAL)는 UnderWorld가 하지 않습니다. **그것은 항상 호출 측(IntegrityFSM / Homeostasis / Runner)의 동역학에서만** 일어납니다.

- ✅ **독립 설치·사용** — `solar`·CookiieBrain 없이 동작
- ✅ **옵셔널 코어** — 물리 엔진이 있으면(덕 타이핑 or 연결) 더 풍부한 감시, 없으면 스텁으로 QUIET
- ✅ **출력은 신호 DTO만** — `ConsciousnessSignal` 불변(frozen), 위변조 불가
- ✅ **다중 신호** — `listen()` 반환은 `List[ConsciousnessSignal]`. 복합 위반 시 누락 없이 전달

---

## 이 패키지가 나온 곳 — 서사적 출처 (본편과 독립 엔진의 구분)

**본편** = [CookiieBrain](https://github.com/qquartsco-svg/cookiie_brain) 의 **solar/** 폴더. 그 안에는 **천지창조 로직**(Creation Days, N-body·빛·궁창·땅·해·달·에덴·지상 동역학)이 **서사적으로 이어지는** 구조로 되어 있다. 엔진·모듈·README는 **그 서사 안에서** "어디에 위치하는지", "어떤 흐름으로 연결되는지"로 쓰여 있다.

**이 레포(UnderWorld)** 는 그 본편과 **따로 간다**.  
본편 서사 안에서 **"지하(하데스)의 목소리"** — 물리 코어(day4)와 지상(EdenOS) 사이에서 거시 룰 위반을 감시하고, 의식 경고(ConsciousnessSignal)만 내보내는 레이어 — 로 등장한 모듈을, **의존성과 서사 맥락을 제거하고** 엔지니어링만 남긴 **완전 독립 패키지**로 분리한 것이다.

- **본편**: 스토리·로직이 서사적으로 이어짐. solar/underworld 는 그 안의 한 레이어.
- **이 패키지**: "어떤 서사에서 생성된 모듈인지" 출처를 밝히고, **설치·API·설계 제약**만 기술한 독립 엔진.

아래 문서은 모두 **독립 엔진** 관점(엔지니어링·활용·확장)이며, 본편의 천지창조 서사는 [CookiieBrain/solar](https://github.com/qquartsco-svg/cookiie_brain/tree/main/solar) 에서 이어져 나간다.

---

## 어떤 시스템인지 — What UnderWorld Is

UnderWorld는 **거시 규칙 위반을 감시하고, 의식 경고(ConsciousnessSignal)만 내보내는 미들웨어**입니다.

| 구분 | 설명 |
|------|------|
| **역할** | 물리/거시 계층(코어)의 “정상 여부”를 읽고, 위반 시 **경고 신호 리스트**만 반환. 판정·처벌·상태 변경은 하지 않음. |
| **입력** | `tick`, (선택) `deep_engine`(물리 코어 또는 덕 타이핑 객체), (선택) `world_snapshot`. |
| **출력** | `List[ConsciousnessSignal]` — QUIET이거나 RULE_VIOLATION / ENTROPY_WARNING / SYSTEM_PANIC 등. |
| **위상** | **50_DIAGNOSTIC_LAYER** — 진단·감시 전용. “지상” 에이전트/FSM/Runner가 이 신호를 구독해 stress·integrity·전이에 반영. |

**한 줄 정의**  
> “행성 규모 제약을 지키는지 **측정만** 하고, 결과를 **불변 신호**로만 전달하는 **진단 엔진**.”

---

## 엔지니어링 개요 — Engineering

### 아키텍처

- **Observer 패턴**: `HadesObserver.listen(tick, ...)` 가 매 틱 호출되며, 코어 스냅샷 → 룰 평가 → 신호 리스트를 **side-effect 없이** 반환.
- **데이터 흐름**: `deep_engine`(또는 None) → `read_deep_snapshot()` → `DeepSnapshot` → `evaluate_rules_all()` → `List[ConsciousnessSignal]`.
- **의존성**: 지상(solar.eden 등)을 **import하지 않음**. 표준 라이브러리 + `dataclasses`만 사용. 코어는 **덕 타이핑**으로만 결합.

### 모듈 책임

| 모듈 | 책임 |
|------|------|
| `consciousness.py` | 신호 DTO(`ConsciousnessSignal`, `SignalType`). **frozen** — 생성 후 변경 불가. |
| `deep_monitor.py` | `DeepSnapshot` 생성. `engine=None`이면 스텁, 있으면 `getattr(engine, "magnetic_ok", True)` 등으로 읽음. |
| `rules.py` | `RuleSpec` 리스트와 `evaluate_rules_all()`. 룰 추가·변경은 이 레이어만 수정 (OCP). |
| `hades.py` | `listen()` 오케스트레이션만. 룰/문구/심각도는 rules에 위임. |

### 제어 구조

- **UnderWorld = 센서(Observer)**. 호출 측 = **제어기(Controller)**.  
- “Plant(행성 물리) → Observer(Hades) → Feedback signal → Controller(FSM/Homeostasis)” 구조로, **전이는 Controller 쪽에서만** 발생.

---

## 서사 — Narrative / Story

UnderWorld는 이름처럼 **“지하(Hades)의 목소리”**를 구현합니다. 단, 여기서 Hades는 **캐릭터가 아니라 행성의 거시 법칙을 대변하는 텔레메트리**입니다.

- **지하**는 “땅 밑”이 아니라 **물리 코어·거시 제약이 지켜지는지 감시하는 레이어**를 가리킵니다.
- **목소리**는 **ConsciousnessSignal** — 에이전트/지상 시스템이 “들을 수 있는” 경고 메시지와 severity입니다.
- **측정만, 행동 없음**: 이 엔진은 “추방해라”, “권한을 빼라” 같은 **결정을 내리지 않습니다**.  
  한계선을 넘었을 때 **“이렇게 위반되었다”**만 알리고, **전이(강등·추방)는 지상의 동역학(FSM, Runner)**이 수행합니다.

그래서 서사적으로는:

- **해킹/스토리 이벤트**가 아니라, **물리·항상성 한계를 넘으면 자연스럽게** 상태가 바뀌는 구조입니다.
- “지하가 심판한다”가 아니라 **“지하가 측정한 값”을 지상이 읽고, 그에 따라 동역학이 전이**합니다.

---

## Why — 설계 의도와 서사

UnderWorld는 **스토리 진행 장치**가 아니라 **거시 제약의 감시 계층**입니다.

- 지상 시스템(EdenOS / World Runner)이 **안정성 한계선**을 넘어갈 때,
- UnderWorld는 **경고만** 제공합니다.
- 지상 시스템은 그 경고를 입력으로 받아 **자연스럽게**(동역학적으로) 상태 전이를 수행합니다.

> 즉, “해킹/추방” 같은 서사 이벤트가 아니라  
> **“항상성 유지 실패 → 경고 증가 → FSM 전이”**로만 흐르게 만드는 것이 목표입니다.

**서사적 위치**  
“지하의 목소리”는 인격화된 악역이 아니라, **행성 규모 물리 룰이 어긋났을 때의 텔레메트리**입니다.  
이 엔진은 그 목소리를 **측정(measure)**만 하고, **행동(act)**은 하지 않습니다.  
그 경계가 유지될 때, 전체 시스템은 “스토리 의존”이 아니라 **동역학적 한계선**으로만 움직입니다.

---

## 활용성과 확장성 — Use Cases & Extensibility

### 활용 사례

| 사례 | 설명 |
|------|------|
| **행성/물리 시뮬레이터** | N-body·자기권·열역학 등 코어가 한계를 넘으면 경고 리스트를 받아, 로깅·대시보드·알림에 사용. |
| **에이전트·게임 러너** | 매 틱 `listen()` 호출 후, 반환된 신호를 observe·stress·integrity에 반영해 “한계 초과 시 자연스러운 전이”만 수행. |
| **모니터링·진단 파이프라인** | 물리/시스템 상태를 덕 타이핑 객체로 넘기면, 룰 위반별 시그널이 나오므로 다운스트림(알람·FSM·로깅)에 연결 가능. |
| **테스트·스텁** | `deep_engine=None`이면 항상 `[QUIET]`. 테스트 시 위반 시나리오만 가진 더미 엔진을 넘겨 단위/통합 테스트에 활용. |

### 확장 방향

| 방향 | 내용 |
|------|------|
| **룰 주입** | `evaluate_rules_all(deep, rules=커스텀_RuleSpec_리스트)` 로 정책만 교체. hades 코드 수정 없음. |
| **world_snapshot 연동** | `getattr(world_snapshot, "eden_index", 1.0)` 등으로 severity·민감도 보정 (덕 타이핑 유지). |
| **다중 수신자** | `listen()` 반환 리스트를 Pub/Sub·이벤트 버스로 배포해 수신자별 필터·가중치 적용. |
| **LLM/Oracle 메시지** | 위반 파라미터 배열을 LLM에 넘겨 자연어 메시지 생성 후 `ConsciousnessSignal.message`만 교체. |
| **시계열·이상 탐지** | 수천 틱 스냅샷을 누적해 ML 기반 이상 탐지 후, “N틱 전” 점진적 경고로 severity 보강. |

독립 패키지이므로 **다른 시뮬레이터·에이전트 프레임워크**에 그대로 끼워 넣어, “거시 진단 레이어”만 교체·재사용할 수 있습니다.

---

## Install

```bash
# 개발 모드(권장)
pip install -e .

# 일반 설치
pip install .
```

---

## Quick Start

### 1) 코어 엔진 없이 (스텁 동작)

코어가 없으면 `deep_monitor`가 스텁 스냅샷을 만들고, Hades는 `[QUIET]` 한 개를 반환합니다.

```python
from underworld import HadesObserver, ConsciousnessSignal

hades = HadesObserver()
signals = hades.listen(tick=1, deep_engine=None)

print(len(signals), signals[0].signal_type, signals[0].severity, signals[0].message)
# 1 QUIET 0.0
```

### 2) 덕 타이핑 코어로 감시 (Plug-and-Play)

`deep_engine`에 **진짜 solar 엔진이 아니어도 됩니다.**  
`magnetic_ok`, `thermal_ok`, `gravity_ok` 속성만 있으면(또는 `getattr`로 읽을 수 있으면) 동작합니다.

```python
from underworld import HadesObserver, ConsciousnessSignal

class MyEngine:
    magnetic_ok = False   # 위반
    thermal_ok = True
    gravity_ok = True

hades = HadesObserver()
signals = hades.listen(tick=10, deep_engine=MyEngine())

for s in signals:
    print(s.signal_type, s.severity, s.message)
# RULE_VIOLATION 0.6 거시 룰: 자기장 이상 감지
```

복합 위반 시 **여러 신호**가 리스트에 담깁니다.

```python
class FailingPlanet:
    magnetic_ok = False
    thermal_ok = False
    gravity_ok = True

signals = hades.listen(1, deep_engine=FailingPlanet())
# [RULE_VIOLATION, ENTROPY_WARNING] 등 위반별 ConsciousnessSignal 리스트
```

---

## API

### `HadesObserver.listen(...) -> List[ConsciousnessSignal]`

```python
def listen(
    tick: int,
    world_snapshot: Any = None,
    deep_engine: Any = None,
) -> List[ConsciousnessSignal]
```

| 인자 | 설명 |
|------|------|
| `tick` | 현재 시뮬레이션 틱. |
| `deep_engine` | (선택) 물리 코어 객체. 덕 타이핑: `magnetic_ok`, `thermal_ok`, `gravity_ok` 등만 있으면 됨. `None`이면 스텁 → `[QUIET]`. |
| `world_snapshot` | (선택) 지상 스냅샷. 추후 민감도 보정용. **UnderWorld는 지상 타입을 import하지 않음** — `getattr(world_snapshot, "eden_index", 1.0)` 등으로만 사용. |

**반환**  
- `List[ConsciousnessSignal]`. 위반 없으면 `[QUIET]` 1개. 복합 위반 시 위반 룰별로 여러 개.
- 각 `ConsciousnessSignal`은 **frozen** — 필드 위변조 불가.

### `ConsciousnessSignal` (frozen)

| 필드 | 의미 |
|------|------|
| `signal_type` | `QUIET` \| `RULE_VIOLATION` \| `ENTROPY_WARNING` \| `SYSTEM_PANIC` |
| `severity` | 0.0 ~ 1.0 |
| `message` | 인간이 읽는 경고 문장 |
| `source` | `"underworld.hades"` |
| `tick` | 신호 발생 틱 |

### 기타

- `make_hades_observer(tick=0) -> HadesObserver` — 관찰자 생성.

---

## Package Layout

```
underworld/
  __init__.py          # ConsciousnessSignal, SignalType, HadesObserver, make_hades_observer
  consciousness.py     # ConsciousnessSignal DTO (frozen)
  deep_monitor.py      # DeepSnapshot + read_deep_snapshot (덕 타이핑, solar 미참조)
  hades.py             # HadesObserver.listen() 오케스트레이터
  rules.py             # RuleSpec, DEFAULT_RULES, evaluate_rules / evaluate_rules_all
```

---

## Core Contract — DeepSnapshot

UnderWorld 내부는 `DeepSnapshot`으로 코어 상태를 읽습니다.

| 필드 | 의미 |
|------|------|
| `magnetic_ok` | 자기장 정상 여부 |
| `thermal_ok` | 열역학/지열 정상 여부 |
| `gravity_ok` | 중력장 정상 여부 |
| `core_available` | 코어 데이터 사용 가능 여부 |
| `extra` | 확장용 dict |

- `deep_engine=None` → `DeepSnapshot.stub()` → 결과적으로 `[QUIET]`.
- `deep_engine`에 위 필드가 있으면(덕 타이핑) 해당 값으로 스냅샷을 만들고, `rules.evaluate_rules_all()`로 위반 목록을 만들어 **리스트**로 반환합니다.

---

## Integration Pattern — 통합 시 권장 패턴

UnderWorld는 **판정 엔진**이 아니라 **감시 엔진**입니다.  
통합 시에는 “관측 → 항상성 업데이트 → FSM 전이”의 **입력**으로만 연결합니다.

```python
# 매 틱 (Runner 등에서)
signals = hades.listen(tick=t, world_snapshot=world, deep_engine=core)

obs = agent.observe(world=world, hades_signal=signals)   # observe는 리스트 또는 첫 항목 사용
homeostasis.update(tick=t, world=world, hades_signal=signals)  # 리스트 시 최대 severity 등으로 가산
integrity_fsm.step(tick=t, integrity=snap.integrity)     # 전이 발생 지점은 여기(지상)만
```

- ✅ UnderWorld는 **상태를 바꾸지 않습니다.**
- ✅ **전이는 오직 지상 동역학 레이어**에서만 일어납니다.
- ✅ Runner가 신호를 Homeostasis/FSM에 **강제 주입**하면, 에이전트가 observe에서 무시해도 임계치 초과 시 자연 강등이 동작합니다.

---

## Extending Rules — 룰 확장

- 룰셋은 **rules.py**의 `RuleSpec`, `DEFAULT_RULES`, `evaluate_rules_all(deep_snapshot, rules=...)`로 **데이터 주도**로 분리되어 있습니다.
- 새 룰 추가·정책 변경 시 **rules.py만 수정**하면 되며, `hades.py`는 건드리지 않습니다 (OCP).
- `world_snapshot`을 `evaluate_rules`에 넘겨 severity/민감도 보정하는 것은 추후 확장 포인트입니다 (덕 타이핑 유지).

---

## Design Constraints — 불가침 규칙

1. **Hades ONLY measures. Hades NEVER acts.**  
   관측·신호 생성만. 결정·처벌·전이는 호출 측 전용.

2. **UnderWorld는 지상 패키지(solar.eden 등)를 import하지 않는다.**  
   의존성 격리·순환 참조 방지.

3. **신호 DTO는 frozen.**  
   수신 측이 severity 등을 위변조할 수 없음.

4. **코어 엔진은 optional / 덕 타이핑 우선.**  
   없으면 스텁으로 QUIET.

5. **“서사 이벤트”가 아니라 “동역학적 한계선”.**  
   지상 FSM이 한계를 넘을 때만 전이한다.

---

## Layer — 엔진 허브 배치

- **50_DIAGNOSTIC_LAYER** (ENGINE_HUB) — 거시 감시·진단용 엔진으로 배치 가능.

---

## Origin & License

- **서사적 출처**: [CookiieBrain](https://github.com/qquartsco-svg/cookiie_brain) `solar/` 의 천지창조·에덴 로직 안에서 **"지하(하데스)의 목소리"** 레이어로 설계된 모듈. 그 모듈을 의존성 없이 분리한 것이 이 독립 패키지.
- **GitHub**: [qquartsco-svg/UnderWorld](https://github.com/qquartsco-svg/UnderWorld)
- **Author**: Qquarts co. (GNJz)
- **License**: MIT
