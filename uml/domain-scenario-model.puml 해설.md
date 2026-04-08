# SafeCrowd UML 설계 해설 - domain-scenario-model.puml

대상 파일: `uml/domain-scenario-model.puml`

## 문서 목적
이 문서는 SafeCrowd의 시나리오 정의 모델을 설명한다. 기존 상위 구조 문서에서 `ScenarioDefinition` 하나로 묶어 두었던 입력 계약을, Pathfinder 반영안 기준에 맞춰 `FacilityLayout`, `PopulationSpec`, `EnvironmentState`, `ControlPlan`, `ExecutionConfig`로 분해해 보여주는 것이 목적이다.

이 그림은 구현 클래스 목록이라기보다 **도메인 계약의 경계**를 정리한 것이다. 따라서 UI 세부 화면이나 엔진 내부 자료구조보다, 어떤 입력이 어느 객체에서 책임져야 하는지를 읽는 데 초점을 둔다.

---

## `FacilityLayout`
- 개요: 검토와 보정을 거쳐 실행 가능 상태로 확정된 시설 레이아웃 계약이다.
- 목적: 시나리오가 사용하는 공간 기준을 `Room`, `Door`, `Connector`, 장애물, 측정 구역까지 포함한 명시적 구조로 고정하기 위해 필요하다.
- 유의사항: import 단계의 raw geometry를 그대로 들고 오지 말고, 실행과 검증에 필요한 토폴로지 수준으로 정규화된 결과만 노출하는 편이 좋다.
- 후속 개선 사항: 층간 연결의 상세 modifier, 측정 구역 preset, 공간 버전 비교 기능을 붙일 수 있다.

## `Floor`
- 개요: 레이아웃 안의 층 단위 구획이다.
- 목적: room과 connector가 어느 층에 속하는지 명확히 하고, 층간 연결을 다룰 근거를 제공한다.
- 유의사항: 2D import가 먼저여도 층 정보를 완전히 배제하면 나중에 connector 모델을 다시 뜯어고쳐야 한다.
- 후속 개선 사항: 층별 피난 정책, 수직 이동 비용, 층별 시야 상태를 연결할 수 있다.

## `Room`
- 개요: 인원 배치, 측정, 경로 비용 계산의 기본 공간 단위다.
- 목적: 시작 배치, 대기, 혼잡, 체류를 모두 같은 공간 단위에서 비교 가능하게 만들기 위해 필요하다.
- 유의사항: 단순 렌더링 polygon과 실행용 room 개념을 섞지 않는 편이 좋다.
- 후속 개선 사항: room category별 기본 파라미터, 용도별 preset을 둘 수 있다.

## `Door`
- 개요: 방과 방, 또는 방과 외부를 잇는 출입 경계다.
- 목적: 처리량, 대기, 통과 기록, 출구 선택 비용 계산의 핵심 기준점으로 사용한다.
- 유의사항: connector와 door는 모두 연결 요소지만 책임이 다르므로 같은 타입으로 뭉개지 않는 편이 좋다.
- 후속 개선 사항: 일방통행, release timing, 운영 이벤트 연동 속성을 확장할 수 있다.

## `Connector`
- 개요: 계단, 램프, 에스컬레이터, 보행로 같은 구간 연결 모델이다.
- 목적: Pathfinder 반영안에서 요구한 공통 connector 모델과 modifier 구조를 한 자리에 고정하기 위해 필요하다.
- 유의사항: connector 종류별로 클래스를 완전히 분리하기보다 공통 속성과 modifier 조합으로 시작하는 편이 단순하다.
- 후속 개선 사항: 보행 약자 제한, 역방향 제한, queue bias 같은 상세 modifier를 붙일 수 있다.

## `ConnectorModifier`
- 개요: connector 통과 비용과 사용 가능 조건을 바꾸는 보정 규칙이다.
- 목적: 속도 저하, 접근 제한, 대기 편향 같은 차이를 connector 공통 구조 안에서 표현하기 위해 필요하다.
- 유의사항: modifier가 운영 이벤트와 영구 속성을 동시에 품기 시작하면 책임이 모호해지므로, 기본 속성과 runtime override를 구분하는 편이 좋다.
- 후속 개선 사항: 시간대별 modifier, occupant profile별 차등 적용을 확장할 수 있다.

## `StaticObstruction`
- 개요: 레이아웃에 고정된 장애물이나 차단 요소다.
- 목적: import와 수동 보정 단계에서 정적 obstacle을 명시적으로 다루기 위해 필요하다.
- 유의사항: 정적 obstruction과 실행 중 바뀌는 obstacle 규칙을 같은 구조로 두면 검증과 bake 책임이 흐려진다.
- 후속 개선 사항: barrier category, clearance heuristic, 시야 차폐 효과를 연결할 수 있다.

## `DynamicObstacleRule`
- 개요: 운영 계획이나 실행 상태에 따라 활성화되는 동적 차단 규칙이다.
- 목적: 런타임에 바뀌는 obstacle을 레이아웃 영구 데이터와 분리해 표현하기 위해 필요하다.
- 유의사항: rule은 "언제 어디가 막히는가"까지만 정의하고, 실제 적용은 control model이나 engine boundary에서 처리하는 편이 맞다.
- 후속 개선 사항: trigger 연결, source 제어, 구역 봉쇄 preset을 확장할 수 있다.

## `MeasurementRegionSpec`
- 개요: heatmap과 요약 지표를 수집할 측정 구역 정의다.
- 목적: 결과 아티팩트에서 `MeasurementRegionSeries`를 만들 기준을 레이아웃 단계에서 명시하기 위해 필요하다.
- 유의사항: 결과 모델에서 생기는 측정값과, 레이아웃에 박아 두는 측정 정의를 혼동하지 않는 편이 좋다.
- 후속 개선 사항: region template, 다중 metric preset, export label 규칙을 추가할 수 있다.

## `ScenarioDefinition`
- 개요: baseline 또는 단일 시나리오의 authoring aggregate다.
- 목적: 상위 구조 문서에서 이미 사용 중인 `ScenarioDefinition` 용어를 유지하면서, 내부 계약을 더 세밀하게 나누기 위해 필요하다.
- 유의사항: 시나리오 자체가 모든 저수준 값을 직접 품는 거대한 객체가 되지 않게 조심해야 한다.
- 후속 개선 사항: 저장 버전, provenance, 추천 시나리오 출처 메타데이터를 확장할 수 있다.

## `ScenarioVariation`
- 개요: 기준 시나리오에서 달라진 부분만 기록하는 variation 단위다.
- 목적: 비교 분석과 반복 실행에서 baseline 대비 delta를 명확히 관리하기 위해 필요하다.
- 유의사항: variation을 baseline 전체 복사본으로 두면 저장, 비교, 추천 근거 관리가 금방 지저분해진다.
- 후속 개선 사항: layered override, scenario family, recommendation lineage를 붙일 수 있다.

## `PopulationSpec`
- 개요: 초기 배치와 동적 유입을 포함하는 인구 입력 계약이다.
- 목적: "처음 어디에 몇 명이 있는가"와 "시간이 지나며 어디서 유입되는가"를 같은 상위 계약 아래 두되 별도 구조로 유지하기 위해 필요하다.
- 유의사항: 인원 수 총합만 남기는 구조로 시작하면 source 기반 유입을 추가할 때 다시 분해해야 한다.
- 후속 개선 사항: group distribution, arrival curve preset, demographic mix를 확장할 수 있다.

## `PopulationProfile`
- 개요: 이동 특성, 분포 파라미터, connector 사용 제약을 가진 프로필 정의다.
- 목적: 속도, 가속, 간격 분포와 접근 제약을 scenario 입력 계약 안에 고정하기 위해 필요하다.
- 유의사항: 엔진용 세부 파라미터를 그대로 노출하기보다 프로필 수준에서 관리하고 calibration으로 내리는 편이 좋다.
- 후속 개선 사항: profile inheritance, behavior affinity, assisted evacuation profile을 붙일 수 있다.

## `InitialPlacement`
- 개요: 실행 시작 시점의 배치 규칙이다.
- 목적: baseline 배치와 variation 차이를 room anchor 기준으로 분명하게 남기기 위해 필요하다.
- 유의사항: source와 같은 구조로 처리하면 초기 상태와 시간 기반 유입의 의미가 섞인다.
- 후속 개선 사항: density cap, spatial sampling rule, batch variation support를 추가할 수 있다.

## `DynamicSource`
- 개요: 시간에 따라 인원이 유입되는 source 정의다.
- 목적: Pathfinder 반영안의 동적 인원 유입 요구를 도메인 모델에서 직접 수용하기 위해 필요하다.
- 유의사항: source는 단순 카운터가 아니라 schedule과 ingress rate를 가져야 한다.
- 후속 개선 사항: trigger 기반 source control, queue coupling, ingress profile library를 붙일 수 있다.

## `EnvironmentState`
- 개요: 시야, 친숙도, 안내 신호 같은 환경 상태 입력이다.
- 목적: 기본 시야/길찾기 저하를 1차 확장 범위에서 다루기 위한 입력 계약을 분리하기 위해 필요하다.
- 유의사항: smoke/FED/FDS 연동은 나중 단계이므로, 초기 구조는 기본 visibility/familiarity/guidance 수준에서 멈추는 편이 낫다.
- 후속 개선 사항: environmental time series, smoke coupling, hazard field reference를 추가할 수 있다.

## `ControlPlan`
- 개요: 운영 이벤트, 행동 전환, route choice binding을 묶는 제어 계약이다.
- 목적: 운영 계획을 단순 on/off 플래그가 아니라 실행 가능한 제어 모델로 만들기 위해 필요하다.
- 유의사항: control model의 세부 구조는 별도 UML에서 더 자세히 다루므로, 여기서는 입력 계약 역할에 집중한다.
- 후속 개선 사항: approval workflow, operator role, recommendation provenance를 확장할 수 있다.

## `ExecutionConfig`
- 개요: 시간 제한, 샘플링 주기, base seed, 반복 횟수를 포함한 실행 계약이다.
- 목적: 시나리오 내용과 실험 조건을 분리하고 재현성을 확보하기 위해 필요하다.
- 유의사항: 반복 실행, variation, cumulative 결과를 만들수록 execution 설정은 더 명시적으로 남겨야 한다.
- 후속 개선 사항: Monte Carlo policy, strict determinism mode, sampling/export preset을 붙일 수 있다.
