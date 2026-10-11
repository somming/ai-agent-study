---
title: "용어집"
part: 00-overview
stage: all
tags: [glossary]
maturity: 채택
status: review
updated: 2026-10-11
---

# 용어집

> **범위**: 위키 전반에서 쓰는 용어의 한/영 병기 정의. 정의는 **이 위키에서의 쓰임**을 기준으로 하며, 학계·벤더의 정의와 다를 수 있는 경우 그 사실을 적는다. 상세는 각 링크.

표기: **용어** (English) — 정의. → 관련 페이지

## 구조·레이어

- **3-Layer 모델** — Knowledge(근거) / Reasoning·Action(추론·행동) / Interaction(상호작용) + Cross-cutting(횡단). 이 위키의 뼈대. → [01](01-reference-architecture.md)
- **레이어 간 계약** (inter-layer contract) — 근거 단위 스키마(①), 렌더 가능한 응답 스키마(②), 사용자 이벤트 스키마(③). 레이어보다 먼저 정한다. → [01](01-reference-architecture.md)
- **단계** (stage) — 프론트매터 `stage` 값. ingestion / indexing / knowledge / retrieval / reasoning / action / interaction / all. 시각화와 페이지를 잇는 키
- **횡단 관심사** (cross-cutting concern) — 보안, 비용·지연, 관측성, 거버넌스, 운영처럼 세 레이어에 일관 적용해야 하는 정책. → [Part 5](../05-cross-cutting/00-overview.md)

## 판정과 생성

- **System 1 / System 2** — Kahneman의 빠른 직관 / 느린 숙고 프레임을 빌린 **은유**. 이 위키에서 System 2는 토큰 생성형 LLM, System 1은 결정 모델·분류기·규칙 같은 판정 전용 수단. 인지과학적 동일성을 주장하지 않는다. → [1/02](../01-foundations/02-system1-vs-system2.md)
- **결정 모델** (decision model, System One model) — 텍스트를 생성하지 않고 개발자가 정의한 타입드 질문에 선택·점수·예/아니오와 (보정) 확률만 반환하는 모델 또는 API. 2026-09 이후 상용화. → [1/02](../01-foundations/02-system1-vs-system2.md), [6/decision-models](../06-trends/decision-models.md)
- **Jev** — TypeSafe AI의 결정 전용 모델. 2026-09-15 얼리액세스. 비자기회귀 단일 패스 주장. 영어 중심. → [6/decision-models](../06-trends/decision-models.md)
- **Decisions API** — OpenAI의 결정 엔드포인트(`/v1/decisions`). 2026-09-29 발표. 범용 모델(gpt-6-luna)의 출력을 제약한 것으로 **결정 전용 모델은 아님**. → [6/decision-models](../06-trends/decision-models.md)
- **보정 확률** (calibrated probability) — "80%"라고 할 때 실제로 10번 중 8번 맞는 확률. 결정 모델의 핵심 주장이며 신뢰도 다이어그램으로 검증한다. → [1/02](../01-foundations/02-system1-vs-system2.md)
- **게이팅** (gating) — 다음 단계로 넘어갈지를 판정하는 지점. 라우팅 게이트, 승인 게이트, 품질 게이트. → [3/11](../03-agent-workflow-rag/11-decision-layer.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)
- **라우팅** (routing) — 요청·하위 과업을 경로·모델·툴·에이전트 중 하나로 보내는 판정. → [3/02](../03-agent-workflow-rag/02-query-understanding.md), [5/02](../05-cross-cutting/02-cost-latency-model-routing.md)
- **구조화 출력** (structured output) — 프로그램이 소비할 수 있는 스키마 준수 출력. JSON 모드 → 함수 호출 → 제약 디코딩 → 결정 모델 순으로 강제력 증가. 스키마 준수 ≠ 내용 정확. → [1/01](../01-foundations/01-llm-fundamentals.md)
- **함수 호출 / 툴 호출** (function calling / tool use) — 모델이 호출하고 싶은 함수와 인자를 구조화해 돌려주는 것. 실행은 호출자 책임. → [1/01](../01-foundations/01-llm-fundamentals.md), [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md)
- **제약 디코딩** (constrained decoding) — 디코딩 단계에서 문법에 어긋나는 토큰을 차단해 100% 스키마 준수. 주로 자체 서빙. → [1/01](../01-foundations/01-llm-fundamentals.md)

## 지식 표현

- **택소노미** (taxonomy) — 개념의 계층 분류(is-a, broader/narrower). 표준 SKOS. "찾게 한다". → [1/03](../01-foundations/03-knowledge-representation.md)
- **온톨로지** (ontology) — 클래스·속성·관계·제약·공리의 형식 정의. 표준 OWL/RDFS, 제약 SHACL. "의미를 갖게 한다". 이 위키의 관심 주제 ★. → [1/03](../01-foundations/03-knowledge-representation.md), [2/06](../02-data-pipeline/06-ontology-driven-modeling.md)
- **지식그래프** (knowledge graph, KG) — 온톨로지에 따라 채워진 인스턴스·관계. RDF 트리플 또는 LPG. "연결되게 한다". → [2/05](../02-data-pipeline/05-knowledge-graph-construction.md)
- **RDF / OWL / SHACL / SPARQL** — W3C 표준 스택. 트리플 데이터 모델 / 온톨로지 언어 / 그래프 형상 제약 / 질의 언어. → [1/03](../01-foundations/03-knowledge-representation.md)
- **LPG** (Labeled Property Graph) — 노드·관계 모두 속성을 가지는 그래프 모델. Neo4j 류. 개발 편의·성능 우위, 추론·상호운용은 표준 스택 우위. → [1/03](../01-foundations/03-knowledge-representation.md)
- **엔티티 해소** (entity resolution) — 다른 이름·ID로 등장하는 같은 실체를 하나로 묶는 것. → [2/05](../02-data-pipeline/05-knowledge-graph-construction.md)
- **시맨틱 레이어** (semantic layer) — 테이블·컬럼 대신 비즈니스 개념을 노출하는 중간 정의 계층. 온톨로지의 실용 구현 형태. → [1/03](../01-foundations/03-knowledge-representation.md)
- **접지** (grounding) — 모델의 출력을 검증 가능한 외부 근거·개념에 묶는 것. RAG의 접지는 근거 문서, 온톨로지의 접지는 허용 개념. → [2/08](../02-data-pipeline/08-grounding-data-design.md)

## 뉴로심볼릭

- **뉴로심볼릭 AI** (neuro-symbolic AI) — 신경망(LLM 포함)과 기호 체계(규칙·온톨로지·논리·솔버·프로그램)의 결합 패턴 집합. 특정 기술이 아님. 이 위키의 관심 주제 ★. → [1/04](../01-foundations/04-neuro-symbolic-ai.md)
- **Kautz 분류** — AAAI 2020에서 제시된 뉴로심볼릭 결합 유형 분류. 소스마다 5~6종. `A|B`(별개 모듈 통신), `A[B]`(내장). → [1/04](../01-foundations/04-neuro-symbolic-ai.md)
- **검증기 패턴** (Generate → Verify) — LLM 출력을 스키마·SHACL·규칙·솔버가 검사하고 불합격 시 재생성·폴백. 실무에서 가장 흔한 뉴로심볼릭 패턴. → [1/04](../01-foundations/04-neuro-symbolic-ai.md), [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)
- **변환기 패턴** (Translate → Execute) — LLM이 자연어를 SQL·Cypher·SPARQL·코드로 변환하고 결정론적 엔진이 실행. 실행 결과가 근거. → [1/04](../01-foundations/04-neuro-symbolic-ai.md)
- **게이트 패턴** (Rule ∧ Model) — 행동 직전 규칙이 최종 허가. 모델이 허용해도 규칙이 거부하면 차단. → [1/04](../01-foundations/04-neuro-symbolic-ai.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)
- **규칙 엔진** (rules engine) — if-then 비즈니스·정책 규칙의 실행기. 일반 서술로만(제품 비교는 하지 않음). → [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)

## 데이터 파이프라인·RAG

- **RAG** (Retrieval-Augmented Generation) — 외부 근거를 검색해 컨텍스트에 넣고 생성. Naive → Advanced → Modular → Agentic으로 진화. → [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md)
- **Agentic RAG** — 검색이 에이전트 루프 "앞"이 아니라 "안"에 있어 모델이 재검색·재작성을 결정하는 구성. → [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md)
- **GraphRAG / LightRAG / HippoRAG 2** — 그래프 기반 RAG 계열. 각각 커뮤니티 요약 / 증분·이중 검색 / Personalized PageRank. 단일 승자 없음. → [6/graphrag-evolution](../06-trends/graphrag-evolution.md)
- **청킹** (chunking) — 문서를 검색 단위로 자르는 것. fixed / recursive / semantic / late / contextual / parent-child. → [2/02](../02-data-pipeline/02-chunking-strategies.md)
- **Late chunking** — 문서 전체를 임베딩 모델에 통과시킨 뒤 토큰 임베딩을 청크 단위로 풀링. 효율↑ 관련성↓(arXiv 2504.19754). → [2/02](../02-data-pipeline/02-chunking-strategies.md)
- **Contextual retrieval** — 각 청크 앞에 문서 맥락 요약을 접두해 임베딩. 일관성↑ 비용↑. → [2/02](../02-data-pipeline/02-chunking-strategies.md)
- **하이브리드 검색** (hybrid search) — 키워드(BM25 등 sparse) + 벡터(dense)를 RRF 등으로 융합. 2026년 기본값. → [2/04](../02-data-pipeline/04-embedding-indexing.md), [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md)
- **RRF** (Reciprocal Rank Fusion) — 여러 검색 결과의 순위를 역수 합으로 융합하는 방법
- **리랭킹** (reranking) — 1차 검색 결과를 cross-encoder·LLM 등으로 재정렬. → [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md)
- **근거 단위** (evidence unit) — 답변에 인용할 수 있는 최소 단위(청크·문장·트리플·레코드). 계약 ①. 이 위키의 핵심 설계 대상 ★. → [2/08](../02-data-pipeline/08-grounding-data-design.md)
- **citation** — 답변의 각 주장을 근거 단위에 연결한 참조. 스키마는 Part 2가 정의하고 Part 4가 그대로 렌더. → [2/08](../02-data-pipeline/08-grounding-data-design.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)
- **출처 추적** (provenance) — 데이터가 어디서 어떻게 왔는지의 기록. W3C PROV. → [2/03](../02-data-pipeline/03-metadata-provenance.md)
- **ACL-aware retrieval** — 사용자 권한을 검색 시점에 적용해 볼 수 없는 문서를 결과에서 제외. → [2/10](../02-data-pipeline/10-access-control-governance.md)
- **faithfulness / context precision / context recall / answer relevancy** — RAGAS 류 표준 RAG 지표. 근거 부합 / 검색 정밀도 / 검색 재현율 / 답변 관련성. → [1/06](../01-foundations/06-evaluation-observability.md)

## 에이전트·워크플로우

- **워크플로우 vs 에이전트** — 코드가 경로를 정하는 결정론적 구성 vs 모델이 경로를 정하는 자율 구성. 스펙트럼이며 "가장 단순한 형태에서 시작". → [3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md)
- **ReAct / Plan-and-Execute / Reflection / Orchestrator-Worker** — 에이전트 설계 패턴. → [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md)
- **Supervisor / Hierarchical / Handoff** — 멀티 에이전트 토폴로지. → [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md)
- **컨텍스트 엔지니어링** (context engineering) — 컨텍스트 윈도우에 무엇을 넣고 **빼고** 어떻게 구조화할지의 공학. 프롬프트 엔지니어링의 확장. → [3/08](../03-agent-workflow-rag/08-state-memory-context.md), [6/context-engineering](../06-trends/context-engineering.md)
- **compaction** — 긴 히스토리·상태를 요약·압축해 컨텍스트 예산 안에 유지하는 것
- **체크포인팅** (checkpointing) — 에이전트 상태를 단계마다 저장해 재개·재시도·감사를 가능하게 하는 것. → [3/08](../03-agent-workflow-rag/08-state-memory-context.md)
- **에이전트 메모리** (agent memory) — 세션 간 지속되는 지식. 대화 컨텍스트용과 운영 지식용으로 분화. no-memory 베이스라인 대비로 평가. → [6/agent-memory](../06-trends/agent-memory.md)
- **가드레일** (guardrail) — 입력·출력·행동에 대한 안전·정책 검사. → [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)
- **HITL** (Human-in-the-Loop) — 사람이 승인·수정·개입하는 지점과 그 UI. → [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)
- **트래젝토리 평가** (trajectory evaluation) — 최종 답이 아니라 에이전트의 행동 경로(툴 순서, 재시도, 비용)를 평가. → [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md)
- **LLM-as-judge** — LLM으로 다른 LLM 출력의 품질을 판정. 편향(길이·위치·자기 선호) 있어 사람 라벨로 보정 필요. → [1/06](../01-foundations/06-evaluation-observability.md)
- **골든셋** (golden set) — 입력·기대 출력·기대 근거 쌍의 평가 데이터. 첫 프롬프트 전에 만든다. → [1/06](../01-foundations/06-evaluation-observability.md)

## 프로토콜

- **MCP** (Model Context Protocol) — 에이전트 ↔ 툴·데이터 소스 프로토콜. Anthropic 2024-11. 2026-07-28 개정으로 무상태 코어, Tasks·Apps 확장. → [1/05](../01-foundations/05-agentic-protocols.md)
- **A2A** (Agent2Agent) — 에이전트 ↔ 에이전트 프로토콜. Google 2025-04, Linux Foundation 2025-06 이관. → [1/05](../01-foundations/05-agentic-protocols.md)
- **AG-UI** (Agent-User Interaction) — 에이전트 백엔드 ↔ 프론트엔드 이벤트 프로토콜. CopilotKit, MIT, SSE. "이벤트를 어떻게 흘릴까". → [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)
- **A2UI** (Agent-to-UI) — 선언적 UI 트리 스펙. Google, Apache 2.0, v0.9(2026-04). "UI가 무엇인가". → [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)
- **MCP Apps** — MCP 서버가 HTML UI를 반환하고 호스트가 샌드박스 iframe에 렌더하는 MCP 확장. → [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)
- **Agent Card** — A2A에서 에이전트가 자기 능력·엔드포인트·인증을 공개하는 메타데이터

## Generative UI

- **Generative UI** (GenUI) — 에이전트가 텍스트 대신 UI를 결정·구성해 돌려주는 것. → [Part 4](../04-dynamic-ui/00-overview.md)
- **Controlled / Declarative / Open-ended** — GenUI 3패턴. 사전 정의 컴포넌트 선택 / UI 트리를 JSON으로 기술 / HTML·코드 생성. 기본값은 Controlled. → [4/01](../04-dynamic-ui/01-generative-ui-spectrum.md)
- **컴포넌트 레지스트리** (component registry) — 에이전트가 선택할 수 있는 컴포넌트 카탈로그와 각 props 스키마. 계약 ②의 기반. → [4/03](../04-dynamic-ui/03-component-registry-schema.md)
- **ui_intent** — 렌더 가능한 응답(계약 ②) 안에서 "어떤 컴포넌트에 어떤 데이터"를 지정하는 선택적 필드. 이 위키의 명명
- **의도→UI 매핑** (intent-to-UI mapping) — 어떤 UI를 띄울지의 판정. 결정 모델·온톨로지·규칙·LLM 툴 호출. → [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md)
- **optimistic UI** — 서버 확인 전에 결과를 먼저 보여주고 실패 시 롤백하는 UI 기법. → [4/04](../04-dynamic-ui/04-streaming-state-sync.md)

## 운영·거버넌스 (정의는 [1/07](../01-foundations/07-operations-vocabulary.md))

- **거버넌스** (governance) — 누가 무엇을 결정·승인·책임지는가의 체계. 정책 + RACI + 변경 관리 + 감사 + 예외 절차. 규제 대응과 동의어가 아님
- **정책 / 규칙 / 가드레일 / 게이트** — 같은 것의 네 층위. 사람이 읽는 선언 → 기계가 평가하는 조건 → 런타임 경로에 심은 예방적 통제(입력·출력·행동) → 통과/차단/승인/저하로 분기하는 판정 지점
- **승인 매트릭스** (approval matrix) — 행동 유형 × 영향 범위 × 요청자 역할 → 승인자·승인 수. 만료는 거부가 기본
- **승인 피로** (approval fatigue) — 승인 요청이 많아 사람이 전부 승인하게 되는 현상. 확신도 임계값·규칙 승격으로 양을 줄인다
- **감사 추적** (audit trail) — 사후에 "왜 그 결정이 났나"를 재구성하는 불변·장기 보존 기록. 관측 로그와 저장소·보존 정책을 분리
- **예방 · 탐지 · 교정 통제** — 가드레일·게이트 / 지표·알림·감사 / 폴백·롤백·킬 스위치. 세 층 모두 필요
- **위험 등급** (1~5) — 가역성 × 영향 범위로 정하는 통제 수준. 5등급은 에이전트 자동 실행 금지
- **장애 모드** (failure mode) / **폴백** (fallback) / **우아한 저하** (graceful degradation) — 실패하는 방식의 목록 / 대체 수단 / 기능을 줄여 계속 동작
- **서킷 브레이커** (circuit breaker) / **백프레셔** (backpressure) / **킬 스위치** (kill switch) — 연쇄 장애 차단 / 과부하 시 입력 억제 / 배포 없이 기능 끄기
- **SLI / SLO / 오류 예산** — 측정 지표 / 목표치 / 허용 실패량. 소진 시 변경 동결
- **런북** (runbook) / **운영 준비도 리뷰** (ORR) — 알림별 대응 절차 / 배포 전 장애 모드·폴백·알림·롤백·소유자 점검
- **카나리 / 섀도 / 피처 플래그 / 롤백 단위** — 일부 트래픽 배포 / 결과 미노출 실행 / 배포 없는 on·off / 함께 되돌릴 묶음
- **호환 매트릭스** (compatibility matrix) — 모델·프롬프트·인덱스·툴·카탈로그 중 함께 배포되어야 하는 조합
- **변화 대응 설계** (design for change) — 안정 계약 뒤에 가변 구현을 두고, 프롬프트·규칙·카탈로그·임계값을 코드 밖 설정으로, 외부 의존을 포트·어댑터 뒤에 두어 변화를 국소적으로 흡수하는 설계
- **안정 계약 vs 가변 구현** — 레이어 간 계약은 버전을 붙여 느리게, 구현은 계약을 지키는 한 자유롭게 교체
- **포트와 어댑터** (ports and adapters) — 외부 의존(모델·DB·프로토콜)을 내부 인터페이스 뒤에 격리. `decide()`·`retrieve()`·`render()` 같은 추상화
- **스키마 진화** (schema evolution) — 추가는 자유, 삭제·의미 변경은 버전 상승. 소비자는 모르는 필드를 무시(tolerant reader)
- **시점 재현성** (point-in-time reproducibility) — 과거의 지식·모델·규칙 버전으로 결과를 재현할 수 있는 성질
- **RACI** — 실행(Responsible)·책임(Accountable)·자문(Consulted)·통보(Informed). A가 빈 행이 없어야 한다
- **데이터 거버넌스** / **모델 리스크 관리** — 분류 등급·소유자·보존·lineage / 모델 카드·검증 기록·변경 승인·재검증 주기

## 평가·관측·운영

- **검증 수준** — 이 위키의 출처 표기. `[검증]` 독립 벤치마크·논문·표준 / `[벤더]` 벤더 주장 / `[2차]` 블로그·해설 / `[실측]` 우리 측정. `[벤더]`·`[2차]`만으로 "채택" 불가. → [작성 가이드](../_meta/writing-guide.md)
- **레이더 위치** — 채택 / 시험 / 관찰 / 보류. 트렌드와 페이지의 `maturity`. → [03](03-trend-radar-2026.md)
- **OTel GenAI 시맨틱 컨벤션** — OpenTelemetry의 LLM·에이전트 트레이스 속성 표준. 2026-10 현재 Development 등급. → [1/06](../01-foundations/06-evaluation-observability.md), [5/03](../05-cross-cutting/03-observability-tracing.md)
- **`invoke_agent` 스팬** — OTel GenAI 컨벤션의 에이전트 호출 스팬. 아래에 모델·툴 호출 자식 스팬
- **프롬프트 인젝션** (prompt injection) — 외부 입력(문서·툴 결과·UI 이벤트)에 담긴 지시가 모델 행동을 바꾸는 공격. 직접/간접. → [5/01](../05-cross-cutting/01-security-prompt-injection.md)
- **LLMOps / AgentOps** — 모델·프롬프트·인덱스·툴·컴포넌트 카탈로그의 버전·평가·배포·롤백 운영. → [5/05](../05-cross-cutting/05-llmops-agentops.md)
- **프롬프트 캐싱** (prompt caching) — 반복되는 접두사(시스템 프롬프트·툴 정의)를 캐시해 비용·지연 절감. → [5/02](../05-cross-cutting/02-cost-latency-model-routing.md)
- **시맨틱 캐시** (semantic cache) — 의미적으로 같은 질의의 응답을 재사용. 근거 신선도·권한과 충돌 주의

## 관련 페이지

- [작성 가이드](../_meta/writing-guide.md) — 용어 표기 규칙(한국어 본문, 영문 첫 병기)
- [조사 노트](../_meta/research-notes-2026-10.md) — 트렌드 용어의 날짜·검증 수준 원천
