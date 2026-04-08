# SafeCrowd UML 설계 해설 - domain-control-model.puml

대상 파일: `uml/domain-control-model.puml`

## 문서 목적
이 문서는 SafeCrowd의 제어 모델을 설명한다. Pathfinder 반영안에서 요구한 핵심은 운영 이벤트를 단순 플래그 모음으로 두지 않고, `행동`, `트리거`, `occupant tag`, `출구 선택 비용`을 함께 갖는 제어 구조로 올리는 것이다.

이 그림은 UI에서 어떤 값을 넣을지를 그대로 보여주는 화면 명세가 아니라, **도메인에서 어떤 제어 개념을 분리해 가져가야 하는가**를 정리하는 구조도다.

---

## `ControlPlan`
- 개요: 시나리오 안에서 적용되는 운영 제어의 루트 계약이다.
- 목적: 이벤트, 태그, 행동, route choice binding을 한 곳에서 일관되게 관리하기 위해 필요하다.
- 유의사항: 시나리오 정의 객체 안에 제어 관련 필드를 흩뿌리기보다 `ControlPlan`으로 묶어 두는 편이 비교와 저장이 쉽다.
- 후속 개선 사항: 승인 상태, 운영자 권한, 추천에서 생성된 plan provenance를 추가할 수 있다.

## `OperationalEvent`
- 개요: 특정 시점이나 상태에서 발동되는 운영 변화 단위다.
- 목적: 출구 개방/폐쇄, 접근 제한, source rate 조정, 행동 교체 같은 변화를 하나의 실행 단위로 다루기 위해 필요하다.
- 유의사항: 이벤트는 "무엇을 바꾸는가"를 나타내고, 실제 동작 규칙은 binding과 policy가 담당하도록 나누는 편이 좋다.
- 후속 개선 사항: rollback 정책, 우선순위 충돌 해결, event provenance를 붙일 수 있다.

## `Trigger`
- 개요: 이벤트가 언제 발동되는지 정의하는 조건 객체다.
- 목적: 시간 기반과 상태 기반 발동을 같은 상위 구조 아래에서 관리하기 위해 필요하다.
- 유의사항: trigger를 이벤트 내부의 단순 timestamp 필드로 축소하면, 상태 기반 발동을 넣는 순간 구조가 깨진다.
- 후속 개선 사항: 다중 조건 조합, debounce, operator acknowledgment를 확장할 수 있다.

## `TriggerCondition`
- 개요: trigger가 평가하는 실제 조건의 추상 타입이다.
- 목적: 조건 종류가 늘어나도 trigger 루트 구조를 유지하기 위해 필요하다.
- 유의사항: 너무 이른 단계에 복잡한 condition DSL을 넣기보다, 먼저 시간/상태 두 축을 명확히 분리하는 편이 낫다.
- 후속 개선 사항: event chaining, perception delay, composite condition을 추가할 수 있다.

## `TimeTriggerCondition`
- 개요: 시뮬레이션 시간 기준으로 발동되는 조건이다.
- 목적: 순차 퇴장, 일정 시간 후 출구 개방 같은 기본 운영 시나리오를 표현하기 위해 필요하다.
- 유의사항: wall-clock과 simulation time을 혼동하지 않는 편이 중요하다.
- 후속 개선 사항: 반복 schedule, relative timing, phase alignment를 추가할 수 있다.

## `StateTriggerCondition`
- 개요: 측정값이나 상태 변화 기준으로 발동되는 조건이다.
- 목적: 특정 구역 혼잡도, queue 길이, 위험 신호에 따라 운영 계획을 바꾸기 위해 필요하다.
- 유의사항: trigger용 상태는 결과 요약과 별도로, 실행 중 평가 가능한 측정값이어야 한다.
- 후속 개선 사항: 다중 metric threshold, hysteresis, guard condition을 확장할 수 있다.

## `OccupantTag`
- 개요: 특정 인원 집합을 선택하는 도메인 태그다.
- 목적: 행동 전환이나 route choice 정책을 전체 인원에 일괄 적용하지 않고 부분 집합에 적용하기 위해 필요하다.
- 유의사항: UI 편의용 label과 실행용 selection rule을 구분하는 편이 좋다.
- 후속 개선 사항: profile 기반 자동 태깅, 공간 기반 태깅, runtime reassignment를 추가할 수 있다.

## `Behavior`
- 개요: 인원이 따르는 상위 행동 정의다.
- 목적: `Wait`, `Goto`, 목표 변경, 안내 대기 같은 로직을 이벤트와 분리해 재사용 가능한 구조로 만들기 위해 필요하다.
- 유의사항: 행동 하나가 모든 경우를 다 품기 시작하면 step 구조 의미가 사라진다.
- 후속 개선 사항: fallback behavior, failure handling, group-aware behavior를 확장할 수 있다.

## `BehaviorStep`
- 개요: 행동을 구성하는 세부 단계다.
- 목적: 행동을 단일 state가 아니라 시퀀스로 다루기 위해 필요하다.
- 유의사항: completion rule과 target selector를 분리하지 않으면 step 전이 조건이 금방 불명확해진다.
- 후속 개선 사항: branch step, conditional step, external guidance hook을 붙일 수 있다.

## `BehaviorBinding`
- 개요: 어떤 태그 집합에 어떤 행동을 적용할지 정의하는 연결 객체다.
- 목적: 이벤트가 직접 behavior를 몰라도 binding 교체만으로 제어 흐름을 바꿀 수 있게 하기 위해 필요하다.
- 유의사항: binding은 계획 수준 구조이고, 이벤트는 그 binding을 활성화하거나 교체하는 방식으로 접근하는 편이 단순하다.
- 후속 개선 사항: priority binding, layered override, recommendation-generated binding을 추가할 수 있다.

## `RouteChoiceBinding`
- 개요: 특정 태그 집합에 route choice policy를 연결하는 구조다.
- 목적: 출구 선택 비용을 인원 그룹별로 다르게 적용하기 위해 필요하다.
- 유의사항: 행동 전환과 route choice를 같은 binding으로 합치면 목적이 다른 두 축이 엉킨다.
- 후속 개선 사항: context-specific override, guidance-aware binding을 붙일 수 있다.

## `RouteChoicePolicy`
- 개요: 경로 재선택과 출구 선택의 상위 정책이다.
- 목적: 단순 nearest exit를 넘어서 queue와 잔여 경로 비용까지 포함한 의사결정을 도메인에서 정의하기 위해 필요하다.
- 유의사항: policy는 엔진 pathfinder 자체보다 상위 레벨의 의사결정 계약으로 유지하는 편이 맞다.
- 후속 개선 사항: learning policy, uncertainty model, recommendation hint integration을 확장할 수 있다.

## `PathCostModel`
- 개요: route choice policy가 사용하는 비용 조합 모델이다.
- 목적: `Current Room Travel Time`, `Queue Time`, `Remaining Route Cost` 같은 요소를 합성하기 위해 필요하다.
- 유의사항: 계산 구현은 engine boundary에서 하더라도, 어떤 비용을 쓰는지는 domain 계약으로 남겨야 비교가 가능하다.
- 후속 개선 사항: visibility penalty, congestion risk weight, connector penalty를 추가할 수 있다.

## `CostComponent`
- 개요: 비용 모델을 이루는 개별 비용 요소다.
- 목적: 비용 구조를 가중치 기반 조합으로 유지해 정책 변경을 더 쉽게 만들기 위해 필요하다.
- 유의사항: component 종류가 폭발하면 관리가 어려워지므로, 먼저 몇 개의 핵심 요소에 집중하는 편이 낫다.
- 후속 개선 사항: normalized cost, contextual cost, operator bias term을 추가할 수 있다.

## `AccessRuleChange`
- 개요: door나 connector 접근 규칙을 바꾸는 이벤트 효과다.
- 목적: 출구 개방/폐쇄, 일방통행, 접근 제한을 제어 모델 안에서 명시적으로 다루기 위해 필요하다.
- 유의사항: layout 기본 속성과 runtime access override를 분리하는 편이 맞다.
- 후속 개선 사항: timed reopening, phased release, accessibility exceptions를 붙일 수 있다.

## `SourceRateChange`
- 개요: 동적 인원 유입 source의 유입률을 조정하는 이벤트 효과다.
- 목적: source 기반 유입을 운영 계획과 연결하기 위해 필요하다.
- 유의사항: source 자체 정의와 source rate 변경 이벤트를 같은 객체로 합치지 않는 편이 좋다.
- 후속 개선 사항: burst pattern, gated entry, queue-linked ingress control을 추가할 수 있다.
