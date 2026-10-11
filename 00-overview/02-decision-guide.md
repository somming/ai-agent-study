---
title: "종합 기술 선택 결정 트리"
part: 00-overview
stage: all
tags: [decision-tree, decision-guide, defaults, requirements, input-variables, trends]
maturity: 채택
status: review
updated: 2026-10-11
---

# 종합 기술 선택 결정 트리

> **범위**: 프로젝트 요구사항에서 출발해 각 단계의 기술을 고르는 질문 기반 결정 트리
> **시각화**: [`assets/html/decision-tree.html`](../assets/html/decision-tree.html) — 질문을 클릭하면 예/아니오 분기가 펼쳐지고, 리프에서 해당 페이지로 이동. 질문 번호는 이 페이지와 동일

## 요약 (3줄)

1. 세 결정 매트릭스([Part 2/99](../02-data-pipeline/99-decision-matrix.md), [Part 3/99](../03-agent-workflow-rag/99-decision-matrix.md), [Part 4/99](../04-dynamic-ui/99-decision-matrix.md))의 입력 변수를 **15개로 중복 제거**하고, 질문 15개를 **범위 → Knowledge → Reasoning → Interaction → 횡단** 순서로 한 그루에 묶었다.
2. 모든 질문의 답은 Part 2·3·4의 특정 페이지와 그 페이지의 **기본값**으로 떨어진다. 기본값은 각 Part의 99 매트릭스 "판단 어려울 때" 열을 그대로 쓴다(어긋나는 4곳은 아래에 적었다).
3. 트렌드 세 축(온톨로지 · 뉴로심볼릭 · 결정 모델)은 Q6 · Q10 · Q11에서 묻는다. 셋 다 **"요구가 `[실측]`으로 확인되었는가"** 가 분기 조건이며, 확인 전 기본값은 항상 "도입하지 않는 쪽"이다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | ① 입력 변수 15개에 프로젝트의 값을 채운 표 ② 질문 15개를 돌린 경로 ③ 단계별 선택·대안·이유·근거 페이지를 적은 **결정 기록**. 품질 기준: 모든 선택에 근거 페이지가 있고, 기본값에서 벗어난 선택마다 "어느 변수 때문에"가 적혀 있다 |
| **언제** (When) | [01 레퍼런스 아키텍처](01-reference-architecture.md)로 범위를 표시한 직후, 각 Part의 `00-overview`로 들어가기 전. 생략 가능: 기존 시스템에 단계 하나만 추가할 때(해당 Part 99 매트릭스만 본다). 미뤄도 되는 것: Interaction 질문(Q13~14)은 Knowledge+Reasoning 범위가 확정된 뒤로 |
| **어떻게** (How) | ① 입력 변수 표 채우기(모르면 "미정"으로 두고 가장 보수적 값으로 진행) → ② Q1부터 순서대로 답하고 리프의 기본값을 받아 적기 → ③ 기본값과 다른 선택을 하고 싶은 단계는 해당 99 매트릭스의 "조건 A/B"에 내 변수가 걸리는지 확인 → ④ 결정 기록 표 작성 → ⑤ 가장 가까운 [Part 7 시나리오](../07-appendix-scenarios/00-overview.md)와 비교해 빠진 단계 점검 → ⑥ 3개월 뒤 변수 값이 바뀌었는지 재확인 |
| **왜** (Why) | 단계별 페이지만 읽으면 각 단계가 자기 기준으로 "최선"을 고르고, 그 결과 Knowledge는 트리플을 만들고 Reasoning은 청크만 받고 Interaction은 표를 그릴 데이터가 없는 상태가 된다. 한 그루의 트리로 돌려야 **같은 변수가 여러 단계를 일관되게 움직인다** |

## 문제와 맥락

Part 2·3·4의 결정 매트릭스는 각각 입력 변수 9~10개를 정의한다. 셋을 나란히 놓으면 **같은 변수가 이름만 다르게 세 번** 나온다 — Part 2의 "정확도·근거 요구"와 Part 3의 "근거 요구", Part 3의 "실패 비용"과 Part 4의 "행동의 위험도", Part 2의 "질의 유형(멀티홉)"과 Part 4의 "온톨로지 보유"가 그렇다. 변수가 셋으로 흩어져 있으면 프로젝트가 Part 2에서는 "참고용"으로, Part 4에서는 "승인 대상"으로 답하는 모순이 발견되지 않는다.

이 페이지는 변수를 한 번만 묻고, 그 답이 세 레이어 어디로 흘러가는지 보여 준다. 기술을 새로 고르지는 않는다 — 고르는 일은 각 99 매트릭스와 단계 페이지의 몫이고, 여기서는 **어느 페이지의 어느 기본값으로 가야 하는지** 안내한다.

## 핵심 개념

### 입력 변수 (세 매트릭스 통합, 중복 제거)

| # | 변수 | 값의 범위 | 영향을 받는 단계 | 출처 |
|---|---|---|---|---|
| V1 | **데이터 유형** | 정형(DB·API) / 반정형(표·티켓·로그) / 비정형(문서·메일·채팅) / 멀티모달(스캔·도표·이미지) | [2/01](../02-data-pipeline/01-source-ingestion.md) 파서, [2/02](../02-data-pipeline/02-chunking-strategies.md) 청킹 기준, [2/04](../02-data-pipeline/04-embedding-indexing.md) sparse 비중, [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) 추출 방식, [2/08](../02-data-pipeline/08-grounding-data-design.md) 근거 단위, [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md), [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) 변환기 | P2 "데이터 유형", P3 "데이터 형태" |
| V2 | **규모·갱신 빈도** | 문서 수 천 / 십만 / 백만 이상 · 갱신 일 / 시간 / 분 · 트래픽 소량 / 대량 | [2/04](../02-data-pipeline/04-embedding-indexing.md) 저장소·양자화, [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) 인덱싱 방식, [2/07](../02-data-pipeline/07-freshness-and-sync.md), [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) 판정 비용, [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) 선택 수단 | P2 "규모", P4 "트래픽" |
| V3 | **지연 요구** | 응답 실시간(<1s) / 대화형(<5s) / 배치 · 신선도 SLO 분 / 시간 / 일 | [2/01](../02-data-pipeline/01-source-ingestion.md) CDC vs 폴링, [2/07](../02-data-pipeline/07-freshness-and-sync.md), [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md), [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) 리랭커, [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md) 루프 횟수, [3/11](../03-agent-workflow-rag/11-decision-layer.md) | P2 · P3 · P4 공통 |
| V4 | **정확도·근거 요구** | 참고용 / 업무 지원 / citation 필수 | [2/03](../02-data-pipeline/03-metadata-provenance.md) 계보 깊이, [2/08](../02-data-pipeline/08-grounding-data-design.md) 신뢰도·상충, [3/04](../03-agent-workflow-rag/04-generation-grounding.md) 후검증, [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) faithfulness | P2 "정확도·근거", P3 "근거 요구" |
| V5 | **규제·감사 여부** | 없음 / 내부 감사 / 법규·조달(접근성 요건 포함) | [2/03](../02-data-pipeline/03-metadata-provenance.md) bitemporal, [2/10](../02-data-pipeline/10-access-control-governance.md) 감사 로그, [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md), [4/09](../04-dynamic-ui/09-ux-accessibility.md), [4/10](../04-dynamic-ui/10-evaluation-genui.md) A/B 금지, [5/04](../05-cross-cutting/04-governance-explainability.md) | P2, P4 "디자인 시스템 엄격도(법규)" |
| V6 | **멀티홉·동일성·설명 요구** | 없음 / 멀티홉·관계 질의 / 엔티티 동일성 / 근거 경로 설명 — 반드시 질의 로그·골든셋 `[실측]`으로 | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) 그래프 도입, [2/06](../02-data-pipeline/06-ontology-driven-modeling.md) 온톨로지, [2/08](../02-data-pipeline/08-grounding-data-design.md) 트리플, [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) GraphRAG 계열, [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md), [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) 온톨로지 바인딩 | P2 "질의 유형", P4 "온톨로지 보유" |
| V7 | **과업 예측 가능성** | 고정 절차 / 몇 가지 분기 / 열린 탐색 | [3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md), [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md), [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md), [3/08](../03-agent-workflow-rag/08-state-memory-context.md) | P3 |
| V8 | **실패 비용** | 참고용(사용자가 거름) / 업무 지원(수정 비용) / 금전·법적·비가역 | [3/04](../03-agent-workflow-rag/04-generation-grounding.md), [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md), [3/11](../03-agent-workflow-rag/11-decision-layer.md) θ, [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md), [4/01](../04-dynamic-ui/01-generative-ui-spectrum.md) 통제 수준, [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) 승인 카드, [4/08](../04-dynamic-ui/08-security-sandboxing.md) | P3 "실패 비용", P4 "행동의 위험도" |
| V9 | **행동(툴 실행) 유무·부수효과** | 0 / 읽기 전용 소수 / 쓰기 포함 20개 이하 / 20개 초과·외부 전송 | [3/08](../03-agent-workflow-rag/08-state-memory-context.md) 멱등·체크포인트, [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md), [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) 게이트, [3/11](../03-agent-workflow-rag/11-decision-layer.md), [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md), [5/01](../05-cross-cutting/01-security-prompt-injection.md) | P3 "툴 수와 부수효과" |
| V10 | **사용자 행위** | 읽기만 / 선택·비교 / 입력·수정 / 승인(행동으로 이어짐) | [01 레퍼런스 아키텍처](01-reference-architecture.md) 범위, [4/01](../04-dynamic-ui/01-generative-ui-spectrum.md), [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) — "읽기만"이면 Part 4 대부분 불필요 | P4 "사용자 행위" |
| V11 | **플랫폼** | 웹만 / 웹 + 모바일 웹 / iOS·Android 네이티브 / 기존 앱 임베딩 / 여러 호스트(IDE·챗·포털) | [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md), [4/07](../04-dynamic-ui/07-frontend-integration.md), [4/08](../04-dynamic-ui/08-security-sandboxing.md) | P4 |
| V12 | **조직 경계** | 툴·에이전트·프론트의 당사자 수: 단일 팀·단일 스택 / 프론트·백 분리 또는 여러 팀 / 타 조직·타 벤더 에이전트 / 서드파티 툴 UI | [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) A2A, [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) MCP, [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md), [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) AG-UI·MCP Apps, [4/07](../04-dynamic-ui/07-frontend-integration.md), [4/08](../04-dynamic-ui/08-security-sandboxing.md) | P3 "조직 경계", P4 "팀 구성"·"서드파티 툴 UI" |
| V13 | **팀 역량** | 데이터 엔지니어링 / 검색엔진·PostgreSQL 운영 / RDF·온톨로지 / MLOps / 네이티브·디자인 시스템 팀 / 사용 언어(Python, Java, .NET, Go, TS) | [2/04](../02-data-pipeline/04-embedding-indexing.md), [2/06](../02-data-pipeline/06-ontology-driven-modeling.md), [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md), [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md), [4/03](../04-dynamic-ui/03-component-registry-schema.md), [4/07](../04-dynamic-ui/07-frontend-integration.md) | P2, P3(언어), P4 |
| V14 | **예산·반출 제약·벤더 종속** | 인덱싱 예산 / API 반출 허용 여부 / 온프레미스 / 벤더 종속 허용·불허 | [2/01](../02-data-pipeline/01-source-ingestion.md), [2/02](../02-data-pipeline/02-chunking-strategies.md), [2/04](../02-data-pipeline/04-embedding-indexing.md), [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md), [3/11](../03-agent-workflow-rag/11-decision-layer.md), [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md), [5/02](../05-cross-cutting/02-cost-latency-model-routing.md) | P2 "예산·반출", P3 "벤더 종속" |
| V15 | **언어** | 한국어 단일 / 한·영 혼합 / 다국어(RTL 포함) | [2/01](../02-data-pipeline/01-source-ingestion.md) OCR, [2/02](../02-data-pipeline/02-chunking-strategies.md) 토큰 기준, [2/04](../02-data-pipeline/04-embedding-indexing.md) 형태소, [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md), [3/11](../03-agent-workflow-rag/11-decision-layer.md)·[3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) 결정 모델 `[실측]` 필수, [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) judge 정렬, [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md), [4/09](../04-dynamic-ui/09-ux-accessibility.md) i18n | P2 · P3 · P4 공통 |

상위 결정력: Part 3/99의 관찰대로 **V7 · V8 · V3 · V4 · V9 · V12** 여섯 개가 Reasoning 선택의 대부분을 정하고, Knowledge는 **V1 · V2 · V6**, Interaction은 **V10 · V11 · V8**이 정한다. 나머지는 조건 A/B를 여는 보조 변수다.

### 판정 지점(decision point)이라는 관점

질문 트리의 Q10이 묻는 "판정 지점"은 [Part 1/02](../01-foundations/02-system1-vs-system2.md)의 개념이다. 라우팅·게이팅·필터링·품질 판정·UI 선택·승인 필요 판정처럼 **선택지가 닫힌 분기**를 가리키며, 이 분기들을 누가 맡느냐(규칙 / 분류기 / 결정 모델 / LLM)가 Part 2/09 · 3/02 · 3/11 · 3/12 · 4/05 · 5/02 여섯 페이지의 선택을 한꺼번에 움직인다. 트리를 돌리기 전에 요청 1건의 경로에서 판정 지점을 전부 표시해 두면 Q10의 답이 쉬워진다.

## 아키텍처 / 파이프라인

### 질문 트리

각 질문은 "예"와 "아니오"가 모두 **특정 페이지의 특정 기본값**으로 떨어진다. 순서대로 답하되, Q1에서 "예"(읽기만)를 고르면 Q13·Q14는 건너뛴다. HTML 시각화의 질문 번호와 동일하다.

**범위 (어느 레이어가 필요한가)**

1. **Q1. 사용자가 결과를 읽기만 하는가?** (V10) → **예**: Knowledge + Reasoning만. Interaction은 citation 렌더([4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) 일부)로 한정, 단 계약 ②·③은 미리 정의([01 레퍼런스 아키텍처](01-reference-architecture.md)) / **아니오**: 3-Layer 전체. Interaction은 Controlled 패턴([4/01](../04-dynamic-ui/01-generative-ui-spectrum.md))부터, Q13~14로
2. **Q2. 행동(툴 실행)이 있는가?** (V9) → **예**: 결정 레이어([3/11](../03-agent-workflow-rag/11-decision-layer.md)) · 검증기([3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)) · 가드레일([3/12](../03-agent-workflow-rag/12-guardrails-hitl.md))을 경로 위에 생략 없이, 툴은 등급별 멱등·게이트([3/09](../03-agent-workflow-rag/09-tools-and-mcp.md)) / **아니오**: 입출력 가드레일 + 사후 샘플링([3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)), HITL·내구 실행 불필요

**Knowledge (Part 2)**

3. **Q3. 데이터가 정형(DB·API) 중심인가?** (V1) → **예**: 레코드를 근거 단위로([2/08](../02-data-pipeline/08-grounding-data-design.md)), 템플릿 직렬화로 청킹 최소([2/02](../02-data-pipeline/02-chunking-strategies.md)), 그래프는 규칙 추출([2/05](../02-data-pipeline/05-knowledge-graph-construction.md)), 정형 질의는 변환기 패턴([3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)) / **아니오**: 청크 근거 단위, 직접 커넥터 + Docling 류 레이아웃 파서([2/01](../02-data-pipeline/01-source-ingestion.md)), 문서 구조 기반 + parent-child 청킹([2/02](../02-data-pipeline/02-chunking-strategies.md)), 스캔 다수면 벤더 OCR 1차
4. **Q4. 신선도 SLO가 분 단위인가?** (V3, V2) → **예**: CDC + 단일 변경 큐, 톰스톤 즉시([2/07](../02-data-pipeline/07-freshness-and-sync.md), [2/01](../02-data-pipeline/01-source-ingestion.md)) / **아니오**: 증분 배치 + 톰스톤 + 주 1회 전체 대조 + 멱등 재처리([2/07](../02-data-pipeline/07-freshness-and-sync.md))
5. **Q5. 규제·감사 대상이거나 citation이 필수인가?** (V4, V5) → **예**: bitemporal 메타데이터([2/03](../02-data-pipeline/03-metadata-provenance.md)), 근거 문장 필수 + 상충 처리([2/08](../02-data-pipeline/08-grounding-data-design.md)), 감사 로그([2/10](../02-data-pipeline/10-access-control-governance.md)), 생성 후검증(근거 정합성 검사기)([3/04](../03-agent-workflow-rag/04-generation-grounding.md)) / **아니오**: 평면 메타데이터 + 필수 4필드(`source_uri`·`content_hash`·`ingested_at`·`locator`)([2/03](../02-data-pipeline/03-metadata-provenance.md)), 청크 + 최소 필드([2/08](../02-data-pipeline/08-grounding-data-design.md)), 구조화 citation + 존재 검증만([3/04](../03-agent-workflow-rag/04-generation-grounding.md))
6. **Q6 ★온톨로지. 질의 로그·골든셋에서 멀티홉·동일성·설명 요구가 `[실측]`으로 확인되었는가?** (V6) → **예**: 경량 온톨로지(클래스 ≤ 30) → SHACL → LPG([2/06](../02-data-pipeline/06-ontology-driven-modeling.md)), 하이브리드 추출 그래프 + 증분 인덱싱([2/05](../02-data-pipeline/05-knowledge-graph-construction.md)), 근거 단위에 트리플·경로 추가([2/08](../02-data-pipeline/08-grounding-data-design.md)), GraphRAG 계열을 **시험**([3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md), [Part 6](../06-trends/graphrag-evolution.md)) / **아니오**: 택소노미(SKOS)만([2/06](../02-data-pipeline/06-ontology-driven-modeling.md)), 그래프 미도입([2/05](../02-data-pipeline/05-knowledge-graph-construction.md)), Advanced RAG(hybrid + 리랭킹)([3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md)). 근거: [온톨로지 르네상스](../06-trends/ontology-renaissance.md)는 "온톨로지 없이 먼저 시험"이 채택 기준의 전제
7. **Q7. 문서별 ACL · 멀티테넌트 · PII 중 하나라도 있는가?** (V5) → **예**: 테넌트 분리 + 메타데이터 사전 필터 + 렌더 전 재확인 + 수집 직후 PII 마스킹 + 게이트웨이 단일 강제([2/10](../02-data-pipeline/10-access-control-governance.md)) / **아니오**: 전 직원 공개·단일 테넌트·PII 없음을 **확인한 뒤** 최소 구성([2/10](../02-data-pipeline/10-access-control-governance.md)) — 확인 못 했으면 "예"로 간다

**Reasoning / Action (Part 3)**

8. **Q8. 과업이 고정 절차 또는 몇 가지 분기로 예측 가능한가?** (V7) → **예**: 워크플로우 — 프롬프트 체이닝 또는 라우팅([3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md)), Tool-use + Router([3/06](../03-agent-workflow-rag/06-agent-design-patterns.md)), 구조화 상태 + 체크포인팅([3/08](../03-agent-workflow-rag/08-state-memory-context.md)) / **아니오**: 자율 에이전트(비용·단계 상한 필수)([3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md)), Plan-and-Execute + Reflection(횟수 상한)([3/06](../03-agent-workflow-rag/06-agent-design-patterns.md)), 내구 실행 런타임([3/08](../03-agent-workflow-rag/08-state-memory-context.md)); 조치 부분은 워크플로우로 고정
9. **Q9. 툴·에이전트의 당사자가 둘 이상인가(여러 팀 · 타 조직 · 외부 소비자)?** (V12) → **예**: 자체 MCP 서버(2026-07-28 개정 지원)([3/09](../03-agent-workflow-rag/09-tools-and-mcp.md)), Supervisor + 공유 상태, 타 조직은 A2A([3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md)), 툴 20개 초과면 2단계 라우팅 / **아니오**: 네이티브 함수 + 등급별 멱등·게이트([3/09](../03-agent-workflow-rag/09-tools-and-mcp.md)), 단일 에이전트([3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md)). 근거: [프로토콜 수렴](../06-trends/agentic-protocols-convergence.md) "당사자 둘 이상" 기준
10. **Q10 ★결정 모델. 판정 지점(라우팅·게이팅·승인 필요·UI 선택)의 선택지가 닫혀 있고, 한국어 평가셋 `[실측]`이 있는가?** (V8, V15, V2) → **예**: 결정 모델(Jev / Decisions API)을 `decide()` 어댑터 뒤에 배치하고 보정 곡선 확인 후 θ 설정([3/11](../03-agent-workflow-rag/11-decision-layer.md)), 확신도 세 갈래 HITL([3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)), 결정 모델 라우터([3/02](../03-agent-workflow-rag/02-query-understanding.md)), 파이프라인 판정은 소형 분류기와 비교([2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md)) / **아니오**: 규칙 + 소형 LLM, 단 `decide()` 인터페이스·트레이스·평가셋을 **먼저** 만든다([3/11](../03-agent-workflow-rag/11-decision-layer.md)); 라우팅은 규칙 + 소형 LLM으로 로그 축적([3/02](../03-agent-workflow-rag/02-query-understanding.md)); 규칙 → 소형 LLM → 분류기 증류([2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md)). 근거: [결정 모델](../06-trends/decision-models.md)은 전 수치 `[벤더]`, 영어 중심 학습 — **`[실측]` 없이 채택하지 않는다**
11. **Q11 ★뉴로심볼릭. 쓰기 이상 툴이 있거나, LLM 추출 결과가 인덱스·그래프에 적재되는가?** (V9, V1) → **예**: JSON Schema + 규칙 게이트 + 재생성 상한 2회([3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md)), 추출 → SHACL/등가 제약 검증([2/05](../02-data-pipeline/05-knowledge-graph-construction.md), [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md)), 정책 엔진(allow/deny/require_approval) + 행동 직전 규칙 최종 허가([3/12](../03-agent-workflow-rag/12-guardrails-hitl.md)) / **아니오**: 구조화 출력 + citation 존재 규칙 검증([3/04](../03-agent-workflow-rag/04-generation-grounding.md))으로 충분, 3/10은 "해당 없음". 근거: [뉴로심볼릭](../06-trends/neuro-symbolic.md) — 검증기 패턴은 운영 표준이나 "결합 전체의 V&V는 미성숙" `[검증]`
12. **Q12. 사용자가 반복 방문하고 멀티세션 요구가 사용 사례로 확인되었는가?** (V7, V10) → **예**: 항목 제한 자체 메모리, **no-memory 대비 `[실측]`** 후 채택, 조회 게이팅([3/08](../03-agent-workflow-rag/08-state-memory-context.md)) / **아니오**: 구조화 상태 + 체크포인팅 + no-memory([3/08](../03-agent-workflow-rag/08-state-memory-context.md)). 근거: [에이전트 메모리](../06-trends/agent-memory.md) MemTrapBench `[2차]`

**Interaction (Part 4) — Q1이 "아니오"일 때만**

13. **Q13. 사용자의 행동이 비가역·고액·규제 대상인가?** (V8, V5) → **예**: Controlled 고정(자유도 0)([4/01](../04-dynamic-ui/01-generative-ui-spectrum.md)), 승인 카드(영향 범위·근거·`as_shown_hash`), 타임아웃 = 실행 안 함 + 에스컬레이션([4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)), 매핑은 규칙만([4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md)), 고위험 UI A/B 금지([4/10](../04-dynamic-ui/10-evaluation-genui.md)) / **아니오**: Controlled 기본 + 선택지 제시·인라인 수정([4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)), 규칙 1차 필터 + LLM 툴 호출 매핑(트레이스 기록)([4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md)), 조합 폭발 의도(임의 열 비교표)만 Declarative 검토([4/01](../04-dynamic-ui/01-generative-ui-spectrum.md))
14. **Q14. 프론트·백이 분리되었거나 백엔드가 여럿인가, 또는 모바일 네이티브·서드파티 툴 UI가 필요한가?** (V11, V12) → **예**: 전송층 AG-UI([4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)), 네이티브 Declarative는 A2UI 렌더러 `[실측]` 후([4/07](../04-dynamic-ui/07-frontend-integration.md)), 서드파티 UI는 MCP Apps iframe "표시만"부터 + 샌드박스([4/08](../04-dynamic-ui/08-security-sandboxing.md)) / **아니오**: 프레임워크 내장 스트리밍, "툴 호출 = 컴포넌트"로 설계([4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md)), React 클라이언트 레지스트리 + 이벤트 어댑터([4/07](../04-dynamic-ui/07-frontend-integration.md)), JSON Schema 레지스트리 원본([4/03](../04-dynamic-ui/03-component-registry-schema.md)), SSE + 혼합 동기화([4/04](../04-dynamic-ui/04-streaming-state-sync.md))

**횡단 (Part 5 · 프레임워크)**

15. **Q15. 단일 벤더 종속을 허용하는가?** (V14, V13) → **예**: 해당 벤더 SDK(Claude Agent SDK / OpenAI Agents SDK / Google ADK — Java·Go·TS 팀은 ADK, .NET은 Semantic Kernel)([3/14](../03-agent-workflow-rag/14-frameworks-landscape.md)) / **아니오**: LangGraph([3/14](../03-agent-workflow-rag/14-frameworks-landscape.md)). **공통**: 프레임워크 밖 모듈(상태 스키마·툴 카탈로그·`decide()`·정책·평가셋)을 먼저, 자체 트레이스 스키마 + OTel 매핑 계층([5/03](../05-cross-cutting/03-observability-tracing.md)), 모델 라우팅·캐시 정책([5/02](../05-cross-cutting/02-cost-latency-model-routing.md)), L1 규칙 CI + L2 골든셋 + 층화 샘플링([3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md))

### 단계별 기본값 — 판단이 어려울 때

각 99 매트릭스의 "기본값" 열을 한 줄씩 옮겼다. 단계 페이지의 "선택 가이드 → 판단이 어려우면"과 어긋나는 곳은 **99를 따랐고** 표 아래에 적었다.

| 단계 | 판단이 어려울 때의 기본값 |
|---|---|
| [2/01 수집·파싱](../02-data-pipeline/01-source-ingestion.md) | 직접 구현 커넥터 + Docling 류 레이아웃 파서 + 실패분 격리 큐. 샘플 100건으로 파서 2종 `[실측]` 후 교체 |
| [2/02 청킹](../02-data-pipeline/02-chunking-strategies.md) | 문서 구조 기반 + 재귀 보정 + parent-child, 맥락 보강 없음. 출처 혼동이 보이면 contextual 접두 추가 |
| [2/03 메타데이터](../02-data-pipeline/03-metadata-provenance.md) | 다섯 축 필수 필드 + 별도 저장소 + 불변 버전(`is_current`) + PROV 어휘 |
| [2/04 임베딩·인덱싱](../02-data-pipeline/04-embedding-indexing.md) | hybrid(BM25 + dense + RRF), 기존 인프라에 가장 가까운 저장소, 양자화 없음, 임베딩 모델 2종 `[실측]` 후 확정 |
| [2/05 KG 구축](../02-data-pipeline/05-knowledge-graph-construction.md) | **질의 로그에서 멀티홉·동일성 요구 확인 전엔 도입 안 함.** 도입 시 경량 온톨로지 + 하이브리드 추출 + SHACL + LPG + 증분 지원 인덱싱 |
| [2/06 온톨로지](../02-data-pipeline/06-ontology-driven-modeling.md) | 범위 질문 30개 → 경량 온톨로지(클래스 ≤ 30) → SHACL → LPG 변환 + 매핑 테이블 → 소유자 지정. 그래프 없으면 택소노미(SKOS)만 |
| [2/07 신선도·동기화](../02-data-pipeline/07-freshness-and-sync.md) | 증분 배치 + 톰스톤 + 주 1회 전체 대조 + 문서 단위 멱등 재처리 + 블루/그린 절차 |
| [2/08 근거 데이터 ★](../02-data-pipeline/08-grounding-data-design.md) | 하이브리드(청크 검색 + 문장 검증·렌더) + 코드 규칙 신뢰도 + 메타데이터 결정론 상충 탐지. 트리플은 그래프 도입 시 |
| [2/09 파이프라인 AI](../02-data-pipeline/09-ai-inside-the-pipeline.md) | 규칙으로 거른 뒤 소형 LLM 시작 → 라벨 축적 → 소형 분류기 증류. 결정 모델은 비교군 |
| [2/10 접근 제어](../02-data-pipeline/10-access-control-governance.md) | 테넌트 분리 + 메타데이터 사전 필터 + 렌더 전 재확인 + 수집 직후 PII 탐지·마스킹 + 계보 기반 삭제 + 게이트웨이 단일 강제 |
| [3/01 RAG 패턴](../03-agent-workflow-rag/01-rag-architecture-patterns.md) | Advanced RAG(하이브리드 검색 + 리랭킹). Naive로 기준선 먼저 |
| [3/02 질의 이해·라우팅](../03-agent-workflow-rag/02-query-understanding.md) | 결정 모델 또는 소형 분류기 라우터 + 필요 시 쿼리 재작성 ¹ |
| [3/03 검색·리랭킹](../03-agent-workflow-rag/03-retrieval-reranking.md) | BM25 + dense + RRF, cross-encoder 리랭커(+ 점수 컷오프) |
| [3/04 생성·근거 강제](../03-agent-workflow-rag/04-generation-grounding.md) | 구조화 출력으로 citation 수신 + citation 존재 규칙 검증(+ abstain). NLI는 faithfulness 측정 후 |
| [3/05 워크플로우 vs 에이전트](../03-agent-workflow-rag/05-workflow-vs-agent.md) | 가장 단순한 형태 — 단일 프롬프트 → 체이닝 → 라우팅 순. 측정된 실패가 다음 단계를 정당화할 때만 |
| [3/06 에이전트 패턴](../03-agent-workflow-rag/06-agent-design-patterns.md) | Tool-use + Router ² |
| [3/07 멀티에이전트](../03-agent-workflow-rag/07-multi-agent-orchestration.md) | 단일 에이전트(멀티 불필요). 도입 시 Supervisor + 최소 공유 상태 스키마 ³ |
| [3/08 상태·메모리·컨텍스트](../03-agent-workflow-rag/08-state-memory-context.md) | 구조화 상태 + 체크포인팅 + no-memory |
| [3/09 툴·MCP](../03-agent-workflow-rag/09-tools-and-mcp.md) | 네이티브 함수 + 등급별 멱등·게이트 정책 + 결과 신뢰 경계 |
| [3/10 뉴로심볼릭](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) | JSON Schema + 규칙 게이트 + 재생성 상한 2회 + 실패 로그 |
| [3/11 결정 레이어 ★](../03-agent-workflow-rag/11-decision-layer.md) | 규칙 + 소형 LLM, 단 `decide()` 인터페이스·트레이스·평가셋을 먼저 |
| [3/12 가드레일·HITL](../03-agent-workflow-rag/12-guardrails-hitl.md) | 규칙 1층 + 정책 엔진(allow/deny/require_approval) + 동기 HITL |
| [3/13 평가](../03-agent-workflow-rag/13-evaluation-rag-agents.md) | L1 규칙 CI + L2 골든셋(라우팅·검색·툴) + 운영 층화 샘플링 judge |
| [3/14 프레임워크](../03-agent-workflow-rag/14-frameworks-landscape.md) | 프레임워크 밖 모듈 먼저 + LangGraph vs 벤더 SDK 하나로 핵심 경로 PoC `[실측]` |
| [4/01 통제 수준](../04-dynamic-ui/01-generative-ui-spectrum.md) | Controlled |
| [4/02 프로토콜](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) | 전송층 프레임워크 내장 스트리밍 · 내용층 자체 레지스트리(Controlled) · 서드파티 UI 수용 안 함 |
| [4/03 레지스트리](../04-dynamic-ui/03-component-registry-schema.md) | JSON Schema 원본 + 생성, 에이전트 노출은 툴 정의(컴포넌트 = 툴) |
| [4/04 스트리밍·동기화](../04-dynamic-ui/04-streaming-state-sync.md) | SSE + HTTP POST 역방향, 스냅샷 + JSON Patch 혼합 |
| [4/05 의도→UI 매핑](../04-dynamic-ui/05-intent-to-ui-mapping.md) | 규칙 1차 필터 + LLM 툴 호출, 판정 결과를 트레이스에 기록 |
| [4/06 HITL UI](../04-dynamic-ui/06-hitl-ui-patterns.md) | citation 렌더 + 피드백 캡처부터 |
| [4/07 프론트엔드 통합](../04-dynamic-ui/07-frontend-integration.md) | React 클라이언트 레지스트리 + 이벤트 어댑터(SDK는 어댑터 뒤) · 모바일은 자체 Controlled 네이티브 렌더러 · 임베딩은 위젯 |
| [4/08 보안](../04-dynamic-ui/08-security-sandboxing.md) | props 스키마 + 서버 재검증 ①~⑤ + 출처 태그. Open-ended는 열지 않는다 |
| [4/09 UX·접근성](../04-dynamic-ui/09-ux-accessibility.md) | 텍스트 우선 + 밀도 상한 + Controlled 접근성 내장 + 라이브 영역 |
| [4/10 평가](../04-dynamic-ui/10-evaluation-genui.md) | 매핑 행당 골든셋 10건 + CI 스키마·카탈로그 대조 + 수정률·무시율·폴백률 |

¹ [3/02](../03-agent-workflow-rag/02-query-understanding.md) 페이지 가이드는 "규칙 + 소형 LLM 라우터로 시작 → 로그 축적 후 분류기·결정 모델로 이전"이다. 99의 기본값을 따르되, Q10이 "아니오"(한국어 `[실측]` 없음)면 페이지 가이드의 출발점이 실질적 기본값이 된다.
² [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md) 페이지 가이드는 "Router → Plan-and-Execute, 각 단계 Tool-use". 99는 Plan-and-Execute를 "다단계 계획 필요" 조건 A로 둔다. 99를 따른다.
³ [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) 페이지 가이드는 멀티에이전트 도입을 전제로 "Supervisor + 최소 공유 상태". 99는 도입 자체를 기본값으로 두지 않는다. 99를 따른다. [2/05](../02-data-pipeline/05-knowledge-graph-construction.md)도 같은 구조(페이지는 도입 전제, 99는 미도입이 기본값)다.

### 결정 기록 템플릿

트리를 돌린 결과를 프로젝트 저장소에 남긴다. HTML 시각화 하단의 "현재 경로" 패널이 이 표의 초안을 만들어 준다.

| 변수 값 (V#) | 질문 (Q#) · 답 | 단계 | 선택 | 대안 (채택 안 한 이유) | 이유 | 근거 페이지 |
|---|---|---|---|---|---|---|
| V10 = 읽기만 | Q1 · 예 | 범위 | Knowledge + Reasoning, citation 렌더만 | 3-Layer 전체 (조작 과업 없음) | 사용자 행위가 읽기라 Interaction 대부분 불필요 | [01](01-reference-architecture.md), [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) |
| V6 = 멀티홉 비율 미측정 | Q6 · 아니오 | 2/05 · 2/06 · 3/01 | 그래프 미도입, 택소노미만, Advanced RAG | 경량 온톨로지 + 그래프 (요구 `[실측]` 없음) | 벡터 + 택소노미로 골든셋 통과 여부를 먼저 본다 | [2/05](../02-data-pipeline/05-knowledge-graph-construction.md), [Part 6](../06-trends/ontology-renaissance.md) |
| V15 = 한·영 혼합, 평가셋 없음 | Q10 · 아니오 | 3/11 · 3/02 | 규칙 + 소형 LLM, `decide()` 먼저 | 결정 모델 (한국어 `[실측]` 없음, 수치 `[벤더]`) | 인터페이스·트레이스가 있으면 평가셋 확보 후 교체 비용이 작다 | [3/11](../03-agent-workflow-rag/11-decision-layer.md), [Part 6](../06-trends/decision-models.md) |
| … | … | … | … | … | … | … |

## 기술 비교표 — 결정 수단 자체의 비교

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| **질문 트리** (이 페이지) | 착수 시 전체 범위를 한 번에 훑어야 할 때. 변수가 일반적 범위 안에 있을 때 | 변수 조합이 특이할 때(예: 정형 데이터인데 열린 탐색 + 규제). 한 단계만 바꿀 때(과잉) | 하 — 15문항, 1~2시간 | 표준(의사결정 기법) | — 위키 내부 구조 |
| **결정 매트릭스** (각 Part 99) | 한 레이어 안에서 조건 A/B를 세밀하게 고를 때, "피해야 할 조합" 점검 | 레이어 간 정합성 확인(각 Part 매트릭스는 다른 Part의 선택을 모른다) | 중 — 10~14행 × 4열을 읽어야 함 | 표준 | — |
| **체크리스트** (각 페이지 "설계 시 고려사항") | 선택이 끝난 뒤 빠진 것 점검, 설계 리뷰 | 선택 자체를 정하는 용도(체크리스트는 "무엇을"은 주지만 "어느 것을"은 안 준다) | 하 | 표준 | — |
| **시나리오 유추** ([Part 7](../07-appendix-scenarios/00-overview.md)) | 변수 조합이 세 시나리오 중 하나와 가까울 때. 조합 전체를 한 번에 보고 싶을 때 | 세 시나리오와 모두 멀 때(유추가 틀린 확신을 준다) | 하 — 읽기만 | 표준 | — |
| 가중 점수 매트릭스(기술 × 기준 점수화) | 최종 후보 2~3개 사이의 공식 선정, 조달·심사 대응 | 후보가 많거나 기준 가중치에 합의가 없을 때(점수가 결론을 정당화하는 도구가 됨) | 중 — 가중치 합의 비용 | 표준 | `[2차]` 일반 의사결정 문헌 |

### 선택 가이드

- **착수 첫 주라면** → 질문 트리 → 결정 기록 → 가장 가까운 시나리오 비교. 이유: 레이어 간 정합성을 가장 싸게 확보한다
- **한 레이어만 재설계한다면** → 해당 Part 99 매트릭스 + 단계 페이지 선택 가이드. 트리는 Q1·Q2만 다시 확인
- **변수 조합이 특이하다면** → 시나리오 유추 + 각 단계 페이지의 "조건 A/B"를 직접 조합. 트리의 기본값은 참고만
- **판단이 어려우면** → 트리의 기본값을 전부 받아들이고 결정 기록에 "기본값"이라고 적는다. 이유: 모든 기본값은 "가장 되돌리기 쉬운 선택"으로 설계되어 있어, 틀려도 전환 비용이 가장 작다

## 설계 시 고려사항 (체크리스트)

- [ ] 입력 변수 15개 중 "미정"이 3개 이하인가 (3개 초과면 요구 분석이 먼저다)
- [ ] V4(근거 요구)·V8(실패 비용)·V10(사용자 행위)의 답이 Knowledge · Reasoning · Interaction 세 레이어에서 **같은 값**으로 쓰였는가
- [ ] Q6 · Q10 · Q11(★ 세 축)의 "예"에 `[실측]` 근거(골든셋·평가셋·질의 로그 분석)가 첨부되어 있는가. 없으면 "아니오"로 되돌린다
- [ ] 기본값에서 벗어난 선택마다 99 매트릭스의 조건 A/B 또는 시나리오 페이지가 근거로 적혀 있는가
- [ ] 99 매트릭스의 "피해야 할 조합"에 걸리는 선택이 없는가
- [ ] 결정 기록이 저장소에 있고, 계약 ①②③([01](01-reference-architecture.md))의 스키마 초안과 함께 리뷰되었는가
- [ ] 재평가 시점(변수 값이 바뀔 사건: 질의 로그 N건 축적, 한국어 평가셋 완성, 규제 변경)이 적혀 있는가

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| 트리의 리프는 **일반 기본값**일 뿐이다 | 변수 조합이 특이하면 기본값이 서로 모순될 수 있다(예: 열린 탐색 + 비가역 행동 + 실시간 지연) | [Part 7 시나리오](../07-appendix-scenarios/00-overview.md) 유추와 각 99의 조건 A/B 직접 조합. 트리는 출발점으로만 |
| 예/아니오 이분법이 연속 변수(규모·지연·실패 비용)를 자른다 | 경계값 근처 프로젝트가 잘못된 가지로 간다 | 경계에 있으면 **보수적 쪽**(규제 있음, 행동 있음, 요구 미확인)으로 답하고 결정 기록에 표시 |
| 기본값은 "되돌리기 쉬운 선택"이지 "최고 성능"이 아니다 | 기본값만으로 운영하면 측정된 실패 뒤에도 전환을 미루게 됨 | 재평가 시점을 결정 기록에 명시, 실패 로그가 다음 단계를 정당화하면 조건 A/B로 이동 |
| 99 매트릭스와 단계 페이지 가이드가 4곳에서 어긋난다 | 두 문서를 읽는 사람이 다른 기본값을 가진다 | 이 페이지는 99를 따른다(각주 ¹²³ 및 2/05). 두 문서 정합은 다음 리뷰에서 |
| 트렌드 세 축의 분기 조건이 `[실측]`을 요구하는데, 착수 시점엔 `[실측]`이 없다 | Q6 · Q10 · Q11이 거의 항상 "아니오"로 떨어진다 | 그것이 의도다. "아니오" 리프마다 **나중에 전환하기 위한 준비물**(`decide()` 인터페이스, 택소노미, 추출 검증 로그)을 포함시켰다 |
| 트리는 Part 5 횡단 관심사를 Q15 하나로 압축한다 | 보안·비용·관측·거버넌스의 세부 선택이 빠진다 | 트리를 돌린 뒤 [Part 5](../05-cross-cutting/00-overview.md) 체크리스트를 별도로 수행 |

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 온톨로지 | Q6의 분기 조건. "예"면 2/05·2/06·2/08·3/01이 한꺼번에 움직이고, "아니오"면 택소노미에서 멈춘다. 트리가 온톨로지를 **요구 확인 뒤의 선택**으로 두는 것 자체가 [르네상스](../06-trends/ontology-renaissance.md) 담론(벤더 프레이밍 다수)에 대한 이 위키의 입장 | 멀티홉·동일성·설명 요구 `[실측]` | `[검증-단일저자]` `[벤더]` `[2차]` |
| 뉴로심볼릭 | Q11의 분기 조건. 쓰기 툴·추출 적재가 있으면 3/10의 기본값이 "프롬프트 규칙"에서 "Schema + 규칙 게이트"로 바뀐다(Part 3/99 3-1절). 트리 자체도 뉴로심볼릭 구조다 — 변수(기호) → 규칙(분기) → LLM이 아닌 사람의 판단 | 쓰기 이상 툴 또는 LLM 추출 → 적재 | `[2차]` `[검증]` V&V 한계 |
| 결정 모델 (Jev / Decisions API) | Q10의 분기 조건. "예"면 3/02·3/11·3/12·4/05·2/09·5/02의 판정 수단이 결정 모델로 바뀌고 HITL이 이분법에서 확신도 세 갈래로 바뀐다. 2026-09 등장이라 거의 모든 프로젝트가 "아니오" — 단, `decide()` 추상화는 "아니오"에서도 만든다 | 선택지 닫힘 + 한국어 `[실측]` | `[벤더]` `[2차]` 소스 충돌 |
| 에이전트 프로토콜 수렴 | Q9 · Q14의 분기 조건 "당사자 둘 이상". MCP(2026-07-28) · A2A(LF 이관) · AG-UI · A2UI · MCP Apps 중 **필요한 경계에만** 하나씩 | 경계의 당사자 둘 이상 | `[2차]` |
| 에이전트 메모리(MemTrapBench) | Q12의 기본값을 "메모리 프레임워크 도입"에서 "no-memory"로 바꾼 근거 | 모든 메모리 검토 | `[2차]` |

## 관련 페이지

- [Part 0 개요](00-overview.md) · [01 레퍼런스 아키텍처](01-reference-architecture.md)(범위와 계약 ①②③) · [03 트렌드 레이더](03-trend-radar-2026.md)(Q6 · Q10 · Q11의 현재 위치)
- 결정 매트릭스: [Part 2/99](../02-data-pipeline/99-decision-matrix.md) · [Part 3/99](../03-agent-workflow-rag/99-decision-matrix.md) · [Part 4/99](../04-dynamic-ui/99-decision-matrix.md) — 각 질문의 조건 A/B와 "피해야 할 조합"
- [Part 1/02 System 1 vs System 2](../01-foundations/02-system1-vs-system2.md) — 판정 지점 목록화 절차
- [Part 5 개요](../05-cross-cutting/00-overview.md) — 트리 이후의 횡단 검토
- [Part 7 시나리오 A](../07-appendix-scenarios/01-enterprise-document-qa.md) · [B](../07-appendix-scenarios/02-commerce-assistant-genui.md) · [C](../07-appendix-scenarios/03-ops-incident-agent.md) — 트리를 돌린 결과의 예

## 참고 문헌

- 이 페이지는 위키 내부 세 결정 매트릭스의 통합이므로 외부 문헌 인용 없음. 질문의 분기 조건에 쓰인 트렌드 사실과 검증 수준은 [`_meta/research-notes-2026-10.md`](../_meta/research-notes-2026-10.md)와 [Part 6](../06-trends/00-overview.md) 각 페이지의 "채택 기준"을 따른다
- `[검증]` Kahneman, *Thinking, Fast and Slow* (2011) — Q10 "판정 지점" 개념의 원전([Part 1/02](../01-foundations/02-system1-vs-system2.md) 경유)

_검증 수준 표기: `[검증]` `[벤더]` `[2차]` `[실측]` — 규칙은 [작성 가이드](../_meta/writing-guide.md) 참조_
