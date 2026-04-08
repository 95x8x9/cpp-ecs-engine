# SafeCrowd UML 설계 해설 - application-run-results-workflow.puml

대상 파일: `uml/application-run-results-workflow.puml`

## 문서 목적
이 문서는 application 레이어에서 시나리오 작성, 실행, 결과 확인, 비교, 내보내기, 추천 검토가 어떤 흐름으로 이어지는지를 설명한다. Pathfinder 반영안에서는 추천 기능을 유지하되 결과 파이프라인 뒤에 두는 것이 중요했기 때문에, 이 그림은 **화면과 서비스의 연결 순서**를 정리하는 데 목적이 있다.

이 문서는 상세 위젯 배치나 Qt 클래스 설계를 다루지 않는다. 대신 어떤 화면이 어떤 domain 서비스와 연결되고, 어떤 데이터가 다음 화면의 입력이 되는지를 중심으로 읽어야 한다.

---

## `Project Workspace`
- 개요: 프로젝트 열기와 저장의 루트 화면 역할을 하는 작업 공간이다.
- 목적: authoring과 analysis가 같은 프로젝트 맥락 안에서 이어지게 만들기 위해 필요하다.
- 유의사항: 하나의 화면에 모든 기능을 다 넣겠다는 뜻이 아니라, 동일한 프로젝트 컨텍스트를 공유한다는 의미로 읽는 편이 맞다.
- 후속 개선 사항: 최근 프로젝트 목록, artifact health check, recommendation inbox를 붙일 수 있다.

## `Scenario Editor`
- 개요: 레이아웃, 인구, 제어 계획을 수정하는 작성 화면이다.
- 목적: baseline과 variation을 같은 편집 흐름 안에서 관리하기 위해 필요하다.
- 유의사항: 결과 비교와 추천 근거까지 여기서 직접 계산하려 들면 화면 책임이 과해진다.
- 후속 개선 사항: variation diff editor, trigger timeline editor, connector rule editor를 추가할 수 있다.

## `Run Control Panel`
- 개요: 실행, 일시정지, 정지, 반복 실행 설정을 담당하는 제어 패널이다.
- 목적: 작성 단계와 실행 단계의 책임을 분리하면서도 자연스럽게 이어 주기 위해 필요하다.
- 유의사항: run control은 엔진 직접 제어보다 batch orchestration 요청의 진입점으로 보는 편이 좋다.
- 후속 개선 사항: queueable runs, deterministic rerun shortcut, diagnostics launch를 붙일 수 있다.

## `Live Viewport`
- 개요: 실행 중 상태를 재생하고 관찰하는 뷰포트다.
- 목적: 사용자가 현재 run 진행 상황을 실시간으로 확인하게 하기 위해 필요하다.
- 유의사항: live viewport가 결과 비교의 주 진입점이 되면 저장된 artifact 흐름이 약해진다.
- 후속 개선 사항: overlay toggle, hotspot marker, measurement probe를 추가할 수 있다.

## `Run Results Panel`
- 개요: 단일 run 또는 variation 요약을 먼저 보여 주는 결과 패널이다.
- 목적: 사용자를 바로 거대한 비교 화면으로 보내지 않고, 실행 결과를 단계적으로 읽게 하기 위해 필요하다.
- 유의사항: 이 패널은 live engine state가 아니라 저장된 결과 아티팩트를 읽는 것이 중요하다.
- 후속 개선 사항: anomaly list, failed-run recovery, run detail drilldown을 추가할 수 있다.

## `Comparison View`
- 개요: baseline과 대안을 비교하는 핵심 분석 화면이다.
- 목적: 결과 파이프라인의 중심 화면으로서 delta와 근거를 한 번에 보여 주기 위해 필요하다.
- 유의사항: 비교 화면은 계산 화면이 아니라 이미 집계된 결과를 읽고 해석하는 화면이어야 한다.
- 후속 개선 사항: side-by-side heatmap, weighted scorecard, recommendation overlay를 붙일 수 있다.

## `Export Dialog`
- 개요: 보고서와 결과 번들을 내보내는 화면이다.
- 목적: 비교에 사용한 동일한 결과 계약을 외부 공유용 파일로 내보내기 위해 필요하다.
- 유의사항: export가 화면 상태에 종속되기보다 artifact selection을 명시적으로 받는 편이 재현성이 좋다.
- 후속 개선 사항: export preset, markdown/pdf bridge, artifact size warning을 추가할 수 있다.

## `Recommendation Panel`
- 개요: 추천 후보와 근거를 확인하고 시나리오화하는 화면이다.
- 목적: 추천 기능을 결과 파이프라인 뒤에 두어 근거가 분명한 의사결정 흐름을 만들기 위해 필요하다.
- 유의사항: live metric을 바로 추천 입력으로 쓰기보다 cumulative artifact와 comparison delta를 기준으로 삼는 편이 맞다.
- 후속 개선 사항: candidate ranking explanation, partial apply editor, recommendation history를 추가할 수 있다.

## `ScenarioBatchRunner`
- 개요: baseline과 variation 실행을 조정하는 domain 서비스다.
- 목적: application이 여러 run을 직접 관리하지 않고 도메인 배치 실행 계약을 호출하게 하기 위해 필요하다.
- 유의사항: UI에서 batch orchestration 세부 로직을 직접 가지면 테스트가 어려워진다.
- 후속 개선 사항: parallel execution, retry policy, batched cancellation을 추가할 수 있다.

## `SimulationSession`
- 개요: 단일 run 수명주기를 담당하는 domain 세션이다.
- 목적: batch runner와 engine runtime 사이의 도메인 조정 지점으로 쓰기 위해 필요하다.
- 유의사항: session은 결과 저장과 추천 계산까지 모두 떠안는 객체가 되면 안 된다.
- 후속 개선 사항: phase hooks, live diagnostics, step replay token을 붙일 수 있다.

## `ResultRepository`
- 개요: run, variation, cumulative 결과 아티팩트를 저장하고 다시 여는 서비스다.
- 목적: application의 모든 결과 화면이 동일한 저장 진입점을 사용하게 하기 위해 필요하다.
- 유의사항: 각 화면이 파일 경로를 직접 관리하지 않도록 repository를 두는 편이 좋다.
- 후속 개선 사항: lazy loading, cache invalidation, remote artifact sync를 추가할 수 있다.

## `ResultAggregator`
- 개요: 비교용 delta와 summary를 계산하는 domain 서비스다.
- 목적: comparison 화면과 recommendation이 같은 비교 결과를 읽게 하기 위해 필요하다.
- 유의사항: application이 delta 계산을 직접 구현하면 용어와 기준이 화면마다 갈라진다.
- 후속 개선 사항: normalized compare, worst-case compare, sensitivity analysis를 붙일 수 있다.

## `AlternativeRecommendationService`
- 개요: 결과 근거를 바탕으로 운영 대안 후보를 생성하는 domain 서비스다.
- 목적: 추천 기능을 UI 헬퍼가 아니라 도메인 정책 서비스로 유지하기 위해 필요하다.
- 유의사항: 추천은 결과 파이프라인 뒤에 위치해야 하며, 안정화된 cumulative artifact를 읽어야 한다.
- 후속 개선 사항: recommendation scoring, scenario auto-generation presets, analyst feedback loop를 추가할 수 있다.

## `EngineRuntime`
- 개요: 실행 중 playback을 담당하는 엔진 진입점이다.
- 목적: application이 직접 ECS를 만지지 않고 세션을 통해 실행을 제어하게 하기 위해 필요하다.
- 유의사항: recommendation이나 compare는 runtime와 직접 연결하지 않는 편이 맞다.
- 후속 개선 사항: replay control, live diagnostics stream, snapshot export를 추가할 수 있다.

## `IRenderBridge`
- 개요: runtime 상태를 뷰포트로 넘기는 좁은 인터페이스다.
- 목적: live viewport가 엔진 세부 구현을 직접 모르도록 만들기 위해 필요하다.
- 유의사항: Qt 구체 타입을 이 인터페이스에 새기지 않는 편이 좋다.
- 후속 개선 사항: double buffer, replay source switch, debug overlay channel을 추가할 수 있다.
