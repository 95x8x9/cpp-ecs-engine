# SafeCrowd UML 설계 해설 - domain-result-artifacts.puml

대상 파일: `uml/domain-result-artifacts.puml`

## 문서 목적
이 문서는 SafeCrowd 결과 아티팩트 모델을 설명한다. Pathfinder 반영안에서 결과 파이프라인은 단순 화면 표시가 아니라 `run`, `variation`, `cumulative` 단위를 분리해 저장하고, 추천과 내보내기가 공통 결과 계약을 읽도록 정리하는 것이 핵심이다.

이 그림은 결과를 파일 포맷 수준으로 정의하는 문서가 아니라, **도메인에서 어떤 결과 단위를 중심으로 저장하고 집계할 것인가**를 고정하는 구조도다.

---

## `RunResult`
- 개요: 한 번의 실행 결과를 나타내는 최소 저장 단위다.
- 목적: 반복 실행이 있더라도 먼저 단일 run 산출물을 명시적으로 남기기 위해 필요하다.
- 유의사항: variation 요약과 run 결과를 한 객체로 합치면 재현성 검증과 실패 run 처리 정책이 꼬인다.
- 후속 개선 사항: run provenance, replay token, diagnostics summary를 붙일 수 있다.

## `RunMetadata`
- 개요: run 결과에 붙는 실행 메타데이터다.
- 목적: scenario id, execution config, 시작/종료 시간 같은 실행 문맥을 결과와 함께 보존하기 위해 필요하다.
- 유의사항: 결과 숫자와 메타데이터를 분리해 두는 편이 비교와 export에서 다루기 쉽다.
- 후속 개선 사항: environment hash, build version, import artifact reference를 추가할 수 있다.

## `MetricSummary`
- 개요: 핵심 요약 지표 묶음이다.
- 목적: 총 대피 시간, 퍼센타일 시간, 최대 밀도, hotspot 수 같은 의사결정용 지표를 단일 계약으로 제공하기 위해 필요하다.
- 유의사항: raw history만 저장하고 summary를 계산하지 않으면 비교와 추천 입력이 과하게 무거워진다.
- 후속 개선 사항: normalized score, confidence interval, scenario delta cache를 붙일 수 있다.

## `DoorHistory`
- 개요: door 또는 출구 단위의 시간 이력이다.
- 목적: 처리량, queue, 개방 상태 변화를 결과 아티팩트로 남기기 위해 필요하다.
- 유의사항: 문별 이력은 heatmap과 다른 성격이므로 room history와 합치지 않는 편이 좋다.
- 후속 개선 사항: connector history와의 결합 뷰, event markers, export down-sampling을 추가할 수 있다.

## `RoomHistory`
- 개요: room 단위의 밀도, 혼잡, 체류 이력이다.
- 목적: 병목, 정체, 탈출 지연 축을 공간 단위로 요약하기 위해 필요하다.
- 유의사항: room history는 geometry-aware 결과이므로 레이아웃 버전과 연결되는 기준이 필요하다.
- 후속 개선 사항: room category rollup, hotspot episode extraction, occupancy animation cache를 붙일 수 있다.

## `MeasurementRegionSeries`
- 개요: 측정 구역 단위 시계열 결과다.
- 목적: predefined measurement region을 기준으로 반복 비교 가능한 결과를 만들기 위해 필요하다.
- 유의사항: region geometry와 결과 시계열을 혼동하지 않도록, spec은 layout 쪽에 두고 series는 결과 쪽에 두는 구조가 맞다.
- 후속 개선 사항: derived metric pack, percentile window, export preset을 추가할 수 있다.

## `OccupantHistory`
- 개요: 개별 인원 추적 이력이다.
- 목적: 상세 디버깅이나 특정 분석 모드에서 필요할 수 있는 per-occupant trace를 선택적으로 제공하기 위해 필요하다.
- 유의사항: Pathfinder 반영안 기준으로 이 구조는 선택 사항이어야 한다. 기본 결과 파이프라인이 이것에 의존하면 저장 비용과 처리 비용이 과도해진다.
- 후속 개선 사항: compressed trace, sampling mode, privacy-safe export 정책을 붙일 수 있다.

## `ArtifactIndex`
- 개요: 실제 저장 위치와 export manifest를 가리키는 인덱스다.
- 목적: 결과 아티팩트가 여러 파일이나 저장 경로로 분리돼도 상위 서비스가 공통 진입점으로 접근하게 하기 위해 필요하다.
- 유의사항: 결과 내용을 전부 다시 중복 저장하는 객체가 아니라 참조와 식별을 담당하는 구조로 유지하는 편이 좋다.
- 후속 개선 사항: content hash, lazy loading, retention policy를 추가할 수 있다.

## `VariationSummary`
- 개요: 동일 variation에 속한 여러 run을 집계한 결과다.
- 목적: 반복 실행과 Monte Carlo 성격의 집계를 단일 run과 분리하기 위해 필요하다.
- 유의사항: 평균값만 남기면 최악 run이나 분산 정보를 놓치기 쉽다.
- 후속 개선 사항: percentile band, worst-case pick, confidence interval을 붙일 수 있다.

## `ScenarioComparison`
- 개요: baseline과 여러 alternative variation의 비교 결과다.
- 목적: 기준 대비 delta를 별도 구조로 유지해 비교 화면과 추천 근거가 같은 계약을 읽게 하기 위해 필요하다.
- 유의사항: 비교 계산을 매 화면마다 즉석에서 다시 하지 말고, domain 결과 객체로 승격시키는 편이 낫다.
- 후속 개선 사항: multi-baseline compare, weighted scorecard, sensitivity view를 추가할 수 있다.

## `CumulativeArtifact`
- 개요: export, 비교, 추천의 공통 입력이 되는 canonical 결과 번들이다.
- 목적: 결과 파이프라인의 상위 소비자가 run, variation, comparison을 제각각 조합하지 않게 하기 위해 필요하다.
- 유의사항: 모든 raw sample을 또 한 번 복제하는 구조가 아니라, 핵심 요약과 참조를 담는 묶음으로 유지하는 편이 좋다.
- 후속 개선 사항: versioned schema, partial export bundle, recommendation evidence package를 확장할 수 있다.
