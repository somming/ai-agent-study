# AI Agent 설계 Wiki — 구조 설계안 (v0.1, 2026-10-11)

> 이 문서는 위키 본문이 아니라 **위키를 어떻게 구성할지에 대한 설계안**이다.
> 확정되면 `README.md`(진입점)와 각 디렉터리의 `00-overview.md`로 분해된다.

---

## 1. 위키의 목적과 독자

| 항목 | 내용 |
|---|---|
| 목적 | AI Agent 시스템을 **설계**할 때 필요한 기술·지식을 계층적으로 정리하고, 각 단계에서 "무엇을, 언제, 왜, 어떤 한계를 감수하고" 쓰는지 판단할 수 있게 한다 |
| 핵심 산출물 | 계층화된 Markdown 페이지 + 아키텍처/파이프라인 HTML 시각화 + 기술 선택 결정 매트릭스 |
| 독자 | 아키텍트, AI 엔지니어, 프론트엔드/데이터 엔지니어, 기술 의사결정자 |
| 비(非)목표 | 특정 프레임워크 튜토리얼, 코드 레벨 API 레퍼런스 (필요 시 외부 링크) |

## 2. 세 가지 주제 축과 그 관계

```
[주제 1] 데이터 파이프라인 & 근거 데이터   →  무엇을 근거로 판단하는가 (Knowledge Layer)
[주제 2] Agent 워크플로우 & LLM/RAG 아키텍처 →  어떻게 추론·행동하는가 (Reasoning/Action Layer)
[주제 3] 동적 UI 연동                       →  결과를 어떻게 사람과 주고받는가 (Interaction Layer)
```

세 축을 관통하는 **공통 기반(Foundations)** 과 **횡단 관심사(Cross-cutting)** 를 별도 섹션으로 둔다.
최신 트렌드(온톨로지, 뉴로심볼릭, System One 모델 등)는 **① 전용 심화 페이지** + **② 각 주제 페이지 안의 "트렌드 적용" 섹션** 두 곳에서 다룬다. 전용 페이지는 "그 기술이 무엇인가", 주제 페이지는 "이 단계에서 어떻게 쓰는가"를 맡는다.

## 3. 디렉터리 계층 (3단계: Part → Chapter → Page)

```
ai-agent-wiki/
├── README.md                              # 진입점: 목적, 읽는 순서, 범례, 전체 맵(HTML 링크)
│
├── 00-overview/                           # Part 0. 전체 조망
│   ├── 01-reference-architecture.md       # 3-Layer 레퍼런스 아키텍처  [HTML]
│   ├── 02-decision-guide.md               # "언제 무엇을 쓰나" 종합 결정 트리  [HTML]
│   ├── 03-trend-radar-2026.md             # 트렌드 레이더 (채택/시험/관찰/보류)  [HTML]
│   └── 04-glossary.md                     # 용어집 (한/영 병기)
│
├── 01-foundations/                        # Part 1. 공통 기반 지식
│   ├── 00-overview.md
│   ├── 01-llm-fundamentals.md             # 컨텍스트 윈도우, 구조화 출력, 함수 호출, 모델 라우팅
│   ├── 02-system1-vs-system2.md           # Kahneman 프레임 → LLM(System 2) vs 결정 모델(System 1) 역할 분담
│   ├── 03-knowledge-representation.md     # 택소노미 → 온톨로지 → 지식그래프, RDF/OWL/SHACL vs Property Graph
│   ├── 04-neuro-symbolic-ai.md            # 정의, Kautz 분류(Symbolic[Neural] 등), 6가지 결합 패턴, 한계
│   ├── 05-agentic-protocols.md            # MCP / A2A / AG-UI / A2UI / MCP Apps 관계도  [HTML]
│   └── 06-evaluation-observability.md     # 평가 체계, 트레이싱(OTel GenAI 컨벤션), 골든셋
│
├── 02-data-pipeline/                      # Part 2. AI 기반 데이터 파이프라인 및 근거 데이터 설계
│   ├── 00-overview.md                     # 파이프라인 전체도  [HTML: data-pipeline-architecture]
│   ├── 01-source-ingestion.md             # 커넥터, CDC, 문서 파싱(PDF/표/이미지), 멀티모달 추출
│   ├── 02-chunking-strategies.md          # fixed / recursive / semantic / late chunking / contextual / parent-child
│   ├── 03-metadata-provenance.md          # 출처·버전·권한·lineage, W3C PROV, 근거 추적성
│   ├── 04-embedding-indexing.md           # dense / sparse / hybrid / multi-vector, 벡터DB 선택, 양자화
│   ├── 05-knowledge-graph-construction.md # LLM 기반 엔티티·관계 추출, 엔티티 해소, GraphRAG/LightRAG 인덱싱  [HTML]
│   ├── 06-ontology-driven-modeling.md     # 비즈니스 온톨로지 설계, 시맨틱 레이어, 스키마 정렬
│   ├── 07-freshness-and-sync.md           # 증분 인덱싱, 삭제/만료 전파, 중복 제거, 재인덱싱 전략
│   ├── 08-grounding-data-design.md        # ★ 근거 데이터: evidence 단위, citation 스키마, 신뢰도 등급, 반증 데이터
│   ├── 09-ai-inside-the-pipeline.md       # 파이프라인 내부의 AI: 분류/정제/라벨링/추출, System One 모델로 저비용 판정
│   ├── 10-access-control-governance.md    # ACL-aware retrieval, PII 마스킹, 멀티테넌시
│   └── 99-decision-matrix.md              # 데이터 유형 × 전략 매트릭스, 한계 정리
│
├── 03-agent-workflow-rag/                 # Part 3. Agent 워크플로우 설계 및 LLM/RAG 파이프라인 아키텍처
│   ├── 00-overview.md                     # 전체도  [HTML: agent-workflow-architecture]
│   ├── 01-rag-architecture-patterns.md    # Naive → Advanced → Modular → Agentic RAG, GraphRAG, Hybrid  [HTML]
│   ├── 02-query-understanding.md          # rewrite, decomposition, HyDE, intent 분류·라우팅 (System One 적용점)
│   ├── 03-retrieval-reranking.md          # hybrid, RRF, cross-encoder, LLM rerank, 컨텍스트 압축
│   ├── 04-generation-grounding.md         # citation 강제, 구조화 출력, 근거 검증, 환각 가드
│   ├── 05-workflow-vs-agent.md            # 결정론적 워크플로우 vs 자율 에이전트 스펙트럼, 선택 기준
│   ├── 06-agent-design-patterns.md        # ReAct, Plan-and-Execute, Reflection, Router, Orchestrator-Worker  [HTML]
│   ├── 07-multi-agent-orchestration.md    # Supervisor / Hierarchical / Handoff, A2A, 상태 공유  [HTML]
│   ├── 08-state-memory-context.md         # 단기/장기 메모리, context engineering, compaction
│   ├── 09-tools-and-mcp.md                # 툴 스키마 설계, MCP 서버, 권한, 멱등성, 실패 처리
│   ├── 10-neuro-symbolic-reasoning.md     # LLM + 규칙엔진/온톨로지 reasoner/제약 검증기 결합 패턴
│   ├── 11-decision-layer-system-one.md    # ★ Jev 등 System One 모델: 게이팅·라우팅·판정에 배치, LLM과 결합, 한계
│   ├── 12-guardrails-hitl.md              # 입출력 가드레일, 정책 엔진, 승인 흐름
│   ├── 13-evaluation-rag-agents.md        # RAG 지표, LLM-as-judge, 트래젝토리 평가
│   ├── 14-frameworks-landscape.md         # LangGraph, Claude Agent SDK, OpenAI Agents SDK, Semantic Kernel 등 비교
│   └── 99-decision-matrix.md
│
├── 04-dynamic-ui/                         # Part 4. AI Agent 기반 동적 UI 연동 설계
│   ├── 00-overview.md                     # 전체도  [HTML: dynamic-ui-architecture]
│   ├── 01-generative-ui-spectrum.md       # Controlled / Declarative / Open-ended 3패턴, 통제 vs 자유도
│   ├── 02-protocols-ag-ui-a2ui-mcp-apps.md# 프로토콜 상세 비교  [HTML]
│   ├── 03-component-registry-schema.md    # 컴포넌트 카탈로그, JSON Schema, 타입 안전성, 디자인 시스템 연결
│   ├── 04-streaming-state-sync.md         # SSE, state patch, optimistic UI, 양방향 이벤트  [HTML]
│   ├── 05-intent-to-ui-mapping.md         # 어떤 UI를 띄울지 결정: System One 판정 + 온톨로지 기반 매핑
│   ├── 06-hitl-ui-patterns.md             # 승인/수정/피드백 루프 UI
│   ├── 07-frontend-integration.md         # React/Next.js, Vercel AI SDK, CopilotKit, 모바일(Flutter/네이티브)
│   ├── 08-security-sandboxing.md          # iframe 샌드박스, UI 경유 prompt injection, 권한 경계
│   ├── 09-ux-accessibility.md
│   ├── 10-evaluation-genui.md
│   └── 99-decision-matrix.md
│
├── 05-cross-cutting/                      # Part 5. 횡단 관심사
│   ├── 01-security-prompt-injection.md
│   ├── 02-cost-latency-model-routing.md   # 모델 라우팅, 캐싱, System One으로 LLM 호출 절감
│   ├── 03-observability-tracing.md
│   ├── 04-governance-explainability.md    # EU AI Act, 감사 추적, 설명가능성(온톨로지의 역할)
│   └── 05-llmops-agentops.md
│
├── 06-trends/                             # Part 6. 최신 트렌드 심화 (레이더에서 링크)
│   ├── ontology-renaissance.md            # 왜 온톨로지가 다시 핵심이 되었나
│   ├── neuro-symbolic.md
│   ├── system-one-models-jev.md           # ★ Jev: 비자기회귀, 타입드 질문→구조화 답, 보정 확률, 한계, 검증 수준
│   ├── graphrag-evolution.md
│   ├── context-engineering.md
│   ├── agentic-protocols-convergence.md
│   └── generative-ui.md
│
├── 07-case-studies/                       # Part 7. End-to-End 레퍼런스 시나리오
│   ├── 01-enterprise-document-qa.md       # 사내 문서 QA (주제1+2 중심)  [HTML]
│   ├── 02-commerce-assistant-genui.md     # 커머스 상담 + 동적 UI (주제1+2+3)  [HTML]
│   └── 03-ops-incident-agent.md           # 운영 인시던트 대응 에이전트 (System One 게이팅 사례)  [HTML]
│
├── assets/
│   ├── html/                              # 시각화 (self-contained, 공통 CSS/JS)
│   │   ├── _shared/                       # 공통 스타일, 범례, 인터랙션 스크립트
│   │   └── *.html
│   └── img/                               # 썸네일(md 미리보기용)
│
├── _templates/
│   ├── page.md                            # 기술 페이지 템플릿
│   ├── decision-matrix.md
│   └── trend.md
│
└── _meta/
    ├── writing-guide.md                   # 작성 규칙, 태그 체계, 검증 수준 표기법
    └── CHANGELOG.md
```

## 4. 페이지 공통 템플릿 (모든 기술 페이지에 동일 적용)

```markdown
---
title:
part: 02-data-pipeline
tags: [chunking, rag]
stage: ingestion | indexing | retrieval | reasoning | action | presentation
maturity: 채택 | 시험 | 관찰 | 보류        # 트렌드 레이더와 연동
updated: 2026-10-11
---

# 제목

## 요약 (3줄)
## 문제와 맥락 — 왜 이 단계가 필요한가
## 핵심 개념
## 아키텍처 / 파이프라인  → [HTML 시각화 링크] + 썸네일
## 설계 시 고려사항 (체크리스트)
## 기술 선택 가이드 — 언제 무엇을 쓰나 (표: 상황 × 선택지 × 이유)
## 한계와 리스크
## 최신 트렌드 적용 — 온톨로지 / 뉴로심볼릭 / System One 등이 이 단계에서 하는 역할
## 관련 페이지 (wiki-link)  /  참고 문헌 (검증 수준 표기)
```

### 검증 수준 표기 규칙 (참고 문헌마다 부착)
| 표기 | 의미 |
|---|---|
| `[검증]` | 독립 벤치마크·논문·표준 문서 |
| `[벤더]` | 벤더 자체 주장 (수치는 재검증 필요) |
| `[2차]` | 블로그·해설 기사 |
| `[실측]` | 우리가 직접 측정 |

예: Jev의 "200배 빠르고 400배 저렴" 주장은 현재 `[벤더]`/`[2차]` 수준이며, 독립 벤치마크 미확인.

## 5. HTML 시각화 목록 (1차 12종)

| # | 파일 | 내용 | 연결 페이지 |
|---|---|---|---|
| 1 | `reference-architecture.html` | 3-Layer 전체 아키텍처 (Knowledge/Reasoning/Interaction + Cross-cutting) | 00/01 |
| 2 | `decision-tree.html` | 종합 기술 선택 결정 트리 (클릭 시 해당 md로 이동) | 00/02 |
| 3 | `trend-radar.html` | 트렌드 레이더 4분면 | 00/03 |
| 4 | `protocol-map.html` | MCP·A2A·AG-UI·A2UI·MCP Apps 관계도 | 01/05 |
| 5 | `data-pipeline-architecture.html` | 수집→파싱→청킹→임베딩/KG→인덱스→동기화 파이프라인 | 02/00 |
| 6 | `kg-construction-pipeline.html` | LLM 기반 KG 구축 흐름, 온톨로지 정렬 | 02/05 |
| 7 | `rag-pattern-evolution.html` | Naive→Agentic RAG, GraphRAG 비교 | 03/01 |
| 8 | `agent-workflow-architecture.html` | 질의 이해→라우팅(System One)→검색→추론→툴→검증→응답 | 03/00 |
| 9 | `agent-design-patterns.html` | ReAct / Plan-Execute / Reflection / Orchestrator-Worker 비교 | 03/06 |
| 10 | `multi-agent-topologies.html` | Supervisor / Hierarchical / Handoff | 03/07 |
| 11 | `dynamic-ui-architecture.html` | Agent 백엔드 ↔ AG-UI 이벤트 ↔ 컴포넌트 레지스트리 ↔ 렌더러 | 04/00 |
| 12 | `genui-streaming-sequence.html` | 스트리밍/상태 동기화 시퀀스 다이어그램 | 04/04 |

**시각화 원칙**: 외부 CDN 없이 단일 파일로 동작, 공통 색상 체계(레이어별 고정색), 호버 시 설명·클릭 시 md 이동, 라이트/다크 모두 가독.

## 6. 최신 트렌드를 어디에 끼워 넣는가 (매핑표)

| 트렌드 | 심화 페이지 | 적용되는 단계 페이지 |
|---|---|---|
| 온톨로지 / 지식그래프 | 06/ontology-renaissance, 01/03 | 02/05, 02/06, 02/08(근거 스키마), 03/10, 04/05(UI 매핑), 05/04(설명가능성) |
| 뉴로심볼릭 | 06/neuro-symbolic, 01/04 | 02/09(파이프라인 내 규칙+LLM), 03/04(근거 검증기), 03/10, 03/12(가드레일) |
| System One 모델 (Jev) | 06/system-one-models-jev, 01/02 | 02/09(분류·품질 판정), 03/02(intent 라우팅), 03/11(결정 레이어), 03/12(게이팅), 04/05(UI 선택), 05/02(비용 절감) |
| GraphRAG / LightRAG | 06/graphrag-evolution | 02/05, 03/01, 03/03 |
| Context Engineering | 06/context-engineering | 03/08, 03/03(압축) |
| Agentic 프로토콜 (MCP, A2A, AG-UI, A2UI, MCP Apps) | 06/agentic-protocols-convergence, 01/05 | 03/07, 03/09, 04/02 |
| Generative UI 3패턴 | 06/generative-ui | 04/01 ~ 04/07 |

## 7. 작성 순서 제안 (단계별 마일스톤)

1. **골격**: README, `_templates`, `_meta/writing-guide`, 각 Part `00-overview` + 레퍼런스 아키텍처 HTML(#1)
2. **Part 1 Foundations** 전체 (뒤의 모든 페이지가 참조하므로 먼저)
3. **Part 2 데이터 파이프라인** + HTML #5, #6 → 결정 매트릭스
4. **Part 3 워크플로우/RAG** + HTML #7~#10 → 결정 매트릭스
5. **Part 4 동적 UI** + HTML #11, #12 → 결정 매트릭스
6. **Part 6 트렌드 심화** (각 Part 작성 중 쌓인 내용 정리) + 트렌드 레이더 HTML(#3)
7. **Part 5 횡단 관심사**, **Part 7 케이스 스터디**, 종합 결정 트리 HTML(#2)

## 8. 미결 사항 (확정 필요)

- 위키 루트 위치: 현재 `workspace/ai-agent-wiki/` 로 가정. 기존 레포 하위로 옮길지 여부
- 언어: 본문 한국어, 기술 용어 영문 병기 (CLAUDE.md 규칙 준수)
- HTML 시각화의 상호작용 수준: 정적 SVG vs 호버/클릭 인터랙션 (제안: 인터랙션 포함)
- 프레임워크 비교(03/14)의 범위: 상용 SDK 포함 여부
