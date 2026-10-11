# CHANGELOG

형식: `날짜 — 변경 유형 — 대상 — 요지`. 변경 유형: 신규 / 상태전이 / 수정 / 삭제 / 구조

## 2026-10-11 (3차) — 전체 Part 본문 작성

- 상태전이 — `00-overview/01, 02, 03, 04` stub/draft → **review**. 레퍼런스 아키텍처에 레이어 간 계약 ①②③ 정의, 종합 결정 트리 Q1~Q15 + 단계별 기본값 34행, 트렌드 레이더 28항목, 용어집
- 상태전이 — `02-data-pipeline/01~10, 99` stub → **review** (11개)
- 상태전이 — `03-agent-workflow-rag/01~14, 99` stub → **review** (15개)
- 상태전이 — `04-dynamic-ui/01~10, 99` stub → **review** (11개). 성숙도는 전부 "시험"
- 상태전이 — `05-cross-cutting/01~05` stub → **review** (5개). 각 페이지에 레이어 간 정합성 검토 체크리스트
- 상태전이 — `06-trends/*` stub → **review** (8개). trend 템플릿, `first_seen` 추가. 결정 모델·에이전트 메모리는 "관찰", 나머지 "시험"
- 신규 — HTML 10종: `data-pipeline-architecture`, `kg-construction-pipeline`, `agent-workflow-architecture`, `rag-pattern-evolution`, `agent-design-patterns`, `multi-agent-topologies`, `dynamic-ui-architecture`, `genui-streaming-sequence`, `decision-tree`(인터랙티브), `trend-radar`
- 신규 — `_meta/research-notes-2026-10.md`: 사실·검증 수준 단일 출처 + "추가 조사 필요" 14건
- 수정 — `06-trends/00-overview.md` 잠정 레이더 표기를 페이지 프론트매터와 일치(결정 모델 "관찰", 컨텍스트 엔지니어링 "시험")
- 수정 — 결정 트리(0/02)가 99 매트릭스와 페이지 선택 가이드의 불일치 4건(3/02, 3/06, 3/07, 2/05)을 99 기준으로 통일하고 각주로 명기
- 주의 — 레이더에서 `[검증]` 없이 "채택"인 항목은 하이브리드 검색 기본값화(T23) 하나. 0/03 한계에 명시
- 작업 방식 — Part 2~6은 Part별 병렬 집필 후 전수 검사(인용 금지 수치, 벤더 수치 검증 표기, 비교표 부적합 열 공란, 링크) 통과

## 2026-10-11 (2차)

- 상태전이 — `01-foundations/01~06` stub → **review**. 본문·비교표·한계·트렌드 적용·참고 문헌 작성 완료. `[실측]` 없음
- 수정 — 조사 반영: MCP 2026-07-28 개정(무상태 코어, Multi Round-Trip, Tasks·Apps 확장), A2A Linux Foundation 이관(2025-06), A2UI v0.9(2026-04-17), AG-UI .NET SDK(2026-09-25), OTel GenAI 컨벤션 Development 상태 유지 및 저장소 분리(v1.42.0)
- 수정 — 결정 모델 서술 통일: Jev(결정 전용·비자기회귀) vs OpenAI Decisions API(gpt-6-luna 출력 제약 엔드포인트) 구분을 01·02·06에 반영

## 2026-10-11 (1차)

- 구조 — 저장소 초기화. `WIKI_PLAN.md`(구조 설계안) 작성
- 신규 — `README.md`, `_meta/writing-guide.md`, `_templates/{page,decision-matrix,trend,scenario}.md`
- 신규 — Part 0~7 `00-overview.md` 8개 (status: draft)
- 신규 — 전체 페이지 스텁 63개 (status: stub). 매니페스트 기반 생성
- 신규 — `assets/html/reference-architecture.html` (3-Layer 레퍼런스 아키텍처, 노드 클릭 → md 이동)
- 구조 — 설계안 대비 변경: `07-case-studies` → `07-appendix-scenarios`로 개명(부록 성격 명시), `03/11-decision-layer-system-one` → `03/11-decision-layer`(OpenAI Decisions API 포함), `06/system-one-models-jev` → `06/decision-models`, `06/agent-memory.md` 추가
