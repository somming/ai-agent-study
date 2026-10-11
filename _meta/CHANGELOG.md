# CHANGELOG

형식: `날짜 — 변경 유형 — 대상 — 요지`. 변경 유형: 신규 / 상태전이 / 수정 / 삭제 / 구조

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
