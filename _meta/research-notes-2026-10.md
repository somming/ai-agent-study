# 조사 노트 (2026-10-11 기준)

> 위키 작성 시 공통으로 인용하는 사실과 검증 수준. 페이지 간 서술이 어긋나지 않도록 **여기 적힌 표현과 검증 수준을 그대로** 쓴다. 새 사실을 추가하면 여기에 먼저 적는다.

## 결정 모델 (System One)

| 항목 | TypeSafe AI **Jev** | **OpenAI Decisions API** | 검증 수준 |
|---|---|---|---|
| 공개 | 2026-09-15 얼리액세스 | 2026-09-29 DevDay 발표(제한 프리뷰) → 2026-10-06경 퍼블릭 베타 보도 | `[2차]` 날짜 소스 충돌 |
| 성격 | **결정 전용 모델**. 비자기회귀(non-autoregressive) 단일 패스, 병렬 샘플링, RLCD(보정 결정 강화학습) 주장 | **범용 생성 모델(gpt-6-luna)의 출력을 제약한 엔드포인트** `POST /v1/decisions`. 별도 결정 전용 모델 아님 | `[벤더]` `[2차]` |
| 입력 | 상태(텍스트/JSON) + 코드로 정의한 타입드 질문 | 컨텍스트(텍스트·이미지) + 개발자 정의 질문 | `[벤더]` |
| 출력 | 선택 / 점수 / 예·아니오 + 보정 확률. 텍스트 생성 없음 | 조건 참 확률 / 선택지 중 선택 / 루브릭 점수 | `[벤더]` |
| 가격(보고) | $0.042 / 1M 입력 토큰 | $0.10 / 1M 입력 토큰, 출력·캐시 과금 없음 (미공개라는 소스도 있음) | `[2차]` 재확인 필요 |
| 지연(보고) | 70~500ms E2E; "LLM 워크플로우 대비 최대 200배 빠르고 400배 저렴" | 미보고 | `[벤더]` 독립 벤치마크 없음 |
| 한계 | 텍스트만(이미지·음성 불가), **영어 중심 학습 → 한국어 자체 평가셋 필수**, 스키마 보장 ≠ 판단 정확 | 단일 모델 종속, 출력 검증·권한·불확실성 처리는 호출자 책임 | `[2차]` |
| 기타 | 창업자 표기 Diogo/Diego Almeida 혼재. LangChain 테스트 1건 외 독립 벤치마크 미확인 | — | `[2차]` |

**서술 규칙**: 둘을 "결정 모델(Jev / Decisions API)"로 묶어 부르되, 내부 구조가 다르다는 점을 한 번은 밝힌다. 성능 수치는 항상 `[벤더]`.

## 온톨로지 · 뉴로심볼릭

- arXiv **2604.00555** (2026-03, 단일 저자): Role / Domain / Interaction 3계층 온톨로지로 LLM 에이전트 접지. 21개 산업 650+ 에이전트 운영 보고. neural[symbolic] 범주 자처. → `[검증-단일저자, 독립 재현 없음]`
- Franz Inc.(AllegroGraph) 2026-06 발표: 지식그래프를 에이전트 시맨틱 레이어로. **Gartner 2026-04 보고서가 NSAI 벤더 식별**했다고 주장. Gartner 원문 미확인 → `[벤더]`
- Neo4j NODES 세션 "Beyond GraphRAG": 많은 GraphRAG가 임베딩+그래프 탐색에 머물러 온톨로지를 간과 → `[2차]` 그래프 벤더 프레이밍
- Semantic Data Europe 2026: "택소노미는 찾게, 온톨로지는 의미를, 지식그래프는 연결을" → `[2차]`
- Kautz 분류(AAAI 2020): 소스마다 5~6종. Garcez & Lamb(2023)는 다른 표기 → `[2차]`
- Renkhoff et al., IEEE TAI 2024 (arXiv 2401.03188): 뉴로심볼릭 V&V 서베이. 신경·기호 부분을 **따로** 검증하는 기법은 부분적으로 충분, 결합 전체는 미성숙 → `[검증]`
- arXiv 2602.00731 (2026-02): 예지보전 분야 뉴로심볼릭 리뷰 → `[검증]`
- 시장 규모 "$25.2B by 2033, CAGR 36.4%": 벤더 보고서, 방법론 미확인 → `[벤더]` 인용 비권장

## RAG · 데이터 파이프라인

- **Late chunking vs Contextual retrieval** 비교 연구 arXiv **2504.19754**: contextual은 의미 일관성↑ 계산 비용↑, late chunking은 효율↑ 관련성·완결성↓ → `[검증]`
- Contextual retrieval 원형: Anthropic 방식(각 청크 앞에 문서 맥락 요약 1문장 접두) → `[벤더]`
- 하이브리드 검색: BM25 + dense + RRF가 단독보다 우수, cross-encoder 리랭커가 난이도 높은 셋에서 추가 이득 → `[2차]` 누적 보고
- 롱컨텍스트 vs RAG: 보완 관계. RULER 스트레스에서 중간 청크 누락 재현 → `[2차]` (원 논문 RULER/Lost in the Middle 2023~24 `[검증]`)
- **Microsoft GraphRAG**: 인덱싱 비용 과다, "brittle and cost-prohibitive" 비판 → `[벤더]` 경쟁사 평가 포함
- **LightRAG**: 엔티티-관계 그래프 + 저수준(엔티티)/고수준(주제) 이중 검색, **증분 업데이트** 지원. "GraphRAG 61만 토큰 쓸 때 100토큰 미만" 수치는 2차 요약 → `[2차]`
- **HippoRAG 2**: Personalized PageRank 기반 연상 검색. arXiv **2607.11683**(RAGU, 2026-07) 재분석: 장황한 답변 프롬프트에서의 우위는 대체로 **포맷 효과**, 포맷 통제 후에는 최난도 멀티홉 MuSiQue에서만 뚜렷한 우위(54.4 vs 40.1) → `[검증]`
- LegalGraphRAG arXiv 2605.28120 (2026-05): 법률 멀티에이전트 GraphRAG → `[검증]`
- "어떤 단일 아키텍처도 모든 시나리오에서 이기지 않는다. 그래프 RAG의 가치는 질의 복잡도에 달렸다" → `[2차]` 실무자 합의
- Agentic RAG: 검색이 에이전트 루프 "앞"이 아니라 "안"으로 → `[2차]`

## 에이전트 프레임워크 · 메모리 · 컨텍스트

- 프레임워크 재편: Swarm → **OpenAI Agents SDK**; Claude Code SDK → **Claude Agent SDK**(Claude Code 런타임을 프로그래밍 노출, hooks); **AutoGen 유지보수 모드**; **Google ADK** 2025-04 출시, Python/Java(1.0.0 2026-03)/Go/TypeScript → `[2차]`
- 구분: 벤더 네이티브 SDK(Claude, OpenAI, Google) vs 독립 프레임워크(LangGraph, CrewAI, Smolagents, Pydantic AI, AutoGen) → `[2차]`
- **LangGraph**: 방향 그래프 + 공유 상태, 체크포인팅·내구 실행. 학습 곡선 가장 가파름 → `[2차]`
- **Google ADK 컨텍스트 관리**: 세션·메모리·툴 출력·아티팩트를 구조화된 뷰로 조립, 자동 요약·무관 이벤트 필터·지연 로딩·토큰 추적 → `[벤더]`
- 메모리 생태계: 대화 컨텍스트용 vs 운영 지식용으로 분화. 벤치마크 **LoCoMo, LongMemEval, BEAM** 표준화(Mem0 보고 → `[벤더]`)
- **MemTrapBench**: 메모리 프레임워크 5종 모두 no-memory 베이스라인보다 **나쁨**. 관련 기억이 이전 과업 프레임에 모델을 고착 → `[2차]`. 결론: 메모리는 no-memory 대비로 평가
- Claude Agent SDK GitHub 스타 7.3K vs 14K 소스 충돌 → 인용 비권장

## 코드 실행형 행동 공간 (CodeAct / Code Mode / Programmatic Tool Calling)

- **CodeAct** — Wang et al., "Executable Code Actions Elicit Better LLM Agents", ICML 2024 (arXiv 2402.01030). 에이전트의 행동 공간을 JSON 툴 호출 대신 **실행 가능한 파이썬 코드**로. 17개 LLM에서 JSON/텍스트 대비 **최대 20% 높은 성공률**, GPT-4 기준 74.4% vs 52.4%·평균 턴 5.5 vs 7.6 → `[검증]` (원 논문 수치)
- **Anthropic "Code execution with MCP"** (2025-11 엔지니어링 블로그): MCP 서버를 코드 API로 노출, 모델이 코드로 호출해 툴을 온디맨드 로드·중간 데이터를 모델 밖에서 필터. Claude SDK의 **Programmatic Tool Calling**, 관련 **Tool Search Tool**(85% 토큰 절감 주장). "98.7% 절감"은 2차 인용, 독립 테스트(AIMultiple)는 78.5% → `[벤더]` `[2차]`
- **Cloudflare Code Mode** (2025-09 공개, 2026-04 Code Mode MCP 서버, 2026-07-30 포털 정책 Off/Opt-in/On/Enforced): MCP 서버 전체를 `search()` + `execute()` 두 툴로 접고 샌드박스 JS 실행. "대형 API에서 입력 토큰 99.9% 절감(117만 토큰 기준선)"은 벤더, 독립 보도는 단순 32% / 복잡 배치 81% → `[벤더]` `[2차]` 소스 간 차이 큼
- **Microsoft Agent Framework CodeAct** (BUILD 2026): 모델이 `call_tool(…)`을 호출하는 짧은 파이썬을 한 번 작성, Hyperlight 마이크로 VM에서 호출당 격리 실행. 알파 패키지. 단일 예시 6,890 → 2,489 토큰 → `[벤더]`
- **Hugging Face smolagents**: `CodeAgent`가 기본값(코드로 툴 호출), `ToolCallingAgent`는 JSON. 샌드박스 옵션 Modal / Blaxel / E2B / Docker / Pyodide·Deno → `[벤더]` 문서
- **OpenHands**(구 OpenDevin): 소프트웨어 에이전트 벤치마크에서 CodeAct 구현체로 자주 등장. 2026 채택 데이터 미확인 → `[2차]`
- 공통 전제: **샌드박스 필수**(네트워크·파일 격리, 자원 상한), 절감 폭은 툴 수·중간 데이터 크기에 비례(단순 과업은 작음). 이 위키의 설계 함의: 툴 호출이 코드 안에서 일어나므로 **행동 게이트를 샌드박스의 `call_tool` 경계에서 강제**해야 하고, **생성 코드 자체가 감사 레코드**가 된다
- 서술 규칙: 세 벤더 구현을 "코드 실행형 행동 공간(CodeAct 계열)"으로 묶고, 수치는 원 논문만 `[검증]`, 나머지 전부 `[벤더]`/`[2차]`

## 프로토콜

- **MCP 2026-07-28 개정**: RC 2026-05-21, 최종 2026-07-28. initialize 핸드셰이크·Mcp-Session-Id 제거 → **무상태 코어**, 라운드로빈 스케일링. **Multi Round-Trip Requests**, **Tasks 확장**(장기 실행), **MCP Apps 확장**(서버 렌더 UI). C# SDK v2.0 하위 호환 → `[2차]` 공식 changelog 재확인 권장
- **A2A**: Google 2025-04 공개, **2025-06 Linux Foundation 이관**, 100+ 기업. A2A 1.0 정황(A2UI 통합 언급)이나 전용 발표 미확인 → `[2차]`
- **AG-UI**: CopilotKit 주도, MIT, SSE 기반 타입드 이벤트(텍스트 델타, 툴 호출 시작/종료, 상태 패치), 양방향. **Microsoft .NET SDK 2026-09-25**, **AWS Bedrock AgentCore 지원 2026-03**, ThoughtWorks Radar 등재. 재단 아닌 스타트업 거버넌스 → `[2차]` `[벤더]`
- **A2UI**: Google, Apache 2.0, 선언적 UI 트리(JSON), 실행 코드가 아닌 데이터라 웹·Flutter·Android·iOS 네이티브 렌더 주장. **v0.9 2026-04-17**, "Standard" 컴포넌트 세트 → "Basic" 개명, Agent SDK 추가, Android Jetpack 라이브러리 → `[벤더]`
- **MCP Apps**: MCP 서버가 HTML UI 반환, 호스트가 샌드박스 iframe 렌더. CopilotKit 2026-01 지원 발표 → `[2차]`
- **Generative UI 3패턴** (CopilotKit 프레이밍): Controlled(AG-UI) / Declarative(A2UI, Open-JSON-UI) / Open-ended(MCP Apps, 커스텀). "대부분 팀이 프레임워크 기본값 때문에 Controlled로 흘러간다" → `[2차]` CopilotKit 자체 프레이밍
- 프로토콜은 보완 관계: MCP(툴·데이터) · A2A(에이전트 간) · AG-UI(UI 이벤트) → `[2차]`

## 평가 · 관측성

- **OTel GenAI 시맨틱 컨벤션**: 2026-10 현재 전 `gen_ai.*` 속성 **Development** 등급. v1.42.0에서 별도 저장소 `semantic-conventions-genai`로 분리, 태그 릴리스 없음(2026-09 Dash0). 2026-06 "안정화" 주장 1건은 레지스트리 미확인 → `[2차]`
- 에이전트 스팬: `invoke_agent {name}` 아래 모델·툴 호출 자식 스팬. 원격 에이전트는 CLIENT, 인프로세스는 INTERNAL. 본문 기록 opt-in → `[2차]`
- RAGAS: context precision/recall, faithfulness, answer relevancy. judge 정렬 가이드 제공. 전용 OTel 계측기 없음(수동 스팬). judge JSON 파싱 실패 → NaN이 집계에서 조용히 제외 → `[2차]`
- Datadog: Ragas 평가 자동 트레이싱, 샘플링 권고 → `[벤더]`

## 추가 조사 필요 (집필 중 확인된 공백)

본문 작성 시 노트만으로 서술이 부족했던 항목. 다음 조사 라운드에서 1차 출처를 확인해 위 절에 반영하고, 해당 페이지의 검증 수준을 갱신한다.

| 항목 | 현재 처리 | 확인할 것 | 영향 페이지 |
|---|---|---|---|
| HippoRAG 2 원 논문 | RAGU 재분석(2607.11683)만 인용 | 원 논문 arXiv 번호, PPR 메커니즘 세부 | 2/05, 3/01, 6/graphrag |
| LoCoMo / LongMemEval / BEAM 원 논문 | Mem0 보고 `[벤더]`로만 | 각 벤치마크 원 출처·측정 조건 | 3/08, 6/agent-memory |
| MemTrapBench | `[2차]`, 측정 조건 미확인 | 원 출처, 5개 프레임워크 목록, 과업 유형 | 3/08, 6/agent-memory |
| MCP 2026-07-28 개정 세부 | 2차 해설 기반 | 공식 changelog(modelcontextprotocol.io)에서 무상태·Multi Round-Trip·Tasks·Apps 최종 사양 | 1/05, 3/09, 6/protocols |
| A2A 1.0 | A2UI 통합 언급으로만 정황 | 공식 릴리스 발표 유무·일자 | 1/05, 3/07 |
| AG-UI 이벤트 정확한 이름·필드, 상태 패치가 JSON Patch(RFC 6902)인지 | 범주만 서술 | 스펙 현재 버전 | 4/02, 4/04 |
| Generative UI 효과의 독립 측정 | 벤더 일반 주장 | 과업 완료율·수정률 등 독립 연구 유무 | 4/01, 4/10, 6/generative-ui |
| first_seen 월 단위 | context-engineering·agent-memory는 2025-10으로 잠정 | 각 트렌드가 각광받기 시작한 월 | 6/*, 0/03 |
| 한국어 성능 | Jev 영어 중심 외 근거 없음 | 임베딩·리랭커·NLI·결정 모델의 한국어 벤치마크 | 2/04, 3/02~04, 3/11 |
| 결정 모델 독립 벤치마크 | 없음 | LangChain 테스트 1건 외 독립 측정 등장 여부 | 1/02, 3/11, 6/decision-models |
| 뉴로심볼릭 벤더 자료(o9 Solutions, Kognitos) | Part 1/04 참고 문헌에만 `[벤더]` | 노트에 정식 등재할지 또는 제거할지 | 1/04, 6/neuro-symbolic |
| Ontology Summit 2026 산업 세션 | Part 1/04에만 언급, 노트 미등재 | 세션 자료 공개 여부 | 1/04 |
| Part 3B·5가 `[검증]`으로 인용한 1차 자료 (Greshake 2023 간접 인젝션, Guo 2017 ECE, Gao 2022 PAL, Willison 2023 이중 LLM, OWASP LLM Top 10, BFCL) | 널리 알려진 1차 자료라 `[검증]` 표기 | 노트에 정식 등재(arXiv 번호·버전) | 3/10, 3/11, 3/13, 5/01 |
| EU AI Act 조항·시행일 | 일반 구조만, 법무 검토 사항으로 | 적용 시점·GPAI 의무 세부 | 5/04 |

## 인용 금지·주의

- 출처 불명 퍼센트(예: "GraphRAG 86% vs 32%", "에이전트 80% 더 진실") → 인용 금지
- Gartner 보고서 내용은 벤더 전언이므로 "벤더 발표에 따르면"으로만
- 소스 충돌이 있는 날짜·가격은 충돌 사실을 함께 적는다
