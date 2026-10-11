---
title: "Part 3. Agent 워크플로우 설계 및 LLM/RAG 파이프라인 아키텍처"
part: 03-agent-workflow-rag
stage: reasoning
tags: [overview, agent, rag, workflow]
maturity: 채택
status: draft
updated: 2026-10-11
---

# Part 3. Agent 워크플로우 설계 및 LLM/RAG 파이프라인 아키텍처

> **시각화**: [`assets/html/agent-workflow-architecture.html`](../assets/html/agent-workflow-architecture.html)
> Reasoning / Action Layer. 에이전트가 **어떻게 추론하고 행동하는지**를 설계한다. 이 Part의 핵심 결정은 두 가지다. ① 어디까지를 결정론적 워크플로우로 고정하고 어디부터 모델에 맡길 것인가. ② 어떤 판단을 생성 모델(System 2)이 아닌 결정 모델·규칙(System 1)에 맡길 것인가.

## 이 Part의 역할

| 질문 | 답 |
|---|---|
| **무엇을** | 질의 → 라우팅 → 검색 → 추론 → 툴 실행 → 검증 → 응답에 이르는 실행 그래프와, 그 안의 상태·메모리·가드레일 |
| **언제** | 항상. 단, 단일 프롬프트로 끝나는 과업은 05(워크플로우 vs 에이전트)만 읽고 가장 단순한 형태를 택한다 |
| **어떻게** | 05에서 자율성 수준 결정 → 01에서 RAG 패턴 결정 → 02~04로 검색·생성 세부 → 11·10으로 판정·검증 레이어 삽입 → 08·09·12로 상태·툴·안전장치 → 13으로 평가 → 14로 프레임워크 |
| **왜** | 에이전트를 "LLM에 툴을 붙인 것"으로 설계하면 비용·지연·비결정성이 통제되지 않는다. 결정론적 부분과 확률적 부분의 경계를 명시해야 운영 가능하다 |

## 단계와 각 단계의 기술 선택지

| 순서 | 단계 | 페이지 | 이 단계에서 비교하는 기술 |
|---|---|---|---|
| 1 | RAG 아키텍처 패턴 | [01](01-rag-architecture-patterns.md) | Naive / Advanced / Modular / Agentic RAG, GraphRAG 계열, 롱컨텍스트 직접 주입, 하이브리드 |
| 2 | 질의 이해·라우팅 | [02](02-query-understanding.md) | 쿼리 재작성, 분해, HyDE, step-back; 라우터: 프롬프트 LLM vs 분류기 vs **결정 모델** vs 규칙 |
| 3 | 검색·리랭킹 | [03](03-retrieval-reranking.md) | hybrid + RRF, cross-encoder vs LLM rerank, 컨텍스트 압축, 그래프 탐색 결합 |
| 4 | 생성·근거 강제 | [04](04-generation-grounding.md) | citation 강제 방식(프롬프트 vs 구조화 출력 vs 후검증), 환각 검출기, 뉴로심볼릭 검증 |
| 5 | 워크플로우 vs 에이전트 | [05](05-workflow-vs-agent.md) | 프롬프트 체이닝 / 라우팅 / 병렬화 / 오케스트레이터-워커 / 평가자-최적화 / 자율 에이전트 |
| 6 | 에이전트 설계 패턴 | [06](06-agent-design-patterns.md) | ReAct, Plan-and-Execute, Reflection, Tool-use, Router, Orchestrator-Worker |
| 7 | 멀티 에이전트 | [07](07-multi-agent-orchestration.md) | Supervisor / Hierarchical / Handoff(Swarm) / 그래프; A2A; 공유 상태 vs 메시지 |
| 8 | 상태·메모리·컨텍스트 | [08](08-state-memory-context.md) | 체크포인팅, 단기/장기 메모리, 메모리 프레임워크(Mem0, cognee, Letta 등), compaction, 컨텍스트 엔지니어링 |
| 9 | 툴·MCP | [09](09-tools-and-mcp.md) | 직접 함수 호출 vs MCP 서버; 툴 스키마 설계; 멱등성·재시도; 권한 모델 |
| 10 | 뉴로심볼릭 추론 | [10](10-neuro-symbolic-reasoning.md) | LLM + 규칙엔진 / 온톨로지 reasoner / SMT·제약 솔버 / 프로그램 합성(PAL) / KG 제약 |
| 11 | **결정 레이어 ★** | [11](11-decision-layer.md) | TypeSafe Jev vs OpenAI Decisions API vs 파인튜닝 분류기 vs 프롬프트 LLM; 배치 위치(라우팅·게이팅·다음 행동·승인 필요 판정) |
| 12 | 가드레일·HITL | [12](12-guardrails-hitl.md) | 입출력 가드레일 라이브러리, 정책 엔진(OPA 등), 승인 흐름, 결정 모델 기반 게이팅 |
| 13 | 평가 | [13](13-evaluation-rag-agents.md) | RAG 지표(faithfulness, context precision 등), LLM-as-judge, 트래젝토리 평가, 회귀 테스트 |
| 14 | 프레임워크 | [14](14-frameworks-landscape.md) | LangGraph, Claude Agent SDK, OpenAI Agents SDK, Google ADK, CrewAI, Semantic Kernel, Pydantic AI, 직접 구현 |
| — | 결정 매트릭스 | [99](99-decision-matrix.md) | 요구사항 × 패턴 × 프레임워크 종합 |

## 이 Part를 관통하는 설계 원칙

1. **가장 단순한 형태에서 시작한다.** 워크플로우로 충분한 것을 에이전트로 만들지 않는다. 자율성은 비용·지연·비결정성의 대가가 있다
2. **판정과 생성을 분리한다.** "다음에 무엇을 할까", "이 요청은 어느 경로인가", "사람 승인이 필요한가"는 생성 과업이 아니다. 결정 모델·분류기·규칙이 먼저고, LLM은 이들이 못 하는 것만 맡는다
3. **근거는 입력에서 강제하고 출력에서 검증한다.** 프롬프트로 "출처를 밝혀라"만 하면 부족하다. 구조화 출력으로 citation을 받고, 뉴로심볼릭 검증기로 근거-답변 정합성을 확인한다
4. **상태는 명시적으로 모델링한다.** 대화 히스토리를 그대로 쌓는 것은 상태 관리가 아니다. 컨텍스트 엔지니어링은 무엇을 넣을지보다 **무엇을 뺄지**의 설계다
5. **메모리는 no-memory 베이스라인과 비교해 채택한다.** 메모리가 이전 과업의 프레임에 모델을 고착시켜 성능을 떨어뜨렸다는 벤치마크(MemTrapBench)가 있다 `[2차]`

## 최근 1년 트렌드가 이 Part에 미친 영향

| 트렌드 | 영향 받는 단계 | 요지 | 검증 수준 |
|---|---|---|---|
| **결정 모델 등장** | 02, 11, 12 | Jev(비자기회귀, 타입드 질문→보정 확률, 2026-09-15 EA)와 OpenAI Decisions API(`/v1/decisions`, gpt-6-luna 기반, 2026-09-29 발표·10월 초 베타). 라우팅·게이팅의 기본 수단이 "작은 LLM 프롬프트"에서 바뀌는 중 | `[벤더]` `[2차]` — 소스 간 날짜 충돌 있음 |
| 뉴로심볼릭 | 04, 10, 12 | 온톨로지 3계층(Role/Domain/Interaction)으로 650+ 에이전트를 접지했다는 보고(arXiv 2604.00555) | `[검증-단일저자]` |
| 컨텍스트 엔지니어링 | 08 | Google ADK가 세션·메모리·툴 출력·아티팩트를 구조화된 뷰로 조립, 자동 요약·지연 로딩 | `[벤더]` |
| 에이전트 메모리 분화 | 08 | 대화 컨텍스트용 vs 운영 지식용으로 프레임워크가 갈림. LoCoMo/LongMemEval이 표준 벤치마크화 | `[벤더]` |
| Agentic RAG | 01, 03 | 검색이 에이전트 루프 "앞"이 아니라 "안"으로 들어감 | `[2차]` |
| 프레임워크 재편 | 14 | Swarm→OpenAI Agents SDK, Claude Code SDK→Claude Agent SDK, AutoGen 유지보수 모드, ADK 다언어화(Java 1.0 2026-03) | `[2차]` |

## 다음 Part로 넘기는 것

이 Part의 출력은 **렌더 가능한 응답 스키마**(텍스트 + citation + 선택적 UI 의도)와 **사용자 이벤트를 받는 상태 머신**이다. [Part 4](../04-dynamic-ui/00-overview.md)가 이것을 소비한다.

## 관련 페이지

- [Part 1/02 System 1 vs System 2](../01-foundations/02-system1-vs-system2.md), [Part 1/04 뉴로심볼릭](../01-foundations/04-neuro-symbolic-ai.md)
- [Part 6/decision-models](../06-trends/decision-models.md), [Part 6/context-engineering](../06-trends/context-engineering.md)
- [시나리오 C: 운영 인시던트 에이전트](../07-appendix-scenarios/03-ops-incident-agent.md) — 결정 레이어와 HITL이 핵심인 예시
