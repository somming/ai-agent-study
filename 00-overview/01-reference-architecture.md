---
title: "레퍼런스 아키텍처 (3-Layer)"
part: 00-overview
stage: all
tags: [architecture, reference, layers, contracts]
maturity: 채택
status: review
updated: 2026-10-11
---

# 레퍼런스 아키텍처 (3-Layer)

> **범위**: Knowledge / Reasoning·Action / Interaction 3계층 + 횡단 관심사로 본 AI Agent 시스템 전체 구조. 각 단계가 어느 페이지에서 다뤄지는지, 레이어 사이의 **계약**이 무엇인지. 개별 단계의 기술 선택은 각 Part에서.
> **시각화**: [`assets/html/reference-architecture.html`](../assets/html/reference-architecture.html) — 노드 클릭 시 해당 페이지로 이동

## 요약 (3줄)

1. AI Agent 시스템은 **무엇을 근거로**(Knowledge), **어떻게 추론·행동하고**(Reasoning/Action), **사람과 어떻게 주고받는가**(Interaction)의 세 레이어로 나뉘고, 보안·비용·관측성·거버넌스·운영이 세 레이어를 가로지른다.
2. 레이어 안의 기술 선택보다 **레이어 사이의 계약**(근거 단위 스키마, 렌더 가능한 응답 스키마, 사용자 이벤트 스키마)이 먼저다. 계약이 없으면 각 레이어가 따로 최적화되어 합쳐지지 않는다.
3. 최근 1년의 트렌드 세 축(온톨로지, 뉴로심볼릭, 결정 모델)은 특정 레이어의 기술이 아니라 **세 레이어 모두에 걸치는 횡단 패턴**이다. 그림에서 점선 노드가 그 자리다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** | 내 시스템의 범위를 이 그림 위에 표시한 "스코프 맵"과, 레이어 간 계약 3개의 초안 |
| **언제** | 프로젝트 착수 첫 주. 어떤 기술도 고르기 전에 |
| **어떻게** | ① 그림의 노드 중 내 시스템에 필요한 것을 표시(없어도 되는 것을 지우는 것이 더 중요) → ② 세 계약의 초안을 한 장씩 → ③ 각 Part의 `00-overview`로 이동해 단계별 선택 시작 → ④ 설계가 끝나면 [Part 5](../05-cross-cutting/00-overview.md)로 횡단 검토 |
| **왜** | "RAG를 만들자"로 시작하면 Knowledge 레이어만 보인다. 사용자가 결과를 조작해야 하는지(Interaction), 어떤 판단이 행동으로 이어지는지(Action), 누가 승인하는지(HITL)가 빠진 채 검색 품질만 올리다가 끝난다 |

## 문제와 맥락

에이전트 프로젝트가 실패하는 전형적 경로는 **한 레이어에서 시작해 거기서 끝나는 것**이다. 검색 팀은 recall을 올리고, 프롬프트 팀은 답변 톤을 다듬고, 프론트 팀은 채팅창을 예쁘게 만든다. 그런데 검색이 돌려주는 청크에는 출처 ID가 없어 답변에 citation을 붙일 수 없고, 답변은 자유 텍스트라 프론트가 표로 보여줄 수 없으며, 사용자가 "이 항목 수정"을 눌러도 에이전트 상태로 돌아갈 경로가 없다. 레이어는 다 있는데 **계약이 없다**.

레퍼런스 아키텍처의 역할은 기술을 정해 주는 것이 아니라 **계약을 정할 자리를 보여 주는 것**이다.

## 핵심 개념

### 세 레이어

| 레이어 | 질문 | 입력 | 출력 | Part |
|---|---|---|---|---|
| **Knowledge** | 무엇을 근거로 판단하는가 | 원천 데이터(문서, DB, 로그, API) | 근거 단위(evidence) + 인덱스 + 접근 정책 | [Part 2](../02-data-pipeline/00-overview.md) |
| **Reasoning / Action** | 어떻게 추론하고 행동하는가 | 사용자 요청 + 근거 + 상태 + 툴 | 렌더 가능한 응답(텍스트 + citations + ui_intent) + 행동(툴 실행) | [Part 3](../03-agent-workflow-rag/00-overview.md) |
| **Interaction** | 결과를 사람과 어떻게 주고받는가 | 렌더 가능한 응답 | 렌더된 UI + 사용자 이벤트(승인·수정·선택) | [Part 4](../04-dynamic-ui/00-overview.md) |
| **Cross-cutting** | 세 레이어에 일관되게 적용할 정책 | — | 보안·비용·관측·거버넌스·운영 정책 | [Part 5](../05-cross-cutting/00-overview.md) |

### 단계 → 페이지 매핑

**Knowledge** (Part 2)

| 단계 | 페이지 | 산출물 |
|---|---|---|
| 수집·파싱 | [2/01](../02-data-pipeline/01-source-ingestion.md) | 정규화된 문서·레코드 |
| 청킹 | [2/02](../02-data-pipeline/02-chunking-strategies.md) | 청크 + 구조 정보 |
| 메타데이터·출처 | [2/03](../02-data-pipeline/03-metadata-provenance.md) | 출처·버전·권한 메타데이터 |
| 임베딩·인덱싱 | [2/04](../02-data-pipeline/04-embedding-indexing.md) | 벡터·키워드 인덱스 |
| 지식그래프 구축 | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) | 엔티티·관계 그래프 |
| 온톨로지 모델링 (점선) | [2/06](../02-data-pipeline/06-ontology-driven-modeling.md) | 클래스·관계·제약 정의 |
| 신선도·동기화 / 접근 제어 | [2/07](../02-data-pipeline/07-freshness-and-sync.md), [2/10](../02-data-pipeline/10-access-control-governance.md) | 갱신 파이프라인, ACL 정책 |
| **근거 데이터 설계 ★** | [2/08](../02-data-pipeline/08-grounding-data-design.md) | **근거 단위 스키마 (계약 ①)** |
| 파이프라인 내부의 AI (점선) | [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) | 분류·정제·판정 결과 |

**Reasoning / Action** (Part 3)

| 단계 | 페이지 | 산출물 |
|---|---|---|
| 질의 이해·라우팅 | [3/02](../03-agent-workflow-rag/02-query-understanding.md) | 재작성된 질의, 경로 |
| 검색·리랭킹 | [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) | 선별된 근거 |
| 생성·근거 강제 | [3/04](../03-agent-workflow-rag/04-generation-grounding.md) | 답변 + citations |
| 툴 / MCP | [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) | 툴 실행 결과 |
| 가드레일·HITL | [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | 허가/차단/승인 요청 |
| 결정 레이어 (점선) ★ | [3/11](../03-agent-workflow-rag/11-decision-layer.md) | 판정 + 확신도 |
| 뉴로심볼릭 검증기 (점선) | [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) | 검증 통과/실패 |
| 상태·메모리·컨텍스트 | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) | 상태 스냅샷 |
| 오케스트레이션 | [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) | 에이전트 간 메시지 |
| 워크플로우 vs 에이전트 | [3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md) | 실행 그래프 |

**Interaction** (Part 4)

| 단계 | 페이지 | 산출물 |
|---|---|---|
| 의도 → UI 매핑 (점선) | [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) | 컴포넌트 선택 |
| 컴포넌트 레지스트리 | [4/03](../04-dynamic-ui/03-component-registry-schema.md) | **렌더 가능한 응답 스키마 (계약 ②)** |
| 스트리밍·상태 동기화 | [4/04](../04-dynamic-ui/04-streaming-state-sync.md) | 이벤트 스트림, **사용자 이벤트 스키마 (계약 ③)** |
| HITL UI | [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) | 승인·수정·피드백 UI |
| 프론트엔드 렌더러 | [4/07](../04-dynamic-ui/07-frontend-integration.md) | 렌더된 화면 |
| 보안·샌드박싱 | [4/08](../04-dynamic-ui/08-security-sandboxing.md) | 샌드박스 정책 |

**Cross-cutting** (Part 5): [보안](../05-cross-cutting/01-security-prompt-injection.md) · [비용·라우팅](../05-cross-cutting/02-cost-latency-model-routing.md) · [관측성](../05-cross-cutting/03-observability-tracing.md) · [거버넌스](../05-cross-cutting/04-governance-explainability.md) · [LLMOps](../05-cross-cutting/05-llmops-agentops.md)

### 레이어 간 계약 3개

| 계약 | 방향 | 정의하는 곳 | 소비하는 곳 | 최소 내용 |
|---|---|---|---|---|
| **① 근거 단위 (evidence)** | Knowledge → Reasoning | [2/08](../02-data-pipeline/08-grounding-data-design.md) | [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md), [3/04](../03-agent-workflow-rag/04-generation-grounding.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) | 근거 ID, 본문(또는 트리플), 출처(문서·위치·버전·시점), 권한 태그, 신뢰도 등급 |
| **② 렌더 가능한 응답** | Reasoning → Interaction | [4/03](../04-dynamic-ui/03-component-registry-schema.md), [3/04](../03-agent-workflow-rag/04-generation-grounding.md) | [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md), [4/07](../04-dynamic-ui/07-frontend-integration.md) | 텍스트, citations(계약 ① 참조), ui_intent(선택적: 컴포넌트명 + 데이터), 확신도, 승인 필요 여부 |
| **③ 사용자 이벤트** | Interaction → Reasoning | [4/04](../04-dynamic-ui/04-streaming-state-sync.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | [3/08](../03-agent-workflow-rag/08-state-memory-context.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | 이벤트 유형(승인·거절·수정·선택·피드백), 대상(응답 ID·컴포넌트 ID), 페이로드, 사용자·시각, 서버 재검증 결과 |

계약은 **스키마 파일**로 저장소에 두고, 세 레이어의 테스트가 같은 스키마를 참조하게 한다. 계약이 바뀌면 세 레이어가 동시에 바뀐다는 뜻이므로 변경 절차를 [Part 5/05](../05-cross-cutting/05-llmops-agentops.md)에 둔다.

### 트렌드 세 축이 걸치는 자리

| 트렌드 | Knowledge | Reasoning / Action | Interaction | Cross-cutting |
|---|---|---|---|---|
| **온톨로지 / KG** | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md), [2/06](../02-data-pipeline/06-ontology-driven-modeling.md), [2/08](../02-data-pipeline/08-grounding-data-design.md) | [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) | [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) | [5/04](../05-cross-cutting/04-governance-explainability.md) |
| **뉴로심볼릭** | [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) | [3/04](../03-agent-workflow-rag/04-generation-grounding.md), [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | [4/03](../04-dynamic-ui/03-component-registry-schema.md) (UI 트리 검증) | [5/01](../05-cross-cutting/01-security-prompt-injection.md) |
| **결정 모델** | [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) | [3/02](../03-agent-workflow-rag/02-query-understanding.md), [3/11](../03-agent-workflow-rag/11-decision-layer.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) | [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) | [5/02](../05-cross-cutting/02-cost-latency-model-routing.md) |

## 아키텍처 / 파이프라인

요청 1건이 흐르는 경로 (시각화의 화살표와 같다):

1. 사용자 요청 → **Interaction**이 이벤트로 받아 → **Reasoning**의 상태 머신 입력
2. 질의 이해·라우팅(결정 모델) → 검색 → **Knowledge**에서 근거 단위(계약 ①) 반환
3. 생성·근거 강제 → 뉴로심볼릭 검증 → 가드레일·HITL 판정
4. 렌더 가능한 응답(계약 ②) → **Interaction**이 의도→UI 매핑 → 렌더
5. 사용자가 승인·수정 → 사용자 이벤트(계약 ③) → **Reasoning** 상태 머신 → (툴 실행) → 2~5 반복
6. 전 과정이 **Cross-cutting**의 트레이스·비용 계측·정책 검사를 통과

## 기술 비교표 — 아키텍처 범위의 선택지

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| Knowledge만 (검색 API, 답변 없음) | 기존 검색 UI에 의미 검색만 추가 | 사용자가 답변·행동을 원함 | 하 | 표준 | — |
| Knowledge + Reasoning (RAG QA, 텍스트 응답) | 읽기 중심 질의응답, citation 표시 | 사용자가 결과를 조작·승인해야 함 | 중 | 안정 | — |
| 3-Layer 전체 (에이전트 + 동적 UI) | 사용자가 선택·승인·수정, 행동(툴 실행)이 있음 | 단순 QA, 소규모 팀의 첫 프로젝트 | 상 | Interaction 레이어는 시험 단계 | — |
| Reasoning + Interaction (지식 없음) | 모델 지식·툴만으로 충분한 과업(변환, 코드, 폼 작성) | 사실 근거가 필요한 도메인 | 중 | 안정 | — |

### 선택 가이드

- **사용자가 읽기만 한다면** → Knowledge + Reasoning. Interaction은 citation 렌더(4/06 일부)만
- **사용자가 선택·승인·수정해야 한다면** → 3-Layer 전체. 단 Interaction은 Controlled 패턴([4/01](../04-dynamic-ui/01-generative-ui-spectrum.md))부터
- **행동(툴 실행)이 있다면** → Reasoning의 결정 레이어·검증기·가드레일(3/10~12)은 생략 불가
- **판단이 어려우면** → Knowledge + Reasoning으로 시작하되 계약 ②·③을 미리 정의. 이유: 나중에 Interaction을 붙일 때 Reasoning을 다시 만들지 않아도 된다

## 설계 시 고려사항 (체크리스트)

- [ ] 그림 위에 내 시스템의 범위를 표시했고, **지운 노드**마다 지운 이유가 있는가
- [ ] 계약 ①②③의 스키마 초안이 저장소에 있는가
- [ ] 각 레이어의 담당(팀·사람)이 정해져 있고, 계약 변경 시 세 레이어가 함께 바뀌는 절차가 있는가
- [ ] 행동(툴 실행)이 있다면 결정 레이어·검증기·가드레일이 경로 위에 있는가
- [ ] 트레이스가 세 레이어를 가로질러 한 요청을 추적하는가 ([5/03](../05-cross-cutting/03-observability-tracing.md))
- [ ] 트렌드 세 축 중 쓰기로 한 것과 안 쓰기로 한 것이 각각 이유와 함께 기록돼 있는가

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| 레퍼런스가 "전부 다 있는 그림"이라 과설계 유도 | 필요 없는 레이어·단계를 만듦 | 지우는 것부터. 가장 단순한 범위에서 시작 |
| 계약을 스키마가 아닌 문서로만 둠 | 레이어 간 불일치가 런타임에 발견 | 스키마 파일 + 레이어별 계약 테스트 |
| Interaction 레이어의 미성숙 | 프로토콜·SDK 변경으로 재작업 | Controlled 패턴, 어댑터 계층, 버전 고정 |
| 트렌드 세 축을 "써야 하는 것"으로 오해 | 온톨로지·결정 모델을 요구 없이 도입 | 각 Part의 "언제" 조건 확인, `[실측]` 없이는 채택 안 함 |

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 온톨로지 | 계약 ①의 근거 단위에 엔티티·관계를 포함할지 결정하는 지점. 계약 ②의 ui_intent 선택지를 도메인 개념에서 가져올지 | 멀티홉·설명·도메인 UI 요구 | `[2차]` |
| 뉴로심볼릭 | 계약 ②(응답)와 ③(이벤트)이 레이어 경계를 넘을 때 스키마·규칙 검증을 거치는 구조. 경계가 곧 검증 지점 | 행동이 있는 시스템 | `[2차]` |
| 결정 모델 (Jev / Decisions API) | 그림의 점선 노드 3곳(2/09, 3/11, 4/05)과 횡단(5/02). "판정"이 독립 레이어 요소로 승격된 것이 최근 1년의 구조적 변화 | 선택지 닫힌 판정 + 한국어 실측 | `[벤더]` `[2차]` |
| 에이전트 프로토콜 수렴 | 레이어 경계마다 개방 프로토콜(툴: MCP, 에이전트 간: A2A, 프론트-백: AG-UI, UI 내용: A2UI) 선택지 | 경계의 당사자가 둘 이상 | `[2차]` [1/05](../01-foundations/05-agentic-protocols.md) |

## 관련 페이지

- [Part 0 개요](00-overview.md), [02 종합 기술 선택 결정 트리](02-decision-guide.md), [03 트렌드 레이더](03-trend-radar-2026.md)
- 각 Part `00-overview.md`
- [Part 7 시나리오](../07-appendix-scenarios/00-overview.md) — 이 그림에 실제 선택을 채운 예

## 참고 문헌

- 이 페이지는 위키 내부 구조를 정의하므로 외부 문헌 인용 없음. 트렌드 사실은 [`_meta/research-notes-2026-10.md`](../_meta/research-notes-2026-10.md)
