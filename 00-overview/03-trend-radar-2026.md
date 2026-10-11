---
title: "트렌드 레이더 2026 (최근 1년)"
part: 00-overview
stage: all
tags: [trend-radar, trends, maturity, ontology, neuro-symbolic, decision-model, protocols, generative-ui]
maturity: 채택
status: review
updated: 2026-10-11
---

# 트렌드 레이더 2026 (최근 1년)

> **범위**: 2025-10 ~ 2026-10 사이 각광받은 기술을 채택/시험/관찰/보류 4분면으로 배치
> **시각화**: [`assets/html/trend-radar.html`](../assets/html/trend-radar.html) — 점 호버 시 요지·검증 수준·first_seen, 클릭 시 심화 페이지. 항목 번호(T#)는 아래 표와 동일

## 요약 (3줄)

1. 최근 1년 트렌드 **29개 항목**(Part 6의 8개 + 단계 페이지가 트렌드로 다룬 세부 21개)을 4분면에 놓았다. **채택 2 · 시험 17 · 관찰 8 · 보류 2**. 채택이 적은 이유는 작성 가이드 5절 — `[벤더]`·`[2차]`만으로는 채택하지 않는다.
2. 관심 주제 ★ 세 축의 위치: **온톨로지 = 시험, 뉴로심볼릭 = 시험, 결정 모델 = 관찰**. 셋 다 "요구가 `[실측]`으로 확인된 프로젝트에서만" 다음 분면으로 간다.
3. 이번 1년의 구조적 변화는 세 가지 — **판정의 독립 레이어화**(System One), **온톨로지 회귀**("스키마 없는 그래프는 또 다른 비정형"), **프로토콜의 경쟁 → 조합**(경계마다 하나씩).

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | 각 트렌드의 분면·first_seen·검증 수준·적용 단계를 한 장에 모은 표와, 분면을 옮기는 조건. 품질 기준: 분면과 first_seen이 [Part 6](../06-trends/00-overview.md) 각 페이지의 프론트매터와 일치하고, 모든 수치·날짜가 [연구 노트](../_meta/research-notes-2026-10.md)에 있다 |
| **언제** (When) | [02 결정 트리](02-decision-guide.md)를 돌린 뒤 "시험해 볼 트렌드"를 고를 때, 분기별 기술 리뷰 때, Part 6 페이지의 채택 기준이 `[실측]`되어 분면을 옮길 때. 생략 가능: 기존 운영 시스템의 유지보수만 할 때 |
| **어떻게** (How) | ① 결정 트리의 Q6·Q10·Q11 답을 확인 → ② 해당 축의 현재 분면과 "이동 조건"을 읽는다 → ③ "시험"에 있는 항목 중 내 변수(V6·V9·V12 등)에 걸리는 것만 PoC 후보로 → ④ PoC 결과를 [Part 7](../07-appendix-scenarios/00-overview.md)에 `[실측]`으로 기록 → ⑤ Part 6 "채택으로 올리는 절차" 4단계를 밟아 분면을 옮기고 CHANGELOG에 적는다 |
| **왜** (Why) | 레이더 없이 Part 6를 읽으면 8개 트렌드가 모두 "써야 하는 것"으로 보인다. 분면이 있어야 "지금 쓸 것 / 조건 확인 후 시험할 것 / 지켜볼 것 / 쓰지 않을 것"이 갈리고, 벤더 서사(온톨로지 필수, 200배 빠름, 메모리 표준)가 설계 근거로 스며드는 것을 막는다 |

## 문제와 맥락

2025-10 ~ 2026-10 사이에 에이전트 설계의 어휘가 크게 바뀌었다. "결정 모델", "컨텍스트 엔지니어링", "온톨로지 르네상스", "Generative UI 3패턴", "프로토콜 수렴"은 모두 이 기간에 생기거나 재포장된 말이다. 문제는 **담론의 생산자 대부분이 벤더**라는 점이다 — 결정 모델 수치는 TypeSafe·OpenAI, 온톨로지 필수론은 Neo4j·Franz, 메모리 벤치마크 표준화는 Mem0, GenUI 3패턴은 CopilotKit이 만들었다. 독립 측정은 GraphRAG 재분석(RAGU)과 뉴로심볼릭 V&V 서베이(Renkhoff) 정도다.

레이더는 이 비대칭을 그대로 드러내는 장치다. 분면은 "얼마나 유망한가"가 아니라 **"우리가 채택해도 될 만큼 검증되었는가"** 를 뜻한다. 그래서 가장 많이 언급된 트렌드(결정 모델)가 가장 바깥 분면(관찰)에 있다.

## 핵심 개념

### 레이더 규칙

| 분면 | 뜻 | 들어오는 조건 | 이 위키에서의 취급 |
|---|---|---|---|
| **채택** | 기본값으로 쓴다 | `[검증]` 또는 `[실측]`이 있고, 적용 단계 페이지의 비교표에 "적합/부적합 상황"이 채워져 있다 | 결정 트리의 기본값에 반영 |
| **시험** | 조건이 확인된 프로젝트에서 PoC로 `[실측]`을 만든다 | 요구가 `[실측]`으로 확인됨(멀티홉 비율, 쓰기 툴 존재, 당사자 둘 이상 등) + 기술 자체의 위험이 낮음(표준 기반 또는 복수 소스 일치) | 99 매트릭스의 조건 A/B에만 등장 |
| **관찰** | 추적하되 PoC 전 | 근거가 `[벤더]`·`[2차]`뿐이거나 공개 직후(수개월), 또는 호환성 파괴 변경이 진행 중 | 어댑터·인터페이스만 미리 둔다 |
| **보류** | 현재 도입하지 않는다 | 역결과가 있거나(no-memory보다 나쁨), 유지보수 모드 등 상태가 종료 | 99 매트릭스 "피해야 할 조합" |

**이동 조건** ([작성 가이드](../_meta/writing-guide.md) 5절, [Part 6 개요](../06-trends/00-overview.md) "채택으로 올리는 절차"):

- 관찰 → 시험: 요구가 `[실측]`으로 확인되고 한국어 평가셋·골든셋 작업이 시작된 시점
- 시험 → 채택: 해당 Part 6 페이지의 "우리 프로젝트에서의 채택 기준"이 **전부** `[실측]`되고, ① 벤더 주장 vs 검증 표 갱신 ② 시나리오 페이지에 PoC 기록 ③ 적용 단계 비교표에 행 추가 ④ 레이더 갱신 + CHANGELOG의 4단계를 마친 뒤
- 어느 분면 → 보류: 역결과 `[검증]`/`[실측]`, 프로젝트 종료·유지보수 전환, 또는 표준 편입으로 독립 항목이 소멸
- 보류 → 관찰: 상태 변화(재활성화, 역결과의 반박, 원문 확인)

**기간**: 2025-10 ~ 2026-10. first_seen이 이보다 앞선 항목(late chunking 비교 연구 2025-04, A2A 이관 2025-06)은 **이 기간에 기본값 담론이 되었거나 성격이 바뀐 것**만 올렸고, 2024년 이전 기술(GraphRAG 원형, RDF/OWL, Text-to-SQL)은 Part 1 기반 지식으로 두었다.

### 레이어 색 (HTML)

Knowledge `#2E7D6B` · Reasoning `#3A5FA8` · Interaction `#B5651D` · Cross-cutting `#6B6B6B`. 레이어는 Part 6 페이지의 `stage`(knowledge / reasoning·retrieval / interaction / all)와 세부 항목의 소속 Part를 따른다. ★ 세 축(온톨로지 · 뉴로심볼릭 · 결정 모델과 그 하위 Jev · Decisions API)은 트렌드 색 `#8E44AD` 테두리.

## 아키텍처 / 파이프라인

레이더는 단독으로 쓰이지 않는다. [01 레퍼런스 아키텍처](01-reference-architecture.md)가 **어디에**(점선 노드), [02 결정 트리](02-decision-guide.md)가 **언제**(Q6·Q10·Q11), 이 페이지가 **지금 얼마나 검증되었는가**를 답한다. 한 항목의 분면이 바뀌면 결정 트리의 해당 질문 리프와 99 매트릭스의 기본값/조건 열이 함께 바뀐다.

## 레이더 표 — 2025-10 ~ 2026-10

Part 6의 8개 트렌드(굵게)와 단계 페이지의 "최신 트렌드 적용"에서 다룬 세부 항목. 분면·first_seen은 Part 6 페이지 프론트매터와 일치시켰고, 세부 항목은 소속 Part 페이지의 `maturity`·서술을 근거로 정했다(근거가 약한 것은 "관찰").

| # | 항목 | 분면 | first_seen | 한 줄 요지 | 검증 수준 | 적용 단계 | 심화 페이지 |
|---|---|---|---|---|---|---|---|
| T01 | **온톨로지 르네상스 ★** | 시험 | 2026-03 | "스키마 없는 그래프는 또 다른 비정형 데이터." 추출 스키마·검증기·접지·설명을 온톨로지로 묶는 흐름. 담론 생산자 대부분이 그래프 DB 벤더 | `[검증-단일저자]` 2604.00555 · `[벤더]` Franz/Gartner 전언 · `[2차]` | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) [2/06](../02-data-pipeline/06-ontology-driven-modeling.md) [2/08](../02-data-pipeline/08-grounding-data-design.md) [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) [5/04](../05-cross-cutting/04-governance-explainability.md) | [ontology-renaissance](../06-trends/ontology-renaissance.md) |
| T02 | **뉴로심볼릭 ★** | 시험 | 2026-02 | LLM 출력을 규칙·제약·온톨로지로 검증하는 2단계 구성이 "기업 적용 패턴"으로 재명명. 검증기 패턴 자체는 운영 표준에 가깝지만 결합 전체의 V&V는 미성숙 | `[검증]` Renkhoff 2024, 2602.00731 · `[벤더]` NSAI 식별 | [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) [3/04](../03-agent-workflow-rag/04-generation-grounding.md) [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) [4/03](../04-dynamic-ui/03-component-registry-schema.md) | [neuro-symbolic](../06-trends/neuro-symbolic.md) |
| T03 | **결정 모델 (범주) ★** | 관찰 | 2026-09 | 생성 없이 타입드 판정 + 확신도만 반환. 라우팅·게이팅·분류의 새 기본값 **후보**. 공개 한 달, 독립 벤치마크 없음, 영어 중심 | `[벤더]` `[2차]` 소스 충돌 | [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) [3/02](../03-agent-workflow-rag/02-query-understanding.md) [3/11](../03-agent-workflow-rag/11-decision-layer.md) [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) [5/02](../05-cross-cutting/02-cost-latency-model-routing.md) | [decision-models](../06-trends/decision-models.md) |
| T04 | Jev (TypeSafe AI) ★ | 관찰 | 2026-09 | 결정 전용 비자기회귀 모델, 단일 패스 병렬 샘플링, RLCD 보정 주장. 70~500ms · "200배/400배"는 비교 조건 미공개 | `[벤더]` `[2차]` LangChain 테스트 1건 외 없음 | [3/11](../03-agent-workflow-rag/11-decision-layer.md) [1/02](../01-foundations/02-system1-vs-system2.md) | [decision-models](../06-trends/decision-models.md) |
| T05 | OpenAI Decisions API ★ | 관찰 | 2026-09 | 범용 모델(gpt-6-luna) 출력을 제약한 `POST /v1/decisions`. 별도 결정 전용 모델 아님, 이미지 입력 가능, 지연 미보고 | `[2차]` 발표일(09-29)·베타(10-06경)·가격 소스 충돌 | [3/11](../03-agent-workflow-rag/11-decision-layer.md) [1/02](../01-foundations/02-system1-vs-system2.md) | [decision-models](../06-trends/decision-models.md) |
| T06 | **GraphRAG의 진화** | 시험 | 2025-10 | GraphRAG 비용 비판 → LightRAG(증분) · HippoRAG 2(PPR)로 분화. 재분석 결과 "단일 승자 없음, 가치는 질의 복잡도에 비례" | `[검증]` RAGU · `[2차]` 실무자 합의 | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) | [graphrag-evolution](../06-trends/graphrag-evolution.md) |
| T07 | LightRAG 증분 업데이트 | 시험 | 2025-10 | 엔티티-관계 그래프 + 저수준/고수준 이중 검색 + 증분 갱신. "61만 토큰 vs 100 미만"은 2차 요약, 설계 근거로 쓰지 않음 | `[2차]` | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) [2/07](../02-data-pipeline/07-freshness-and-sync.md) | [graphrag-evolution](../06-trends/graphrag-evolution.md) |
| T08 | HippoRAG 2 · RAGU 재분석 | 시험 | 2026-07 | 장황 프롬프트의 우위는 포맷 효과. 포맷 통제 후 MuSiQue에서만 뚜렷(54.4 vs 40.1). 그래프 RAG 평가 설계의 기준 문헌 | `[검증]` arXiv 2607.11683 | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) | [graphrag-evolution](../06-trends/graphrag-evolution.md) |
| T09 | **컨텍스트 엔지니어링** | 시험 | 2025-10 | "무엇을 넣고 뺄지"의 공학. 선택·압축·구조화·예산. ADK 등 프레임워크가 1급 기능으로 흡수. 방법론 효과는 `[2차]` | `[검증]` Lost in the Middle·RULER(기반) · `[벤더]` ADK · `[2차]` | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) [3/04](../03-agent-workflow-rag/04-generation-grounding.md) [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) [5/02](../05-cross-cutting/02-cost-latency-model-routing.md) | [context-engineering](../06-trends/context-engineering.md) |
| T10 | **에이전트 메모리 생태계** | 관찰 | 2025-10 | 대화 컨텍스트용 / 운영 지식용으로 분화. 벤치마크 표준화 보고(벤더 발)와 역결과(MemTrapBench)가 공존 | `[벤더]` Mem0 · `[2차]` | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) | [agent-memory](../06-trends/agent-memory.md) |
| T11 | LoCoMo · LongMemEval · BEAM 표준화 | 관찰 | 2026 (월 미기록) | 메모리 평가의 표준이라는 보고. 메모리 벤더 발이고 원 논문이 노트에 미기록 | `[벤더]` | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) [1/06](../01-foundations/06-evaluation-observability.md) | [agent-memory](../06-trends/agent-memory.md) |
| T12 | MemTrapBench 교훈 — no-memory 베이스라인 | 시험 | 2026 (월 미기록) | 메모리 프레임워크 5종 모두 no-memory보다 나쁨. 평가 기준점을 "메모리 vs 없음"으로. 원칙은 즉시 적용, 결과 자체는 원문 확인 전 | `[2차]` | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) | [agent-memory](../06-trends/agent-memory.md) |
| T13 | 메모리 프레임워크 기본 도입 | 보류 | 2026 (월 미기록) | 멀티세션 요구 확인·no-memory 대비 `[실측]` 없이 메모리 프레임워크를 넣는 것. 99 매트릭스 "피해야 할 조합" | `[2차]` 역결과 | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) | [agent-memory](../06-trends/agent-memory.md) |
| T14 | **에이전트 프로토콜 수렴** | 시험 | 2025-06 | MCP(툴) · A2A(에이전트 간) · AG-UI(이벤트) · A2UI(UI 내용) · MCP Apps(툴 UI)가 경쟁에서 경계별 조합으로. "당사자 둘 이상인 경계에만" | `[2차]` | [1/05](../01-foundations/05-agentic-protocols.md) [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) | [agentic-protocols-convergence](../06-trends/agentic-protocols-convergence.md) |
| T15 | MCP 2026-07-28 개정 | 시험 | 2026-07 | RC 05-21 → 최종 07-28. 무상태 코어(세션 ID 제거), Multi Round-Trip, Tasks, Apps 확장. "툴 프로토콜"에서 "에이전트 런타임 프로토콜"로 | `[2차]` 공식 changelog 재확인 권장 | [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) | [agentic-protocols-convergence](../06-trends/agentic-protocols-convergence.md) |
| T16 | A2A Linux Foundation 이관 | 시험 | 2025-06 | Google 소유 → LF, 100+ 기업. 조직 간 연동의 벤더 중립 선택지. 1.0은 정황만 | `[2차]` 복수 소스 일치 | [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) [1/05](../01-foundations/05-agentic-protocols.md) | [agentic-protocols-convergence](../06-trends/agentic-protocols-convergence.md) |
| T17 | AG-UI 대형 벤더 지원 | 시험 | 2026-03 | AWS Bedrock AgentCore(2026-03) · Microsoft .NET SDK(2026-09-25) · ThoughtWorks Radar. 거버넌스는 여전히 CopilotKit 단일 스타트업 | `[2차]` `[벤더]` | [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) [4/04](../04-dynamic-ui/04-streaming-state-sync.md) [4/07](../04-dynamic-ui/07-frontend-integration.md) | [agentic-protocols-convergence](../06-trends/agentic-protocols-convergence.md) |
| T18 | A2UI v0.9 | 관찰 | 2026-04 | 선언적 UI 트리(JSON). "Standard" → "Basic" 개명, Agent SDK, Android Jetpack. 모바일 네이티브 렌더 주장, 호환성 파괴 변경 진행 중 | `[벤더]` | [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) [4/07](../04-dynamic-ui/07-frontend-integration.md) | [generative-ui](../06-trends/generative-ui.md) |
| T19 | MCP Apps | 관찰 | 2026-01 | MCP 서버가 HTML UI 반환, 호스트가 샌드박스 iframe 렌더. 2026-07 MCP 확장으로 편입. 기본값은 "수용하지 않음" | `[2차]` | [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) [4/08](../04-dynamic-ui/08-security-sandboxing.md) [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) | [generative-ui](../06-trends/generative-ui.md) |
| T20 | **Generative UI 3패턴** | 시험 | 2026-01 | Controlled / Declarative / Open-ended, 각각 AG-UI / A2UI / MCP Apps. Controlled가 사실상 기본값 — 어휘는 채택, 프레이밍 출처는 AG-UI 주도사 | `[2차]` CopilotKit 프레이밍 · `[벤더]` | [4/01](../04-dynamic-ui/01-generative-ui-spectrum.md) [4/03](../04-dynamic-ui/03-component-registry-schema.md) [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) [4/10](../04-dynamic-ui/10-evaluation-genui.md) | [generative-ui](../06-trends/generative-ui.md) |
| T21 | Late chunking | 시험 | 2025-04 (비교 연구) | 장문 임베딩 후 분할. 효율↑ 관련성·완결성↓. 인덱싱 비용 제약 + 장문 임베딩 모델일 때만 조건부, 관련성 저하는 `[실측]` | `[검증]` arXiv 2504.19754 | [2/02](../02-data-pipeline/02-chunking-strategies.md) | [context-engineering](../06-trends/context-engineering.md) |
| T22 | Contextual retrieval | 시험 | 2025-04 (비교 연구) | 청크 앞에 문서 맥락 1문장 접두. 의미 일관성↑ 계산 비용↑. 출처 혼동 + 인덱싱 예산 있을 때 | `[검증]` 2504.19754 · 원형 `[벤더]` Anthropic | [2/02](../02-data-pipeline/02-chunking-strategies.md) | [context-engineering](../06-trends/context-engineering.md) |
| T23 | 하이브리드 검색 기본값화 | **채택** | 2025-10 | BM25 + dense + RRF(+ cross-encoder)가 단독보다 우수하다는 보고 누적. 2/04 · 3/03의 기본값. 각 요소를 끄고 켜며 `[실측]`하기 쉬움 | `[2차]` 누적 보고 — 유일하게 `[검증]` 없는 채택 항목(한계 참조) | [2/04](../02-data-pipeline/04-embedding-indexing.md) [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) | [graphrag-evolution](../06-trends/graphrag-evolution.md) (기준선 근거) |
| T24 | 롱컨텍스트 ↔ RAG 보완 관계 | **채택** | 2025-10 | "롱컨텍스트가 RAG를 대체" 기각. 롱컨텍스트는 양, RAG는 선택. RULER 스트레스에서 중간 청크 누락 재현 | `[검증]` 원 논문(Lost in the Middle·RULER) · `[2차]` 정리 | [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) [2/02](../02-data-pipeline/02-chunking-strategies.md) [3/08](../03-agent-workflow-rag/08-state-memory-context.md) | [context-engineering](../06-trends/context-engineering.md) |
| T25 | OTel GenAI 컨벤션 저장소 분리 | 관찰 | 2026-09 | 전 `gen_ai.*` 속성 Development 등급. v1.42.0에서 `semantic-conventions-genai` 분리, 태그 릴리스 없음. 직접 계측 대신 자체 스키마 + 매핑 계층 | `[2차]` 2026-06 "안정화" 1건은 레지스트리 미확인 | [5/03](../05-cross-cutting/03-observability-tracing.md) [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) | [1/06 평가·관측성](../01-foundations/06-evaluation-observability.md) |
| T26 | 프레임워크 재편 | 시험 | 2026-03 | Swarm → OpenAI Agents SDK · Claude Code SDK → Claude Agent SDK(hooks) · Google ADK 다언어(Java 1.0.0 2026-03) · AutoGen 유지보수 모드. 기본값이 "프레임워크 선택"에서 "밖 모듈 먼저"로 | `[2차]` | [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md) [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) [3/08](../03-agent-workflow-rag/08-state-memory-context.md) | [3/14 프레임워크 비교](../03-agent-workflow-rag/14-frameworks-landscape.md) |
| T27 | AutoGen 신규 채택 | 보류 | 2026 (월 미기록) | 유지보수 모드 전환 → 신규 프로젝트 비권장. 99 매트릭스 "피해야 할 조합" | `[2차]` | [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md) | [3/14 프레임워크 비교](../03-agent-workflow-rag/14-frameworks-landscape.md) |
| T28 | Agentic RAG | 시험 | 2025-10 | 검색이 에이전트 루프 "앞"이 아니라 "안"으로 — 검색도 툴이며 신뢰 경계·스키마 규칙을 따른다 | `[2차]` | [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) | [3/01 RAG 패턴](../03-agent-workflow-rag/01-rag-architecture-patterns.md) |
| T29 | 코드 실행형 행동 공간 (CodeAct / Code Mode / Programmatic Tool Calling) | 시험 | 2025-09 (Cloudflare Code Mode; Anthropic 2025-11) | 행동 공간을 JSON 툴 호출 1회에서 **실행 가능한 코드 1블록**으로. 원 논문(ICML 2024)은 기간 앞이나 상용화(Anthropic·Cloudflare·MS·smolagents 기본값)는 최근 1년. 절감 수치는 `[벤더]`이고 소스 간 차이 큼(독립 보도 단순 32% / 복잡 배치 81%). 설계 함의: 행동 게이트를 샌드박스 `call_tool` 경계에서, 생성 코드가 감사 레코드 | `[검증]` Wang et al. ICML 2024 · 절감 `[벤더]` `[2차]` | [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md) [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | 심화 페이지 없음 — [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md) "CodeAct" 소절, [연구 노트](../_meta/research-notes-2026-10.md) |

### 분면별 해설

| 분면 | 왜 여기인가 | 무엇이 확인되면 이동하는가 |
|---|---|---|
| **채택** (T23 · T24) | 둘 다 "기술"이라기보다 **정리된 결론**이다. 하이브리드는 보고가 누적되어 위키의 기본값이 되었고, 롱컨텍스트-RAG 보완은 원 논문이 `[검증]`이다 | 자체 코퍼스에서 하이브리드 각 요소의 기여가 `[실측]`되지 않으면 그 프로젝트에서는 "시험"으로 읽는다 |
| **시험** (17개) | 온톨로지·뉴로심볼릭은 표준(W3C, JSON Schema) 기반이라 기술 위험은 낮으나 **효과 주장**이 미검증. GraphRAG 계열·late/contextual chunking은 `[검증]` 벤치마크가 있지만 우위 범위가 좁거나 트레이드오프. 프로토콜·GenUI·프레임워크는 전환기(반년 단위 변경). 코드 실행형 행동 공간(T29)은 원 논문 `[검증]`(2024)이 있고 복수 벤더가 상용화했으나 절감 수치가 전부 `[벤더]`·소스 간 차이가 크고 샌드박스 운영이 전제 | 각 Part 6 페이지의 "채택 기준" 전부 `[실측]`. 공통: 요구 확인 → 대조 실험(없이/있이) → 비용·지연 → 운영 절차. T29: 자체 과업에서 Tool-use 대비 턴·토큰·성공률 `[실측]` + `call_tool` 게이트·샌드박스 운영 절차 1회 이상 가동 |
| **관찰** (8개) | 결정 모델 셋(T03~05)은 공개 한 달·전 수치 `[벤더]`·영어 중심. 메모리(T10~11)는 벤더 발 표준화 보고. A2UI(T18)는 v0.9 호환성 파괴 중, MCP Apps(T19)는 스펙 편입 직후, OTel GenAI(T25)는 Development 등급 | 결정 모델: 한국어 평가셋 착수 시 "시험". A2UI: 렌더러 동일성 `[실측]`. MCP Apps: 명시적 서드파티 UI 요구 + 샌드박스 `[실측]`. OTel: 태그 릴리스 또는 Stable 승격 |
| **보류** (T13 · T27) | 역결과(메모리 5종 모두 no-memory보다 나쁨) 또는 상태 종료(AutoGen 유지보수 모드). "쓰지 말라"가 아니라 "이 조건으로는 넣지 말라" | T13: 멀티세션 요구 확인 + no-memory 대비 `[실측]` → "시험"(그때는 T10과 합쳐진다). T27: 활성 개발 재개 |

### ★ 세 축의 현재 위치와 이동 조건

| 축 | 현재 분면 | 왜 | 다음 분면으로 가는 조건 (Part 6 "채택 기준" 요약) | 결정 트리 |
|---|---|---|---|---|
| **온톨로지** | 시험 | 기술은 W3C 표준(`[검증]`)이나 "에이전트 품질이 온톨로지로 달라진다"는 단일 저자 논문 1편 + 벤더 전언뿐. 벡터 + 택소노미로 골든셋을 통과하면 올리지 않는다 | ① 멀티홉·설명 질문 비율과 벡터 실패율 `[실측]` ② 자유 추출 vs 스키마 강제 추출 대조 `[실측]` ③ SHACL 거부율·정밀도 ④ 유지 절차 1회 이상 가동 ⑤ 설명 산출물 도메인 검수 | [Q6](02-decision-guide.md) |
| **뉴로심볼릭** | 시험 | 검증기·변환기 패턴은 실질 "채택"에 가깝다. 그러나 "뉴로심볼릭"이란 이름의 주장(NSAI 벤더 식별, 시장 규모)은 `[벤더]`이고 결합 전체의 V&V 미성숙은 `[검증]`된 결론 | ① 검증기 없이/있이 정답률·유해 출력률 `[실측]` ② 거부율·재생성 비용 ③ 의미 오류 누출률(스키마 통과했지만 틀림) ④ E2E 골든셋 상시 운영 ⑤ 지점별 패턴·기호 체계 명세 | [Q11](02-decision-guide.md) |
| **결정 모델 (Jev / Decisions API)** | 관찰 | 2026-09 등장. 지연·비용·보정 전부 `[벤더]`, Jev는 영어 중심 학습 명시, Decisions API는 날짜·가격 소스 충돌. 두 접근은 인터페이스만 같고 내부가 다르다 | 관찰 → 시험: 한국어 평가셋 착수. 시험 → 채택: ① 한국어 원문 평가셋에서 프롬프트 LLM·소형 분류기 둘 다 이상 ② 보정 곡선(ECE) ③ p50/p95 지연·비용 ④ 폴백 ⑤ 규칙 게이트 결합 ⑥ 둘을 따로 측정 ⑦ 어댑터 추상화 | [Q10](02-decision-guide.md) |

### 이번 1년의 구조적 변화 3가지

1. **판정의 독립 레이어화.** 라우팅·게이팅·승인 필요·UI 선택 같은 "판정"이 LLM 프롬프트 안에 섞여 있던 상태에서, 결정 모델의 등장(2026-09)과 함께 **별도 레이어**(레퍼런스 아키텍처의 3/11 점선 노드, `decide()` 추상화)로 승격되었다. Part 3/99의 기본값이 "소형 LLM 프롬프트"에서 "결정 모델 또는 분류기(`decide()`)"로, HITL이 이분법에서 확신도 세 갈래로 바뀐 것이 그 흔적이다. 기술 자체는 관찰이지만 **구조 변화는 이미 반영**되었다 — 인터페이스를 먼저 만들고 구현체는 나중에 고른다.
2. **온톨로지 회귀.** GraphRAG 시대의 "그래프를 만들면 멀티홉이 된다"가 "스키마 없는 그래프는 또 다른 비정형 데이터"로 바뀌었다(Neo4j NODES `[2차]`, 2604.00555). 동시에 RAGU 재분석(`[검증]`)이 그래프 RAG의 우위를 최난도 멀티홉으로 좁혀, "그래프 → 온톨로지 → 둘 다 요구 확인 뒤"라는 이 위키의 순서가 정해졌다. 택소노미(SKOS)에서 멈추는 프로젝트가 많다는 것도 같은 변화의 일부다.
3. **프로토콜의 경쟁 → 조합.** 2025년 초의 "MCP냐 A2A냐"가 A2A의 LF 이관(2025-06)과 MCP 2026-07-28 개정(무상태·Tasks·Apps)을 거치며 "경계마다 하나씩"으로 정리되었다. UI 층도 AG-UI(전송) · A2UI(내용) · MCP Apps(툴 UI)로 질문이 갈렸다. 실무적 의미는 "다섯 개를 다 쓰라"가 아니라 **"당사자가 둘 이상인 경계에만"** 이다.

네 번째 후보로 **평가 기준점의 이동**(메모리는 no-memory 대비, 그래프 RAG는 포맷 통제 후 비교)이 있으나, 한 분야의 교훈이 두 건이라 구조적 변화로는 아직 올리지 않았다. 다섯 번째 후보는 **행동 공간의 코드화**(T29) — 툴 호출이 모델 출력에서 샌드박스 안의 코드로 옮겨가며 "행동 직전 게이트"의 자리가 `call_tool` 경계로 이동하고 생성 코드가 감사 대상이 되었다. 세 벤더가 1년 안에 같은 구조로 수렴했지만 절감 수치가 `[벤더]`뿐이라, 구조 변화는 3/06·3/09·3/12에 반영하되 레이더에서는 시험에 둔다.

## 설계 시 고려사항 (체크리스트)

- [ ] 레이더에서 "시험" 항목을 PoC 후보로 고를 때, 결정 트리의 해당 질문(Q6·Q9·Q10·Q11·Q12·Q14)이 "예"로 떨어지는 변수 근거가 있는가
- [ ] "관찰" 항목에 대해 어댑터·인터페이스(`decide()`, 프로토콜 어댑터, OTel 매핑 계층)만 미리 두고 구현체 종속은 피했는가
- [ ] 벤더 수치(200배/400배, 61만 vs 100 토큰, $25.2B 시장)를 설계 문서의 근거로 쓰지 않았는가
- [ ] PoC 결과를 `[실측]`으로 Part 7 시나리오 페이지에 적고, Part 6 "벤더 주장 vs 검증" 표의 "우리의 판단"을 갱신했는가
- [ ] 분면을 옮길 때 CHANGELOG에 날짜·근거를 남겼는가
- [ ] 한국어 데이터로 재측정했는가 (결정 모델·메모리·엔티티 추출·토큰 효율은 영어 벤치마크가 전이되지 않는다)

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| **2차 소스 비중이 높다** — 29개 중 `[검증]`이 핵심 근거인 항목은 7개(T02 · T06 · T08 · T21 · T22 · T24 · T29 — 단 T29의 `[검증]`은 패턴 자체이고 절감 수치는 `[벤더]`) | 분면 자체가 흔들릴 수 있다. 특히 MCP 개정 세부(T15), MemTrapBench(T12), 결정 모델 날짜·가격(T05)은 소스 충돌 또는 원문 미확인 | 연구 노트의 "재확인 권장" 항목을 분기마다 원문 대조. 확인되면 검증 수준과 분면을 함께 조정 |
| **벤더 프레이밍이 어휘에 들어와 있다** — 3패턴(CopilotKit), 온톨로지 필수(Neo4j·Franz), 메모리 표준(Mem0), System One(TypeSafe) | 어휘를 쓰는 것만으로 벤더의 문제 설정을 받아들이게 된다 | 어휘는 채택하되 표의 "요지" 열에 출처 편향을 명시. 반대 증거(온톨로지 없는 LightRAG·HippoRAG 2 성과, no-memory 우위)를 같은 표에 둔다 |
| **채택 항목 T23이 `[검증]` 없이 채택이다** | 작성 가이드 5절의 예외처럼 보인다 | 하이브리드는 기술 선택이 아니라 "기준선"이며 각 요소 on/off `[실측]`이 쉬워 유지. 프로젝트별로 `[실측]`이 없으면 "시험"으로 읽는다 |
| **first_seen이 월 단위로 없는 항목이 5개** (T11 · T12 · T13 · T27 · 일부 2025-10 표기) | 레이더의 시간 축(HTML 반지름)이 거칠다 | "2025-10"은 트렌드 기간 시작점 표기임을 명시. 원문 확인 시 갱신 |
| **분면은 이 위키의 판단이지 업계 합의가 아니다** | 다른 팀의 레이더와 다를 수 있다 | 각 항목에 "왜 그 분면인가"를 적어 반박 가능하게. 이동 조건이 명시되어 있어 논쟁이 "`[실측]`이 있는가"로 수렴 |
| **한국어 관점이 과대 반영될 수 있다** | 영어 환경 프로젝트에는 결정 모델의 "관찰"이 보수적일 수 있다 | 영어 단일 환경이면 T03~05의 이동 조건 ①(한국어 평가셋)을 자체 언어 평가셋으로 치환 |
| **기간 경계가 자의적이다** — late chunking 비교 연구(2025-04), A2A 이관(2025-06)은 기간 앞 | 포함·제외 기준이 흔들려 보인다 | "이 기간에 기본값 담론이 되었거나 성격이 바뀐 것"으로 기준을 적고, 2024년 이전 기술은 Part 1로 보냈다 |

## 최신 트렌드 적용 (최근 1년)

이 페이지 자체가 트렌드의 위치를 정하는 곳이므로, 세 축의 역할은 위 "★ 세 축의 현재 위치와 이동 조건"에 있다. 요약:

| 트렌드 | 이 페이지에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 온톨로지 | T01. 시험. "온톨로지 없이 먼저 시험"이 채택 기준의 전제. 결정 트리 Q6의 리프를 정한다 | 멀티홉·동일성·설명 요구 `[실측]` | `[검증-단일저자]` `[벤더]` `[2차]` |
| 뉴로심볼릭 | T02. 시험. 검증기 패턴은 운영 표준, "뉴로심볼릭 구성이 품질을 개선했다"는 주장이 채택 대상. Q11 | 쓰기 툴·추출 적재 | `[검증]` V&V 한계 · `[벤더]` |
| 결정 모델 (Jev / Decisions API) | T03~05. 관찰. 구조적 변화(판정의 레이어화)는 이미 반영, 구현체는 `[실측]` 뒤. Q10 | 선택지 닫힘 + 한국어 평가셋 | `[벤더]` `[2차]` 소스 충돌 |
| 프로토콜 수렴 · GenUI | T14~T20. 시험/관찰. "당사자 둘 이상" · "Controlled부터". Q9 · Q14 | 경계·행위 확인 | `[2차]` `[벤더]` |

## 관련 페이지

- [Part 0 개요](00-overview.md) · [01 레퍼런스 아키텍처](01-reference-architecture.md)(트렌드 세 축이 걸치는 점선 노드) · [02 결정 트리](02-decision-guide.md)(Q6 · Q10 · Q11)
- [Part 6 개요](../06-trends/00-overview.md) — 분류 기준, "채택으로 올리는 절차", 결정 모델 두 접근 비교표
- Part 6 심화: [온톨로지 르네상스](../06-trends/ontology-renaissance.md) · [뉴로심볼릭](../06-trends/neuro-symbolic.md) · [결정 모델](../06-trends/decision-models.md) · [GraphRAG의 진화](../06-trends/graphrag-evolution.md) · [컨텍스트 엔지니어링](../06-trends/context-engineering.md) · [에이전트 메모리](../06-trends/agent-memory.md) · [프로토콜 수렴](../06-trends/agentic-protocols-convergence.md) · [Generative UI](../06-trends/generative-ui.md)
- [Part 1/02 System 1 vs System 2](../01-foundations/02-system1-vs-system2.md) · [Part 1/05 에이전트 프로토콜](../01-foundations/05-agentic-protocols.md) — 트렌드의 기반 지식
- [Part 7 시나리오](../07-appendix-scenarios/00-overview.md) — PoC `[실측]`을 기록하는 곳
- [`_meta/CHANGELOG.md`](../_meta/CHANGELOG.md) — 분면 이동 기록

## 참고 문헌

- 이 페이지의 모든 날짜·수치·검증 수준은 [`_meta/research-notes-2026-10.md`](../_meta/research-notes-2026-10.md)와 Part 6 각 페이지의 참고 문헌에서 가져왔으며 새로 추가한 사실은 없다
- `[검증]` RAGU, arXiv 2607.11683 (2026-07) — T08. 포맷 통제 후 MuSiQue 54.4 vs 40.1
- `[검증]` Renkhoff et al., IEEE TAI 2024 (arXiv 2401.03188) — T02. 결합 시스템 V&V 미성숙
- `[검증]` arXiv 2504.19754 — T21 · T22. Late chunking vs Contextual retrieval 트레이드오프
- `[검증-단일저자, 독립 재현 없음]` arXiv 2604.00555 (2026-03) — T01
- `[검증]` Wang et al., "Executable Code Actions Elicit Better LLM Agents", ICML 2024 (arXiv 2402.01030) — T29 원 패턴. 상용 구현(Anthropic 2025-11, Cloudflare 2025-09, Microsoft BUILD 2026, smolagents)과 절감 수치는 `[벤더]` `[2차]`
- `[벤더]` TypeSafe AI System One 문서, Google A2UI v0.9 (2026-04-17), Google ADK 문서, Mem0 보고, Franz Inc. (2026-06) — T03 · T04 · T18 · T09 · T11 · T01
- `[2차]` MCP 2026-07-28 해설(blog.mcpservers.org, Stacklok), A2A LF 이관 보도(2025-06), AG-UI 채택 보도(InfoWorld, Microsoft, AWS), CopilotKit 3패턴, MemTrapBench, OTel GenAI 저장소 분리(Dash0, 2026-09), Decisions API 해설(Hugging Face, Vercel 등) — 소스 충돌 항목은 표에 명시

_검증 수준 표기: `[검증]` `[벤더]` `[2차]` `[실측]` — 규칙은 [작성 가이드](../_meta/writing-guide.md) 참조_
