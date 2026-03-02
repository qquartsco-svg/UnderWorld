# UnderWorld — 거시 감시·의식 경고 독립 엔진

**Hades ONLY measures. Hades NEVER acts.**

행성 규모 룰 위반을 감시하고, **의식 경고(ConsciousnessSignal)** 만 내보내는 독립 패키지.  
CookiieBrain/solar 의존 없이 단독 설치·사용 가능.

---

## 설치

```bash
pip install -e .
# 또는
pip install .
```

---

## 사용

```python
from underworld import HadesObserver, ConsciousnessSignal

hades = HadesObserver()
signals = hades.listen(tick=1, world_snapshot=None, deep_engine=None)
# → [ConsciousnessSignal(QUIET)] (엔진 없으면 스텁)

# 물리 엔진이 있으면 덕 타이핑으로 전달
class MyEngine:
    magnetic_ok = False
    thermal_ok = True
    gravity_ok = True
signals = hades.listen(1, deep_engine=MyEngine())
# → 위반된 룰별 ConsciousnessSignal 리스트
```

---

## API

| API | 설명 |
|-----|------|
| `HadesObserver.listen(tick, world_snapshot=None, deep_engine=None) -> List[ConsciousnessSignal]` | 현재 틱 기준 신호 리스트. 위반 없으면 `[QUIET]`. |
| `ConsciousnessSignal` | 불변(frozen) 신호 DTO. severity, signal_type, message. |
| `make_hades_observer(tick=0)` | HadesObserver 생성. |

---

## 설계 규칙

- **언더월드는 관측·신호 생성만.** 결정·처벌·전이는 호출 측(FSM/Runner) 전용.
- **ConsciousnessSignal 은 frozen.** 위변조 불가.
- **deep_engine 은 덕 타이핑.** `magnetic_ok`, `thermal_ok`, `gravity_ok` 등만 있으면 됨.
- **world_snapshot 은 덕 타이핑.** 지상 타입 import 없음.

---

## 레이어

- **50_DIAGNOSTIC_LAYER** (ENGINE_HUB) — 거시 감시·진단용 엔진으로 배치 가능.

---

## 출처

- CookiieBrain `solar/underworld` 에서 분리.  
- GitHub: [qquartsco-svg/UnderWorld](https://github.com/qquartsco-svg/UnderWorld)
