# AI Agent 설계 Wiki

> **프로젝트에서 AI Agent 시스템을 설계할 때, 각 단계에서 무엇을 근거로 어떤 기술을 선택하고 어떻게 진행할지 판단하기 위한 가이드.**

**처음이라면 이 한 페이지부터 → [에이전트 시스템을 설계할 때 무엇을 고려해야 하는가](00-overview/00-design-considerations.md)**

[전체 아키텍처 보기 (HTML)](assets/html/reference-architecture.html) · [기술 선택 결정 트리](00-overview/02-decision-guide.md) · [트렌드 레이더 2026](00-overview/03-trend-radar-2026.md) · [부록: 시나리오 예시](07-appendix-scenarios/00-overview.md)

---

## 이 위키가 다루는 것

AI Agent 시스템을 세 개의 레이어로 나누고, 각 레이어의 단계마다 **무엇을 · 언제 · 어떻게 · 왜** 쓰는지와 **그 단계에서 쓸 수 있는 기술들의 비교**, **한계**를 기술한다.

| 레이어 | 질문 | Part |
|---|---|---|
| **Knowledge** | 무엇을 근거로 판단하는가 | [Part 2. AI 기반 데이터 파이프라인 및 근거 데이터 설계](02-data-pipeline/00-overview.md) |
| **Reasoning / Action** | 어떻게 추론하고 행동하는가 | [Part 3. Agent 워크플로우 설계 및 LLM/RAG 파이프라인 아키텍처](03-agent-workflow-rag/00-overview.md) |
| **Interaction** | 결과를 사람과 어떻게 주고받는가 | [Part 4. Agent 기반 동적 UI 연동 설계](04-dynamic-ui/00-overview.md) |

세 레이어에 공통인 기반 지식은 [Part 1. Foundations](01-foundations/00-overview.md), 레이어를 가로지르는 보안·비용·관측성·거버넌스는 [Part 5. Cross-cutting](05-cross-cutting/00-overview.md)에 둔다.

### 운영을 전제로 한 설계

모든 단계 페이지에는 **"운영 설계 — 확장성 · 안정성 · 유지보수"** 섹션이 있다. 10배 규모에서 무엇이 먼저 깨지는지, 장애 모드와 폴백, 버전·호환, **변화(지식·모델·규칙·사용자 환경·프로토콜)를 어느 추상화 경계에서 흡수하는지**, **무엇을 기록·측정·알림해야 그 장애가 보이는지(관측성)**, 위험 등급과 가드레일·승인·감사 접점을 단계마다 답한다. 용어와 판단 기준은 [Part 1/07](01-foundations/07-operations-vocabulary.md), 한 장 요약은 [Part 5/06](05-cross-cutting/06-operational-design-matrix.md).

### 트렌드를 다루는 방식

최근 1년(2025-10 ~ 2026-10)에 각광받은 기술을 [Part 6. Trends](06-trends/00-overview.md)에서 심화하고, **그 단계의 선택지나 기본값을 실제로 바꾸는 트렌드만** 해당 단계 페이지의 "최신 트렌드 적용" 섹션에 적는다. 억지로 연결하지 않는다. 다음 세 주제는 특별히 깊게 다루는 관심 주제이며, 각각 의미가 있는 단계에서만 등장한다.

- **온톨로지 / 지식그래프** — 근거 데이터 구조화, 환각 억제, 설명가능성의 기반
- **뉴로심볼릭 AI** — LLM과 규칙·제약·추론기의 결합으로 검증 가능한 판단
- **결정 모델 (System One)** — TypeSafe AI **Jev**(2026-09-15 얼리액세스), OpenAI **Decisions API**(2026-09~10 프리뷰→베타). 생성 대신 타입드 판정만 반환하는 모델을 라우팅·게이팅·분류에 배치

### 부록: 시나리오 기반 End-to-End 예시

[Part 7. Appendix](07-appendix-scenarios/00-overview.md)는 구체적 상황을 주고 데이터 파이프라인 → 에이전트 구성 → Generative UI까지 **어떤 선택을 왜 했는지** 단계별로 보여준다. 위키 본문이 "선택지와 기준"이라면 부록은 "적용 예시"다.

---

## 목차

### [Part 0. Overview](00-overview/00-overview.md)
- [00 에이전트 시스템을 설계할 때 무엇을 고려해야 하는가 — 한 페이지 종합](00-overview/00-design-considerations.md)
- [01 레퍼런스 아키텍처 (3-Layer)](00-overview/01-reference-architecture.md) `HTML`
- [02 종합 기술 선택 결정 트리](00-overview/02-decision-guide.md) `HTML`
- [03 트렌드 레이더 2026](00-overview/03-trend-radar-2026.md) `HTML`
- [04 용어집](00-overview/04-glossary.md)

### [Part 1. Foundations — 공통 기반](01-foundations/00-overview.md)
- [01 LLM 기초: 컨텍스트, 구조화 출력, 함수 호출](01-foundations/01-llm-fundamentals.md)
- [02 System 1 vs System 2: 결정 모델과 생성 모델의 역할 분담](01-foundations/02-system1-vs-system2.md)
- [03 지식 표현: 택소노미 → 온톨로지 → 지식그래프](01-foundations/03-knowledge-representation.md)
- [04 뉴로심볼릭 AI](01-foundations/04-neuro-symbolic-ai.md)
- [05 에이전트 프로토콜: MCP · A2A · AG-UI · A2UI · MCP Apps](01-foundations/05-agentic-protocols.md) `HTML`
- [06 평가와 관측성 기초](01-foundations/06-evaluation-observability.md)
- [07 운영 용어와 판단 기준: 거버넌스·가드레일·안정성·확장성·유지보수·변화 대응](01-foundations/07-operations-vocabulary.md)

### [Part 2. 데이터 파이프라인 및 근거 데이터 설계](02-data-pipeline/00-overview.md) `HTML`
- [01 소스 수집과 문서 파싱](02-data-pipeline/01-source-ingestion.md)
- [02 청킹 전략](02-data-pipeline/02-chunking-strategies.md)
- [03 메타데이터와 출처 설계](02-data-pipeline/03-metadata-provenance.md)
- [04 임베딩과 인덱싱](02-data-pipeline/04-embedding-indexing.md)
- [05 지식그래프 구축 파이프라인](02-data-pipeline/05-knowledge-graph-construction.md) `HTML`
- [06 온톨로지 기반 데이터 모델링](02-data-pipeline/06-ontology-driven-modeling.md)
- [07 신선도와 동기화](02-data-pipeline/07-freshness-and-sync.md)
- [08 근거 데이터 설계 ★](02-data-pipeline/08-grounding-data-design.md)
- [09 파이프라인 내부의 AI](02-data-pipeline/09-ai-inside-the-pipeline.md)
- [10 접근 제어와 거버넌스](02-data-pipeline/10-access-control-governance.md)
- [99 결정 매트릭스](02-data-pipeline/99-decision-matrix.md)

### [Part 3. Agent 워크플로우 설계 및 LLM/RAG 파이프라인 아키텍처](03-agent-workflow-rag/00-overview.md) `HTML`
- [01 RAG 아키텍처 패턴](03-agent-workflow-rag/01-rag-architecture-patterns.md) `HTML`
- [02 질의 이해와 라우팅](03-agent-workflow-rag/02-query-understanding.md)
- [03 검색과 리랭킹](03-agent-workflow-rag/03-retrieval-reranking.md)
- [04 생성과 근거 강제](03-agent-workflow-rag/04-generation-grounding.md)
- [05 워크플로우 vs 에이전트](03-agent-workflow-rag/05-workflow-vs-agent.md)
- [06 에이전트 설계 패턴](03-agent-workflow-rag/06-agent-design-patterns.md) `HTML`
- [07 멀티 에이전트 오케스트레이션](03-agent-workflow-rag/07-multi-agent-orchestration.md) `HTML`
- [08 상태·메모리·컨텍스트 엔지니어링](03-agent-workflow-rag/08-state-memory-context.md)
- [09 툴과 MCP 통합](03-agent-workflow-rag/09-tools-and-mcp.md)
- [10 뉴로심볼릭 추론 적용](03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)
- [11 결정 레이어: System One 모델과 Decisions API ★](03-agent-workflow-rag/11-decision-layer.md)
- [12 가드레일과 Human-in-the-Loop](03-agent-workflow-rag/12-guardrails-hitl.md)
- [13 RAG·에이전트 평가](03-agent-workflow-rag/13-evaluation-rag-agents.md)
- [14 프레임워크 비교](03-agent-workflow-rag/14-frameworks-landscape.md)
- [99 결정 매트릭스](03-agent-workflow-rag/99-decision-matrix.md)

### [Part 4. Agent 기반 동적 UI 연동 설계](04-dynamic-ui/00-overview.md) `HTML`
- [01 Generative UI 스펙트럼](04-dynamic-ui/01-generative-ui-spectrum.md)
- [02 AG-UI · A2UI · MCP Apps 상세 비교](04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)
- [03 컴포넌트 레지스트리와 스키마](04-dynamic-ui/03-component-registry-schema.md)
- [04 스트리밍과 상태 동기화](04-dynamic-ui/04-streaming-state-sync.md) `HTML`
- [05 의도 → UI 매핑](04-dynamic-ui/05-intent-to-ui-mapping.md)
- [06 Human-in-the-Loop UI 패턴](04-dynamic-ui/06-hitl-ui-patterns.md)
- [07 프론트엔드 통합](04-dynamic-ui/07-frontend-integration.md)
- [08 보안과 샌드박싱](04-dynamic-ui/08-security-sandboxing.md)
- [09 UX와 접근성](04-dynamic-ui/09-ux-accessibility.md)
- [10 Generative UI 평가](04-dynamic-ui/10-evaluation-genui.md)
- [99 결정 매트릭스](04-dynamic-ui/99-decision-matrix.md)

### [Part 5. Cross-cutting — 횡단 관심사](05-cross-cutting/00-overview.md)
- [01 보안과 프롬프트 인젝션](05-cross-cutting/01-security-prompt-injection.md)
- [02 비용·지연·모델 라우팅](05-cross-cutting/02-cost-latency-model-routing.md)
- [03 관측성과 트레이싱](05-cross-cutting/03-observability-tracing.md)
- [04 거버넌스와 설명가능성](05-cross-cutting/04-governance-explainability.md)
- [05 LLMOps / AgentOps](05-cross-cutting/05-llmops-agentops.md)
- [06 운영 설계 종합 매트릭스: 단계 × 확장성·안정성·유지보수·변화 대응·거버넌스](05-cross-cutting/06-operational-design-matrix.md)
- [07 폐쇄망·망분리 환경에서의 설계: 배포 환경이 바꾸는 단계별 선택](05-cross-cutting/07-air-gapped-design.md)

### [Part 6. Trends — 최근 1년 심화](06-trends/00-overview.md)
- [온톨로지 르네상스 ★](06-trends/ontology-renaissance.md)
- [뉴로심볼릭 ★](06-trends/neuro-symbolic.md)
- [결정 모델: Jev와 OpenAI Decisions API ★](06-trends/decision-models.md)
- [GraphRAG의 진화](06-trends/graphrag-evolution.md)
- [컨텍스트 엔지니어링](06-trends/context-engineering.md)
- [에이전트 메모리 생태계](06-trends/agent-memory.md)
- [에이전트 프로토콜 수렴](06-trends/agentic-protocols-convergence.md)
- [Generative UI](06-trends/generative-ui.md)

### [Part 7. Appendix — 시나리오 기반 설계 예시](07-appendix-scenarios/00-overview.md)
- [시나리오 A: 사내 문서 QA 어시스턴트](07-appendix-scenarios/01-enterprise-document-qa.md) `HTML`
- [시나리오 B: 커머스 상담 + Generative UI](07-appendix-scenarios/02-commerce-assistant-genui.md) `HTML`
- [시나리오 C: 운영 인시던트 대응 에이전트](07-appendix-scenarios/03-ops-incident-agent.md) `HTML`

---

## 읽는 순서

| 독자 | 순서 |
|---|---|
| 처음 접하는 아키텍트 | [0/00 한 페이지 종합](00-overview/00-design-considerations.md) → Part 0 → Part 1 → 관심 Part의 `00-overview` → `99-decision-matrix` → Part 7 시나리오 |
| 특정 단계 담당자 | 해당 단계 페이지 → 같은 `stage` 태그의 다른 Part 페이지 → Part 5 |
| 기술 트렌드 파악 | Part 0/03 트렌드 레이더 → Part 6 → 각 단계의 "최신 트렌드 적용" 섹션 |
| 바로 설계 시작 | Part 0/02 결정 트리 → Part 7에서 가장 가까운 시나리오 → 해당 단계 페이지 |

## 범례

- `★` 이 위키에서 특히 깊게 다루는 주제 (온톨로지, 뉴로심볼릭, 결정 모델, 근거 데이터)
- `HTML` 해당 페이지에 인터랙티브 시각화가 있음 (`assets/html/`)
- 검증 수준 `[검증]` `[벤더]` `[2차]` `[실측]` — [작성 가이드](_meta/writing-guide.md) 5절
- 레이어 색상: Knowledge 초록 · Reasoning 파랑 · Interaction 주황 · Cross-cutting 회색

## 기여 규칙

- 모든 페이지는 [`_templates/`](_templates/)의 템플릿을 따른다
- 작성 규칙은 [`_meta/writing-guide.md`](_meta/writing-guide.md)
- 변경 이력은 [`_meta/CHANGELOG.md`](_meta/CHANGELOG.md)
- 구조 설계 원안은 [`WIKI_PLAN.md`](WIKI_PLAN.md)
