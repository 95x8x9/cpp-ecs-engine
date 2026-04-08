# SafeCrowd UML 설계 해설 - engine-routing-and-connectors.puml

대상 파일: `uml/engine-routing-and-connectors.puml`

## 문서 목적
이 문서는 room, door, connector, 장애물, path cost, flow measurement가 domain과 engine 경계에서 어떻게 이어지는지를 설명한다. Pathfinder 반영안의 핵심 중 하나가 connector와 경로 비용, 동적 obstacle, 측정 구조를 한 번에 정리하는 것이었기 때문에, 이 그림은 **레이아웃 의미 모델이 엔진 실행 자원으로 변환되는 과정**에 초점을 둔다.

이 그림은 엔진 내부 pathfinding 알고리즘 상세를 설명하는 문서가 아니다. 오히려 domain이 어떤 형태로 엔진에 topology와 규칙을 넘기고, 엔진은 어떤 실행용 자원으로 그것을 소비하는지를 보여준다.

---

## `FacilityLayout`
- 개요: domain에서 확정한 실행용 공간 계약이다.
- 목적: room, door, connector, 장애물, 측정 구역 정의를 한 곳에 모아 routing 입력의 기준으로 사용하기 위해 필요하다.
- 유의사항: 엔진이 raw import geometry를 직접 해석하게 두면 계층 경계가 무너진다.
- 후속 개선 사항: 층별 topology partition, layout versioning, import trace 연결을 확장할 수 있다.

## `ConnectorSpec`
- 개요: 계단, 램프, 에스컬레이터, 보행로를 포괄하는 connector 정의다.
- 목적: connector 종류를 공통 모델로 관리하면서 traversal rule만 다르게 적용하기 위해 필요하다.
- 유의사항: connector 종류별 전용 엔진 코드로 바로 분기하기보다, spec과 modifier 조합으로 시작하는 편이 단순하다.
- 후속 개선 사항: lane semantics, assisted traversal rule, bi-directional capacity 모델을 붙일 수 있다.

## `ConnectorModifier`
- 개요: connector 통과 비용과 접근 규칙을 바꾸는 modifier다.
- 목적: 속도 계수, queue bias, 접근 허용 여부 같은 차이를 connector 공통 구조 안에서 표현하기 위해 필요하다.
- 유의사항: modifier는 기본 topology에 더해지는 보정 규칙이지, topology 자체를 대체하는 구조가 아니다.
- 후속 개선 사항: 시간대별 modifier, profile-aware modifier, event-scoped modifier를 추가할 수 있다.

## `StaticObstruction`
- 개요: topology bake 전에 반영되는 정적 장애물이다.
- 목적: room과 edge의 유효 폭, clearance, 통과 가능 여부를 사전에 결정하기 위해 필요하다.
- 유의사항: 정적 obstruction까지 runtime field로 미루면 매번 path graph를 재해석해야 한다.
- 후속 개선 사항: visibility blocking, risk hotspot prior, geometry simplification 정보를 추가할 수 있다.

## `DynamicObstacleRule`
- 개요: 실행 중 활성화되거나 해제되는 동적 차단 규칙이다.
- 목적: 운영 이벤트나 상태 변화에 따른 일시적 차단을 topology 재생성 없이 다루기 위해 필요하다.
- 유의사항: rule은 activation 조건과 영향을 정의하고, 실제 적용은 engine runtime 자원이 담당하는 편이 맞다.
- 후속 개선 사항: trigger linkage, multi-stage closure, dynamic lane reservation을 붙일 수 있다.

## `MeasurementRegionSpec`
- 개요: 측정 구역 정의다.
- 목적: heatmap, room history, region series의 수집 지점을 runtime 전에 고정하기 위해 필요하다.
- 유의사항: 결과 시계열과 측정 정의를 같은 객체로 두지 말고 분리해야 한다.
- 후속 개선 사항: region template library, derived metrics, export naming rules를 확장할 수 있다.

## `RouteChoicePolicy`
- 개요: 상위 수준의 출구 선택 정책이다.
- 목적: 단순 최단거리 대신 queue와 잔여 비용을 포함한 의사결정을 도메인에서 정의하기 위해 필요하다.
- 유의사항: 엔진은 비용 계산과 path query를 담당하되, 어떤 비용을 쓸지는 domain 계약에서 내려와야 비교가 가능하다.
- 후속 개선 사항: visibility penalty, guidance bias, risk-aware reroute policy를 추가할 수 있다.

## `PathCostModel`
- 개요: 경로 비용 요소의 조합 모델이다.
- 목적: `travel`, `queue`, `remaining route cost`를 명시적으로 합성하기 위해 필요하다.
- 유의사항: 비용 요소를 엔진 내부 상수로 숨기면 시나리오 차이와 추천 근거가 불투명해진다.
- 후속 개선 사항: connector-specific penalty, uncertainty band, operator override weight를 붙일 수 있다.

## `RoutingTopologySnapshot`
- 개요: domain 레이아웃을 엔진이 읽을 수 있는 안정된 routing 스냅샷으로 컴파일한 결과다.
- 목적: 엔진이 domain 객체 그래프에 직접 의존하지 않고 실행용 topology 자원을 받게 하기 위해 필요하다.
- 유의사항: 경계 모델이 불안정하면 엔진과 domain이 다시 강결합된다.
- 후속 개선 사항: incremental rebuild, topology diff, serialized cache를 확장할 수 있다.

## `EngineWorld`
- 개요: routing 관련 runtime 자원을 담는 엔진 파사드다.
- 목적: topology graph, obstacle field, measurement service 같은 자원을 등록하고 시스템이 접근하게 한다.
- 유의사항: domain 객체를 엔진 월드 안에 그대로 집어넣는 식의 편법은 피하는 편이 좋다.
- 후속 개선 사항: read-only routing view, diagnostics resource, replay snapshot을 붙일 수 있다.

## `NavigationGraph`
- 개요: path query에 사용하는 실행용 그래프다.
- 목적: room, door, connector를 path planner가 사용할 수 있는 노드/에지 구조로 제공하기 위해 필요하다.
- 유의사항: graph는 bake된 topology 자원이고, 동적 변경은 별도 field나 evaluator가 담당해야 한다.
- 후속 개선 사항: hierarchical routing, cached shortest path, multi-floor acceleration을 추가할 수 있다.

## `TravelCostEvaluator`
- 개요: 현재 조건에서 경로 비용을 계산하는 평가기다.
- 목적: topology, dynamic penalties, route choice weights를 합쳐 실제 reroute 비용을 계산하기 위해 필요하다.
- 유의사항: evaluator가 topology ownership까지 가지면 책임이 과도해진다.
- 후속 개선 사항: congestion forecast, familiarity penalty, stochastic choice model을 붙일 수 있다.

## `DynamicObstacleField`
- 개요: 현재 활성화된 동적 obstacle과 penalty 상태를 가진 runtime 자원이다.
- 목적: topology 전체를 다시 굽지 않고도 차단과 cost penalty를 적용하기 위해 필요하다.
- 유의사항: 영구 레이아웃 데이터와 실행 중 상태를 분리하는 것이 핵심이다.
- 후속 개선 사항: time-window cache, per-tag blocking, safety buffer expansion을 추가할 수 있다.

## `ConnectorTraversalState`
- 개요: connector의 대기열, 처리량, 통과 상태를 담는 실행 자원이다.
- 목적: path cost와 measurement가 connector 상태를 함께 읽을 수 있게 하기 위해 필요하다.
- 유의사항: 문 자체의 door throughput과 connector traversal state를 같은 측정값으로 취급하지 않는 편이 좋다.
- 후속 개선 사항: queue length estimation, connector occupancy history, disabled lane state를 붙일 수 있다.

## `PathPlannerSystem`
- 개요: path query와 reroute 갱신을 수행하는 시스템이다.
- 목적: 현재 routing graph와 비용 평가 결과를 바탕으로 인원 경로를 갱신하기 위해 필요하다.
- 유의사항: domain 정책 판단까지 이 시스템에 넣기보다, domain이 준 policy를 비용 모델로 해석하는 수준에 머무는 편이 좋다.
- 후속 개선 사항: batch replanning, partial reroute, congestion-aware trigger를 확장할 수 있다.

## `FlowMeasurementService`
- 개요: door, room, measurement region 단위의 흐름 샘플을 수집하는 경계 서비스다.
- 목적: 나중에 `DoorHistory`, `RoomHistory`, `MeasurementRegionSeries`로 변환될 원시 측정 샘플을 만들기 위해 필요하다.
- 유의사항: 결과 아티팩트 구조를 알 필요는 없고, sample emission까지만 맡는 편이 경계가 깔끔하다.
- 후속 개선 사항: adaptive sampling, diagnostic counters, on-demand probes를 붙일 수 있다.

## `MeasurementSample`
- 개요: 특정 시점의 측정값 묶음이다.
- 목적: runtime 샘플과 domain 결과 아티팩트 사이를 이어 주는 중간 단위로 사용하기 위해 필요하다.
- 유의사항: sample은 domain-friendly result object가 아니라 저수준 관측값에 가깝다.
- 후속 개선 사항: compressed sample block, export token, replay-aligned sample format을 추가할 수 있다.
