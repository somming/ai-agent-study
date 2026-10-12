---
title: "근거 데이터 설계 ★"
part: 02-data-pipeline
stage: knowledge
tags: [grounding, evidence, citation, provenance, confidence, contradiction, faithfulness, schema]
maturity: 채택
status: review
updated: 2026-10-11
---

# 근거 데이터 설계 ★

> **범위**: evidence 단위 정의, citation 스키마, 신뢰도 등급, 반증 데이터, 근거-답변 정합성
> **시각화**: [`assets/html/data-pipeline-architecture.html`](../assets/html/data-pipeline-architecture.html) — 벡터 경로와 그래프 경로가 합류하는 "근거 단위 확정" 노드

## 요약 (3줄)

1. 이 Part의 진짜 산출물은 인덱스가 아니라 **근거 단위(evidence unit)**다. "답변의 어느 문장이 어느 원문의 어느 위치에 기대고 있는가"를 기계가 검증하고 사람이 확인할 수 있는 최소 단위. 이것을 먼저 정하고 청킹·인덱싱을 거꾸로 설계한다.
2. 근거 단위는 하나가 아니다. **청크(검색) → 문장(하이라이트) → 트리플(경로)**를 한 citation 스키마 안에 담는 **하이브리드**가 운영 기본값이다. 어느 하나만 고르면 검색·검증·렌더 중 하나가 어색해진다.
3. 근거 데이터는 두 소비자와 **계약**을 맺는다. [Part 3/04](../03-agent-workflow-rag/04-generation-grounding.md)의 정합성 검증기는 "답변 문장 ↔ 근거 텍스트"를 대조할 수 있어야 하고, [Part 4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)의 UI는 citation 필드만으로 "제목 — 섹션 — 위치 — 버전 — 신뢰도"를 그릴 수 있어야 한다. 이 두 계약이 citation 스키마의 필수 필드를 결정한다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | ① 근거 단위 유형과 그 식별자 체계 ② citation 스키마(필수·선택 필드, 타입) ③ 신뢰도 등급 체계와 산정 규칙 ④ 상충·반증 근거의 표현과 처리 규칙 ⑤ 하류 두 소비자(검증기·UI)와의 계약 문서 |
| **언제** (When) | 답변에 출처가 붙어야 하는 모든 경우. 사실상 모든 업무용 에이전트. **생략**: 순수 변환·요약(입력이 곧 근거)이나 창작. **미뤄도 되는 것**: 상충 근거 처리(단일 정본 소스로 시작할 때), 트리플 근거(그래프 미도입 시) |
| **어떻게** (How) | ① 답변 UI 목업에서 사용자가 보고 싶은 근거 형태를 정함(링크? 하이라이트? 경로?) → ② 그 형태에 필요한 근거 단위 유형 결정 → ③ citation 스키마 초안, 필수 필드 확정 → ④ 신뢰도 등급 규칙(소스 등급 × 추출 방식 × 신선도) → ⑤ 상충 근거 규칙 → ⑥ 검증기·UI 팀과 계약 리뷰 → ⑦ [02](02-chunking-strategies.md)·[03](03-metadata-provenance.md)·[05](05-knowledge-graph-construction.md)에 역산 요구 전달 |
| **왜** (Why) | 근거 없는 답은 환각이고, 출처 없는 근거는 감사 불가다. 근거 단위를 나중에 정하면 청크 경계와 인용 범위가 안 맞아 "문서 X 어딘가"로 끝난다. 신뢰도 없는 근거는 블로그 글과 공식 규정을 같은 무게로 인용한다. 상충 처리가 없으면 개정 전후 규정이 한 답변에 섞인다 |

## 문제와 맥락

RAG는 "검색된 청크를 프롬프트에 넣으면 모델이 근거에 기반해 답한다"는 가정에서 출발했다. 운영에서 이 가정은 세 군데서 깨진다.

- **근거는 있는데 답이 근거와 다르다**: 모델이 청크를 읽고도 다른 말을 한다. 이를 잡으려면 "답변 문장 ↔ 근거 텍스트" 대조가 가능한 **단위**가 있어야 한다. 청크 전체는 너무 커서 대조가 모호하다
- **근거는 맞는데 사용자가 확인할 수 없다**: "문서 A 참조"만 있고 어느 페이지 어느 문장인지 없다. 사용자는 30쪽을 뒤진다. UI가 하이라이트하려면 **위치**가 있어야 한다
- **근거끼리 싸운다**: 2024년 규정과 2026년 개정판, 본사 지침과 지사 지침, 공식 문서와 사내 위키. 모델은 둘 다 읽고 적당히 섞는다. 이를 막으려면 근거에 **신뢰도·시점·상충 관계**가 있어야 한다

근거 데이터 설계는 이 세 문제를 **데이터 구조**로 푼다. 프롬프트로 "근거에 기반하라"고 쓰는 것은 확률적 준수이고, 구조는 검증 가능한 준수다.

## 핵심 개념

### 근거 단위 후보

| 단위 | 정의 | 검색 적합 | 검증(정합성) 적합 | 렌더 적합 | 비용 |
|---|---|---|---|---|---|
| **청크** (chunk) | 검색용 텍스트 조각(수 문장~수 문단) | 높음 (인덱스 단위) | 낮음~중 (범위가 넓어 대조 모호) | 중 ("펼쳐 보기"에 적합) | 하 (이미 있음) |
| **문장** (sentence) | 청크 안의 문장, 오프셋 보유 | 낮음 (단독 검색은 맥락 부족) | 높음 (문장 대 문장 대조) | 높음 (하이라이트) | 중 (문장 분할·오프셋) |
| **트리플** (triple) | (주어, 관계, 목적어) + 근거 문장 ID | 중 (그래프 탐색) | 높음 (구조 대조) | 중 (경로 시각화, 일반 사용자에겐 생소) | 상 (추출·검증·해소) |
| **레코드** (record) | 정형 DB 행·API 응답 | 높음 (SQL/필터) | 높음 (값 대조) | 높음 (표·카드) | 하 (이미 구조화) |
| **생성물** (derived) | VLM 캡션, LLM 요약, contextual 접두 | 중 | 낮음 (원문 아님) | 중 ("생성됨" 표시 필수) | 중 |
| **LLM 파생 페이지** (wiki·요약·종합) | LLM Wiki(사전 컴파일형 지식 베이스) 등이 원문 **여러 건**을 읽고 사전 종합한 페이지. 1차 원문이 아닌 **파생 근거**(derived evidence) `[2차]` | 높음 (종합·비교·누적 질문에 index 탐색) | 낮음 (원문 아님 — 검증은 `derived_from[]`의 원문으로 치환) | 중 ("LLM 생성 · 컴파일 시각" 표시 필수, 출처는 페이지 단위) | 중 (컴파일 선불, 원문 변경 시 재컴파일) |
| **하이브리드** | 청크로 검색, 문장으로 검증·하이라이트, 트리플로 경로, 레코드로 값 | 높음 | 높음 | 높음 | 중~상 (ID 체계 통합) |

핵심은 **식별자 체계**다. 문장은 청크 안의 오프셋으로, 트리플은 근거 문장 ID로, 생성물은 `derived_from`으로 연결되어 어느 단위에서든 **원문 위치로 내려갈 수 있어야** 한다. LLM 파생 페이지는 단일 원본이 아니라 **종합**이므로 `derived_from[]`(배열)로 여러 원문에 연결되고, 원문이 바뀌면 `stale`이 된다. 패턴 자체는 [Part 3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md)의 "질의 시 합성 vs 사전 컴파일" 소절.

### citation 스키마 (제안)

하류 두 소비자가 각자 필요한 필드를 역산해 합친 것이다. 필수(●)는 모든 citation에 반드시 있다.

| 필드 | 타입 | 필수 | 소비자 | 설명 |
|---|---|---|---|---|
| `evidence_id` | string | ● | 검증기·UI | 근거 단위 전역 ID |
| `evidence_type` | enum: chunk / sentence / triple / record / derived | ● | 둘 다 | 단위 유형 |
| `text` | string | ● | 검증기 | 대조에 쓸 근거 텍스트(문장·청크 원문, 트리플은 근거 문장, 레코드는 직렬화) |
| `doc_id`, `doc_version` | string | ● | UI·감사 | 어느 문서 어느 판 |
| `title` | string | ● | UI | 표시용 제목 |
| `locator` | object | ● | UI | `{page, bbox}` 또는 `{char_start, char_end}` + `heading_path`. 하이라이트 좌표 |
| `source_uri` | string | ● | UI | "원문 보기" 링크 |
| `source_system` | string | ● | UI·신뢰도 | 어느 시스템에서 왔나 |
| `confidence` | object | ● | 둘 다 | `{grade: A/B/C/D, factors: {...}}` 아래 등급 체계 |
| `valid_from`, `valid_to` | date | ● | 검증기·UI | 유효 기간. `valid_to` 지났으면 "만료" 배지 |
| `acl_checked` | bool | ● | 보안 | 이 사용자에게 노출 가능함이 확인됨 |
| `parent_id` | string | ○ | UI | 문장 → 청크, 청크 → 섹션 "펼쳐 보기" |
| `graph_path` | array | ○ | UI | 트리플 근거의 경로 `[(s,p,o,evidence_id), ...]` |
| `derived_from` | string | ○ | UI·검증기 | 생성물의 원본 근거 ID. 있으면 "생성됨" 표시 |
| `derived_from[]` | array | ○ | UI·검증기·신선도 | LLM 파생 페이지(위키·종합)가 종합한 원문 근거 ID 목록. 단일 `derived_from`의 복수형 — 검증기는 이 원문들로 치환해 대조하고, 재컴파일 트리거는 이 배열을 역참조한다 |
| `compiled_at` | datetime | ○ | UI·신선도 | 파생 페이지의 컴파일 시각. 원문 `doc_version` 변경 시각과 비교해 `stale` 산정 |
| `compiler_version` | string | ○ | 감사 | 컴파일한 모델·프롬프트(스키마 파일) 버전. 원문의 `pipeline_version`에 해당 |
| `stale` | bool | ○ | 검증기·UI | `derived_from[]` 중 하나라도 `compiled_at` 이후 변경·삭제됨. true면 근거 제외(등급 F) + 원문 RAG 폴백 |
| `conflicts_with` | array | ○ | 둘 다 | 상충 근거 ID 목록과 관계(`superseded_by`, `contradicts`, `scope_differs`) |
| `retrieval_score`, `rerank_score` | number | ○ | 디버그 | 검색·리랭크 점수. 사용자 노출 비권장 |
| `pipeline_version` | string | ○ | 감사 | 파서·청커·추출기 버전 묶음 |

이 필드들은 [03 메타데이터](03-metadata-provenance.md)의 다섯 축에서 온다. 메타데이터에 없는 필드는 citation에도 없다. 그래서 메타데이터를 먼저 설계해야 한다.

### 신뢰도 등급

신뢰도는 검색 점수가 아니다. 검색 점수는 "질문과 가까운가"이고 신뢰도는 "이 근거를 믿어도 되는가"다. 세 요인의 조합으로 등급을 매긴다.

| 요인 | 값 예시 | 근거 |
|---|---|---|
| **소스 등급** | 공식 규정·마스터 DB(상) / 승인된 문서(중) / 위키·메일·채팅(하) / 외부 웹(최하) | 소스 시스템과 문서 유형 택소노미 |
| **생성 방식** | 원문(상) / 규칙 추출(상) / LLM 추출·검증 통과(중) / LLM 생성물(하) | 계보 `derived_from`, 추출기 유형 |
| **신선도** | 유효 기간 내(상) / 검토 기한 경과(중) / 만료(하) | `valid_to`, `review_due` |

| 등급 | 의미 | 답변에서의 취급 |
|---|---|---|
| **A** | 공식 소스 + 원문/규칙 + 유효 | 단독 근거 가능 |
| **B** | 승인 문서 또는 검증 통과 추출 + 유효 | 단독 가능, UI에 등급 표시 |
| **C** | 비공식 소스 또는 생성물, 또는 검토 기한 경과 | 다른 근거와 함께만, "참고" 표시 |
| **D** | 만료 또는 외부 미검증 | 기본 제외. 명시 요청 시만, 경고와 함께 |
| **E** (파생) | LLM 생성 파생 페이지(위키·요약·종합), `derived_from[]` 보유, `stale=false` | 종합·비교 질의의 근거로 가능하되 "LLM 생성 · 컴파일 시각" 표시. 검증기 대조는 원문으로 치환, 문장 단위 출처·감사 요구 질의에서는 제외. 취급은 C에 준함 |
| **F** (파생) | LLM 생성, `derived_from[]` 없음 또는 `stale=true` | **인용 금지**. 재컴파일 큐 + 원문 RAG 폴백 |

E·F는 A~D와 **다른 축**("원문이 아니다")이다. E가 D(만료)보다 낮다는 뜻이 아니라 원문 등급과 섞어 비교하지 않는다는 뜻이며, `confidence.grade` enum에 E·F를 더하는 것은 추가 전용 스키마 변경이다. 등급 규칙은 **코드로** 둔다(택소노미 × 계보 × 날짜의 결정론적 함수). LLM에게 "신뢰도를 매겨라"고 하지 않는다. 그것은 보정되지 않은 숫자다.

### 상충·반증 근거

| 상충 유형 | 예 | 표현 | 처리 |
|---|---|---|---|
| **개정** (superseded) | 2024 규정 vs 2026 개정판 | `conflicts_with: [{id, relation: superseded_by}]`, 구판 `valid_to` 설정 | 기본은 현행만. "변경 이력" 질문이면 둘 다 + 관계 명시 |
| **범위 차이** (scope_differs) | 본사 지침 vs 지사 지침 | `scope` 메타데이터 + `relation: scope_differs` | 사용자 맥락(소속)으로 선택, 불명이면 둘 다 제시하고 범위 명시 |
| **진짜 모순** (contradicts) | 두 공식 문서가 다른 수치 | `relation: contradicts`, 둘 다 등급 유지 | **답변이 하나를 고르지 않는다.** 모순 사실과 두 근거를 제시, 소유자에게 알림 |
| **반증** (refutes) | 근거 A를 부정하는 근거 B | `relation: refutes` | 검증기가 반증 근거를 함께 보고 답변의 단정을 약화 |

상충 탐지는 ① 메타데이터(같은 `canonical_doc_id`의 다른 버전, 같은 주제 택소노미의 다른 범위)로 결정론적으로, ② 잔여는 NLI(자연어 추론) 모델이나 판정 모델로 "모순인가"를 닫힌 판정으로([09](09-ai-inside-the-pipeline.md)). 상충 관계는 **데이터로 기록**해 다음 질의부터 재판정 없이 쓴다.

### 두 계약

**검증기 계약 ([Part 3/04](../03-agent-workflow-rag/04-generation-grounding.md))**: 생성 모델은 답변 문장마다 `evidence_id` 목록을 구조화 출력으로 내놓는다. 검증기는 각 쌍(답변 문장, `text`)의 함의(entailment)를 판정한다. 그러려면 (a) `text`는 문장 수준으로 좁아야 하고 (b) 트리플 근거는 근거 문장 `text`를 함께 가져야 하며 (c) 생성물(`derived_from` 있음)은 검증 근거로 쓰이면 안 된다(원본으로 치환).

**UI 계약 ([Part 4/06](../04-dynamic-ui/06-hitl-ui-patterns.md))**: UI는 citation 객체 **하나만으로** 다음을 그린다 — 제목·섹션(`title`, `heading_path`), 위치(`locator`), 버전·유효(`doc_version`, `valid_to`), 등급 배지(`confidence.grade`), 생성됨 표시(`derived_from`), 상충 표시(`conflicts_with`), 원문 링크(`source_uri`), 펼쳐 보기(`parent_id`). UI가 추가 조회를 해야 한다면 스키마가 부족한 것이다.

## 아키텍처 / 파이프라인

```
[벡터 경로] 청크 (02) ──┐
[그래프 경로] 트리플 (05) ─┼─▶ 근거 단위 확정: 공통 evidence_id 체계, 문장 오프셋, 트리플 → 근거 문장, 레코드 직렬화
[정형] 레코드 ───────────┘            ▼
                           메타데이터 (03) 병합 → 신뢰도 등급 산정 (코드) → 상충 관계 기록 → ACL 플래그 (10)
                                      ▼
                           citation 스키마 인스턴스 ─▶ Part 3 검색 결과의 단위 ─▶ Part 3/04 정합성 검증 ─▶ Part 4/06 렌더
```

입력은 세 경로의 파생물과 메타데이터, 출력은 **citation 스키마를 만족하는 근거 단위 집합**이다. 필수 필드를 못 채우는 단위는 검색 결과로 나가지 않는다.

## 기술 비교표 — 이 단계에서 쓸 수 있는 선택지

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| 청크 단위만 | PoC, 참고용, "문서 링크"면 충분 | 하이라이트·문장 검증·규제 설명 | 하 | 표준 | `[2차]` 1세대 RAG |
| 문장 단위 (오프셋) | 정합성 검증·하이라이트 필요. 업무 지원 이상 | 검색 단위로 단독 사용(맥락 부족) | 중 | 안정 | `[2차]` 근거 하이라이트 UI 실무 표준 |
| 트리플·경로 단위 | 멀티홉 설명, 규제의 "왜" 요구, 그래프 보유 | 그래프 없음, 일반 사용자 대상(생소) | 상 | 신생~안정 | `[2차]` |
| 레코드 단위 | 정형 데이터(상품·가격·재고·티켓) | 비정형 서술 | 하 | 표준 | — |
| 생성물을 근거로 허용 | 도표·이미지가 핵심이고 원문 텍스트가 없음 | 검증 근거·규제 설명(원본 아님) | 중 | 신생 | `[2차]` "생성됨" 표시 필수 |
| LLM 파생 페이지(위키)를 근거로 허용 — 등급 E·F + 파생 필드 | 코퍼스 소규모·변화 느림·종합 질문 반복(LLM Wiki 하이브리드) | 문장 단위 출처·규제 감사, 실시간 갱신 | 중 (파생 필드 + 재컴파일 트리거) | 신생 (2026-04) | `[2차]` 원 gist 미확인 |
| 하이브리드 (청크 검색 + 문장 검증·렌더 + 트리플 경로 + 레코드 값) | 대부분의 운영 환경 | 단일 소스 소규모 PoC(과잉) | 중~상 | 안정 | `[2차]` |
| 신뢰도 = 검색 점수 | 없음 (안티패턴) | 모든 운영 | 하 | — | `[2차]` 혼동 사례 다수 |
| 신뢰도 = 코드 규칙 (소스 × 생성 방식 × 신선도) | 모든 운영 | — | 중 | 안정 | `[2차]` |
| 신뢰도 = LLM 자가 평가 | 없음 (비보정) | 모든 운영 | 중 | — | `[검증]` LLM 확신도 비보정 널리 보고 |
| 상충 탐지 — 메타데이터 결정론 | 버전·범위 상충 | 내용 모순 | 하 | 표준 | — |
| 상충 탐지 — NLI / 판정 모델 | 내용 모순·반증 | 대량 쌍 전수(비용 제곱) → 같은 주제 후보만 | 중 | 안정(NLI) / 신생(결정 모델) | `[검증]` NLI / `[벤더]` 결정 모델 |

### 선택 가이드

- **참고용이고 "문서 링크"면 충분하다면** → 청크 단위 + 필수 필드 최소(제목·URI·버전·등급). 이유: 검증·하이라이트가 없으면 문장 단위는 과잉
- **업무 지원 이상이고 UI에 근거 하이라이트가 있다면** → 하이브리드(청크 검색 + 문장 검증·렌더). 이유: 검증기와 UI 계약을 동시에 만족하는 최소 구성
- **멀티홉 설명이나 규제 "왜" 요구가 있다면** → 트리플·경로 추가, 트리플에 근거 문장 필수. 이유: 경로 없이는 설명 불가, 근거 문장 없이는 검증 불가
- **정형 데이터가 섞여 있다면** → 레코드 단위를 별도 `evidence_type`으로. 이유: 값 대조와 표 렌더가 텍스트와 다르다
- **코퍼스가 소규모·변화 느림이고 종합·반복 질문이 많아 LLM Wiki를 쓴다면** → 위키 페이지를 LLM 파생 페이지로 받되 등급 E·F + 파생 필드 4개(`derived_from[]`·`compiled_at`·`compiler_version`·`stale`). 이유: 원문 등급과 섞이면 "근거 있는 환각"이 A처럼 보이고, `stale` 없이는 원문 개정이 위키에 닿지 않는다
- **판단이 어려우면** → 기본값: 하이브리드(청크 + 문장) + 코드 규칙 신뢰도 + 메타데이터 결정론 상충 탐지, 트리플은 그래프 도입 시 추가. 이유: 두 계약을 만족하면서 확장 가능

## 설계 시 고려사항 (체크리스트)

- [ ] UI 목업에서 사용자가 보고 싶은 근거 형태(링크 / 하이라이트 / 경로 / 표)를 확인했는가
- [ ] 근거 단위 유형별 식별자가 원문 위치(문서·버전·페이지·오프셋)로 내려갈 수 있는가
- [ ] citation 필수 필드가 검증기·UI 두 계약에서 역산되었고 양 팀이 리뷰했는가
- [ ] 필수 필드를 못 채운 근거 단위는 검색 결과에서 제외되는가
- [ ] 신뢰도 등급이 코드 규칙(소스 × 생성 방식 × 신선도)이고 LLM 자가 평가가 아닌가
- [ ] 검색 점수와 신뢰도 등급을 UI에서 혼동하지 않게 분리했는가
- [ ] 생성물(캡션·요약·접두)이 `derived_from`을 갖고 검증 근거에서 원본으로 치환되는가
- [ ] LLM 파생 페이지가 등급 E·F와 `derived_from[]`·`compiled_at`·`compiler_version`·`stale`을 갖고, `stale=true`는 인용되지 않는가
- [ ] 트리플 근거가 근거 문장 `text`를 함께 갖는가
- [ ] 상충 유형(개정 / 범위 / 모순 / 반증)별 표현과 답변 처리 규칙이 있는가
- [ ] 진짜 모순은 답변이 하나를 고르지 않고 둘 다 제시 + 소유자 알림이 되는가
- [ ] `acl_checked`가 검색 후·렌더 전에 반드시 설정되는가([10](10-access-control-governance.md))
- [ ] citation 스키마 버전이 있고 Part 3·4 소비자와 호환 정책이 있는가

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| 근거가 있어도 모델이 다르게 답함 | 근거 데이터만으로는 환각을 못 막음 | 검증기 계약([Part 3/04](../03-agent-workflow-rag/04-generation-grounding.md))이 반드시 뒤따라야 함. 이 페이지는 검증 **가능하게** 만들 뿐 |
| 문장 분할의 언어 의존 | 한국어 문장 경계(인용·괄호·목록) 오분할 → 오프셋 어긋남 | 한국어 문장 분할기 `[실측]`, 분할 실패 시 청크 폴백 |
| 신뢰도 등급의 소스 택소노미 의존 | 택소노미가 없거나 틀리면 등급도 틀림 | 소스 등급을 소유자가 승인, 분기 점검 |
| 상충 탐지의 비용 | 쌍 전수 비교는 제곱 | 같은 주제·같은 정본의 후보만, 관계는 데이터로 기록해 재사용 |
| 트리플 근거의 사용자 이해도 | 경로 시각화를 일반 사용자가 못 읽음 | 경로를 자연어 한 문장으로 렌더 + 펼치면 경로 |
| 생성물의 근거 자격 | VLM 캡션이 틀리면 "근거 있는 환각" | "생성됨" 표시, 등급 C 이하, 검증 근거 불허 |
| LLM 파생 페이지의 **환각 누적** | 위키에 들어간 오류가 다음 답의 근거가 되고, 그 답이 다시 페이지로 저장되어(compounding) 오류가 누적된다. 출처가 페이지 단위라 어느 문장이 어느 원문에서 왔는지 흐려짐 `[2차]` | 등급 E·F로 원문과 분리, 검증기는 `derived_from[]` 원문으로 치환, 환류(새 페이지 저장)는 검증 통과 답만, lint 모순 보고의 소유자 지정 |
| 원문 변경이 파생 페이지에 전파되지 않음 | 원문이 개정·삭제됐는데 위키 페이지가 옛 내용으로 답함 | 위키를 [07](07-freshness-and-sync.md) 삭제·변경 레인의 **소비자**로 등록, `derived_from[]` 역참조 → `stale=true` → 재컴파일 전까지 인용 금지(F) |
| 스키마 변경의 파급 | 필드 추가가 Part 3·4 양쪽 변경 | 스키마 버전, 선택 필드로 추가 후 승격 |
| 독립 벤치마크 부재 | 근거 단위 설계가 정답률·신뢰에 미치는 효과의 공개 측정 드묾 | 자체 골든셋으로 정합성·사용자 확인율 `[실측]` |

## 운영 설계 — 확장성 · 안정성 · 유지보수

용어·판단 기준은 [Part 1/07 운영 용어](../01-foundations/07-operations-vocabulary.md). 이 단계의 산출물은 Part 3·4와 맺은 **계약(citation 스키마)**이므로, 운영의 핵심은 처리량보다 **계약을 깨지 않고 바꾸는 절차**와 **필수 필드를 못 채울 때 fail-closed로 동작하는 것**이다. 스키마 변경의 파급은 [한계와 리스크] 참조.

### 확장성 — 10배가 되면 무엇이 먼저 깨지는가

| 확장 축 | 10배 시 깨지는 것 | 설계 수단 | 비용이 비례하는 변수 |
|---|---|---|---|
| 데이터 (문서·근거 단위 수) | 문장 단위 오프셋 레코드가 청크의 수 배. 상충 후보 쌍이 같은 주제 안에서도 제곱. 신뢰도 등급을 질의 시 계산하면 메타데이터 조인 폭증 | 문장은 텍스트 복제 없이 **오프셋 배열**로만 저장(텍스트는 청크에서 절단), 상충 후보는 같은 `canonical_doc_id`·같은 주제 택소노미로 블로킹, 등급은 **색인 시 선계산** + 신선도 이벤트([07](07-freshness-and-sync.md))에서만 재계산 | 근거 단위 수(저장), 상충 후보 쌍 수(NLI 호출), 등급 재계산 이벤트 수 |
| 사용자·요청 | 질의당 citation 조립(메타데이터 조인 + ACL 재확인 + 상충 조회)이 p95를 지배 | 조립을 무상태 함수로 두고 `(evidence_id, doc_version)` 키 캐시, ACL 재확인은 후보를 묶어 1회 배치 호출, k 상한 | QPS × k × 조인 수 |
| 소비자 (UI 표면·에이전트 수) | 소비자마다 다른 필드 요구 → 스키마 비대 | 필수 필드는 두 계약의 교집합으로 고정, 소비자별 요구는 **선택 필드**로 | 소비자 수 × 선택 필드 수 |
| 테넌트 | 소스 등급 택소노미가 테넌트마다 다름 | 등급 매트릭스에 테넌트 키, 기본 매트릭스 상속 | 테넌트 수 |

### 운영 안정성 — 장애 모드와 대응

| 장애 모드 | 사용자에게 보이는 증상 | 탐지 신호 | 자동 대응 (폴백·저하) | 사람 개입 기준 |
|---|---|---|---|---|
| 메타데이터 저장소·ACL 서비스 다운 (의존 서비스 다운) | 필수 필드(`doc_version`, `acl_checked`)를 못 채움 | 조립 실패율, 의존 서비스 오류율 | **fail-closed**: 필수 필드 미충족 근거는 제외. 근거가 하나도 안 남으면 Part 3에 "근거 확인 불가" 신호 → 답변을 내지 않거나 저하 표시. 근거 없이 답하지 않는다 | 다운이 예시 5분 초과 → 온콜. ACL 재확인 불가 시 [10](10-access-control-governance.md) 정책대로 거부 |
| 조립 지연 급증 | 답변 지연 | 조립 p95, 조인별 지연 | k 축소 → 선택 필드(`graph_path`, `conflicts_with`) 생략하고 `assembly=partial` 플래그 → 캐시 적중만 허용. 플래그는 계약 ② `degradation[]`에 `evidence_partial`로 전달(불균형 ④ 반영) | p95가 SLO(예: 200ms) 2배 5분 지속 |
| 필수 필드 누락 폭증 (검증 실패 폭증) | 검색 결과에서 특정 소스 근거가 전부 사라짐 | 소스별 제외율 급변 | 제외는 자동. 상류([03](03-metadata-provenance.md)) 스키마·게이트 변경 알림 | 제외율 예시 5%p 상승 → 상류 변경 역추적 |
| locator 불일치 (데이터 불일치) | 하이라이트가 엉뚱한 문장을 가리킴 | 샘플 검증 `text == original[locator]` 불일치율, UI 하이라이트 실패 신고 | 불일치 근거는 `locator_verified=false`로 저하(펼쳐 보기만, 하이라이트 없음) + 해당 문서 재처리 큐 | 재파싱 후 오프셋 미갱신 경로 확인 |
| 상충 탐지기(NLI·판정 모델) 다운 (단계 고유) | 모순 근거가 섞여 제시될 수 있음 | 호출 오류율 | 메타데이터 결정론 상충(개정·범위)만 적용하고 `conflict_check=metadata_only` 플래그 → 계약 ② `degradation[]`에 `conflict_check_metadata_only`로 전달되어 Part 4가 "내용 모순 미검사"를 고지(불균형 ④ 반영) | 다운 장기화 시 고위험 컬렉션(규정·가격)은 등급 A·B만 노출 |
| 등급 오산정 (단계 고유) | 비공식 문서가 A등급으로, 공식 규정이 C로 | 소스 등급 택소노미 변경 이력, 샘플 감사, 사용자 신고 | 등급은 결정론 함수이므로 입력(택소노미·계보·날짜) 오류. 택소노미 롤백 후 배치 재계산 | 등급 상향 방향 오류(비공식 → A) 1건이라도 즉시 |
| 한국어 문장 분할 실패 (단계 고유) | 문장 하이라이트가 반 토막 | 분할 결과 길이 이상치, 오프셋 검증 실패 | 해당 청크는 **청크 단위로 폴백**(`evidence_type=chunk`) | 특정 문서 유형(목록·인용 많음)에 집중되면 분할기 규칙 보강 |

SLI 후보: ① citation 필수 필드 완전성(노출된 근거 중 100%) ② locator 유효율(샘플) ③ 조립 p95 ④ 하류 정합성 통과율(Part 3/04와 공유 지표). SLO 초안(예시): 완전성 = 100%(게이트 보장), locator 유효율 ≥ 99.9%, 조립 p95 ≤ 200ms. 쓰기 행동의 멱등성: 근거 단위 레코드는 `(doc_id, doc_version, pipeline_version)`에서 결정론적으로 재생성되는 **파생물**이라 재처리가 멱등. 상충 관계는 `(evidence_a, evidence_b, 탐지 규칙/모델 버전)` 키로 기록해 재판정 없이 재사용. `acl_checked`는 **요청 단위**이며 절대 영속화하지 않는다(사용자·시점에 종속). 이 원칙은 **캐시된 답변에도 적용**된다 — 결과·시맨틱 캐시([Part 3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md)·[Part 3/03](../03-agent-workflow-rag/03-retrieval-reranking.md)·[Part 5/02](../05-cross-cutting/02-cost-latency-model-routing.md))에 저장된 응답의 citation은 `acl_checked`가 소거된 것으로 간주하고, 캐시 키에 **주체 집합 해시**를 넣어 다른 주체에게 재사용되지 않게 하며, 캐시 TTL ≤ 주체 해석 TTL([10](10-access-control-governance.md) 예시 5분), ACL·톰스톤 이벤트가 캐시 무효화를 트리거한다(불균형 ② 반영).

### 유지보수 — 버전·변경·호환

| 버전 관리 대상 | 바꾸면 영향받는 것 | 회귀 테스트 | 함께 배포할 것 (호환) |
|---|---|---|---|
| citation 스키마 (버전) | **Part 3/04 검증기, Part 4/06 렌더러** — 두 소비자 | 스키마 호환 테스트(추가 전용, 소비자 tolerant reader), 계약 테스트(샘플 citation으로 검증기·렌더러 실행) | 메이저 변경 시 두 소비자와 동시 배포 또는 이중 버전 병행 기간 |
| `evidence_type`·`conflicts_with.relation` enum | 검증기 분기, 렌더 분기 | enum 소비 테스트 | 온톨로지 매핑([06](06-ontology-driven-modeling.md)) |
| 신뢰도 등급 규칙 (소스 등급 × 생성 방식 × 신선도 매트릭스) | 모든 근거의 등급 → 노출 여부 | 골든 근거셋 등급 비교, 등급 분포 diff | 배치 재계산(재색인 없음), Part 4 등급 배지 의미 |
| 소스 등급 택소노미 | 등급 입력 | 택소노미 변경 → 등급 분포 변화 리포트 | 데이터 소유자 승인 |
| 문장 분할기 (버전·언어 규칙) | 문장 단위 오프셋 전체 | 한국어 분할 평가셋, 오프셋 왕복 | 영향 문서 재처리(임베딩 재계산 없음) |
| 상충 탐지 규칙·모델 버전 | `conflicts_with` 관계 | 라벨된 상충 쌍 셋 | 기존 관계 재판정 배치(블로킹 범위만) |

롤백 단위는 "citation 스키마 버전 + enum + 등급 규칙 버전"이며, 소비자 두 곳이 같은 버전을 참조한다. 기술 부채로 기록할 것: 트리플 근거 미도입(그래프 미도입 시) 상태와 도입 조건, 상충 처리를 단일 정본 소스로 시작해 미룬 범위, 등급 재계산을 신선도 이벤트가 아니라 야간 배치로만 하는 경우의 지연, 한국어 분할기 `[실측]` 미완.

### 변화 대응 — 무엇이 바뀌고, 어느 경계에서 흡수하는가

| 변화 유형 | 이 단계에서 바뀌는 것 | 안정적으로 유지할 계약·인터페이스 | 교체 가능한 구현 | 흡수 방식 | 전파 범위 |
|---|---|---|---|---|---|
| **지식** (가장 잦음) | 문서 새 판 → 근거 단위 재생성, 구판 `valid_to`·`superseded_by` 설정, 삭제 → 근거 제외 | `evidence_id` 체계, citation 스키마 | — | 문서 단위 파생물 재생성(증분). 개정 상충은 메타데이터 규칙으로 자동 기록. 삭제는 톰스톤이 등급·노출에 즉시 반영 | 해당 문서 근거 + 그 근거를 캐시한 Part 3 답변 캐시 |
| **지식 — LLM 파생 페이지** (LLM Wiki 사용 시) | 원문 판·삭제 → `derived_from[]` 역참조로 영향 페이지 `stale=true` → 재컴파일 | citation 스키마의 파생 필드(`derived_from[]`, `compiled_at`, `compiler_version`, `stale`) | 컴파일러(모델·프롬프트) | **재컴파일 트리거**: 위키를 [07](07-freshness-and-sync.md) 변경 큐 삭제·변경 레인의 소비자로 등록. 삭제는 파생 페이지 제거까지 전파 | 파생 페이지 + 그 페이지를 근거로 한 Part 3 결과 캐시 |
| **규칙·정책** (두 번째) | 소스 등급 승격·강등(위키가 공식 소스로), 등급별 노출 정책, 상충 처리 규칙 | 등급 함수 인터페이스(`grade(source, method, freshness) → A~D`) | 등급 매트릭스 값 | **설정 테이블 교체 + 배치 재계산**(재색인 없음) + 시행일 기록 | 영향 소스의 근거 등급, Part 4 배지 |
| **도메인 모델** | `evidence_type` 추가(예: 이미지 영역), 상충 관계 유형 추가, 주제 택소노미 변경 | enum은 온톨로지 참조 | enum 값 | 온톨로지 매핑 + 스키마 선택 필드로 추가 후 승격 | 검증기·렌더러 분기 |
| **모델** | 상충 탐지 NLI·판정 모델 교체, 문장 분할기 교체 | `is_conflict(a, b) → {relation, confidence}`, `split(text) → [offsets]` 포트 | 어댑터(NLI / 결정 모델 / 규칙) | 어댑터 교체 + 라벨셋 회귀 + 기존 관계 샘플 재판정 | 상충 관계 레코드, 문장 오프셋 |
| **사용자 환경** | 새 UI 표면(모바일 — 짧은 `text` 요구), 새 언어, 사용자 소속에 따른 `scope` 해석 | citation 스키마(변경 없음) | 렌더러(Part 4) | 렌더러 쪽에서 흡수. 스키마는 선택 필드 추가로만 대응(예: `text_short`) | Part 4만 |
| **프로토콜** | Part 4 UI 프로토콜 변경, Part 3 검색 결과 전달 형식 변경 | citation 스키마 | 소비자 어댑터 | 소비자 측 어댑터. 스키마는 불변 | 소비자 경계만 |

가장 자주 바뀌는 것: **지식**(문서 판·삭제) — 증분 재생성으로 흡수. 두 번째는 **규칙**(소스 등급·노출 정책)으로, 등급을 코드가 아니라 **설정 테이블 × 결정론 함수**로 두면 재색인 없이 배치 재계산으로 끝난다. 코드 밖 설정으로 둘 것: 등급 매트릭스, 소스 등급 택소노미, `evidence_type`별 필수 필드 목록, 상충 후보 블로킹 규칙, k 상한, 등급별 노출 정책(D 기본 제외 등). 포트·어댑터 뒤에 둘 외부 의존: NLI·판정 모델, 메타데이터 저장소, ACL 재확인 서비스, 문장 분할기. 시점 재현성: citation에 `doc_version`·`pipeline_version`·등급 규칙 버전이 있어 "그때 그 근거가 왜 B등급이었나"를 복원한다.

### 관측성 — 이 단계에서 무엇을 기록·측정·알림하는가

운영 안정성 표의 "탐지 신호"가 **실제로 어디서 나오는지**를 설계한다. 기준은 [Part 1/07 관측성 설계 요소](../01-foundations/07-operations-vocabulary.md)와 [Part 5/03](../05-cross-cutting/03-observability-tracing.md). 이 단계는 **두 시점**에 스팬을 낸다 — 색인 시 근거 단위 생성(`evidence.build`, 문서 트레이스)과 질의 시 citation 조립(`evidence.assemble`, Part 3 `retrieval.assemble`의 자식). 질의 시 스팬이 Part 2에서 유일하게 사용자 요청 트레이스에 직접 들어가므로, `request_id`·`agent.evidence_ids[]`·`agent.acl_checked`를 1/07 공통 속성 그대로 쓴다. citation `text` 본문은 스팬에 넣지 않고 `evidence_id`만 — 5/03의 "citation은 청크 ID를 그대로" 원칙이 트레이스 자동 검증(citation ⊂ 검색 결과)의 전제다.

| 기록 대상 | 스팬 / 이벤트 (이름, 부모) | 핵심 속성 (버전 키 포함) | 지표 (→ 어느 SLI) | 샘플링 · 보존 · PII |
|---|---|---|---|---|
| 근거 단위 생성 (색인 시) | `evidence.build` (부모: 문서 트레이스 `ingest.fetch`) | `doc_id`, `doc_version`, `tenant_id`, `agent.pipeline_version`, `agent.citation_schema_version`, `agent.grade_rules_version`(→ `agent.rules_version`), `agent.source_grade_taxonomy_version`, `agent.sentence_splitter_version`, `agent.evidence_count{sentence,chunk,triple}`, `agent.chunk_fallback_count`(분할 실패 → 청크 단위), `agent.grade_distribution{A,B,C,D}`, `agent.outcome` | `evidence.build.chunk_fallback_ratio{doc_type}` → "한국어 문장 분할 실패", `evidence.build.sentence_len` 히스토그램 → "분할 결과 길이 이상치", `evidence.build.grade_distribution{source_grade,grade}` → "등급 분포"(택소노미 버전 변경 전후 diff = 등급 오산정 탐지) | 메타 전수. 문장 텍스트는 기록 금지(오프셋만) |
| 상충 탐지 (색인 시) | `evidence.conflict` (부모 `evidence.build`), 내용 모순 판정은 자식 `evidence.conflict.nli`(`gen_ai.*` 또는 결정 모델 속성) | `agent.conflict_method`(metadata / nli / decision_model), `agent.conflict_model_version`, `agent.candidate_pairs`(블로킹 후), `agent.conflicts_found{revision,scope,content}`, `agent.confidence` 분포, `gen_ai.usage.*`, `agent.cost_estimate` | `evidence.conflict.errors` → "상충 탐지기 호출 오류율", `evidence.conflict.mode{nli,metadata_only}` 비율, `evidence.conflict.cost_per_1k_pairs` → 대시보드 ③ | 전수. 쌍의 텍스트는 기록 금지 |
| locator 샘플 검증 | `evidence.verify` 배치 이벤트 (부모 없음, 대상 문서 트레이스에 링크) | `agent.sample_size`, `agent.locator_valid_count`, `agent.mismatch_evidence_ids[]`, `agent.parser.version`·`agent.chunk_version`(미갱신 경로 추적) | `evidence.verify.locator_valid_ratio` → **SLI ② locator 유효율**, 탐지 신호 "`text == original[locator]` 불일치율" — 01·02도 이 지표를 자기 버전 키로 분해해 읽음 | 일 1회(예시). 보존 90일 |
| citation 조립 (질의 시) | `evidence.assemble` (부모: Part 3 `retrieval.assemble`), 자식 `evidence.assemble.meta_join` · `evidence.assemble.acl_check` · `evidence.assemble.conflict_lookup` | `request_id`, `tenant_id`, `agent.evidence_ids[]`, `agent.evidence_count{candidate,exposed,excluded}`, `agent.exclusion_reasons[]`(missing_field / meta_unavailable / acl_denied / grade_d), `agent.acl_checked`(bool, 요청 단위), `agent.subject_set_hash`, `agent.citation_schema_version`, `agent.grade_rules_version`, `agent.index_version`, `agent.cache_hit`, `agent.degradation[]`(`evidence_partial`, `conflict_check_metadata_only`), `agent.outcome`(success / degraded / blocked) | `evidence.assemble.errors{dependency}` → "조립 실패율·의존 서비스 오류율", `evidence.assemble.duration_ms{join}` → **SLI ③ 조립 p95**, "조인별 지연", `evidence.assemble.excluded_ratio{source,reason}` → "소스별 제외율", `evidence.assemble.zero_evidence` 카운터(근거 0건 → Part 3 신호), `evidence.assemble.acl_unchecked_exposed`(0이어야 함) → **SLI ① 필수 필드 완전성**의 보장 검증 | 메타 전수(사용자 턴 트레이스의 일부). 본문 없음. 사용자 ID 없음(`subject_set_hash`만). `blocked`·`degraded` 턴 100% |
| 파생 페이지 신선도 (LLM Wiki 사용 시) | `wiki.ingest`(재컴파일, 문서 단위 트레이스) / `evidence.assemble`에 파생 근거 속성 추가 — 스팬 이름은 [Part 1/07](../01-foundations/07-operations-vocabulary.md) 규약 | `agent.evidence_derived`(bool), `derived_from[]` 개수, `agent.compiler_version`, `agent.stale_excluded_count`, 재컴파일 지연(원문 변경 시각 − `wiki.ingest` 완료) | **`stale` 비율**(노출 후보 중 stale로 제외된 파생 근거 비율), 재컴파일 지연 p95, 파생 근거(E) 노출 비율 → "원문 변경 미전파" 탐지 | 메타 전수. 페이지 본문 없음 |
| 하류 정합성 (공유) | Part 3/04 검증기 스팬 (이 단계 소유 아님) | `agent.citation_schema_version`, `agent.evidence_type` | citation ⊂ 검색 결과 비율·검증기 통과율을 `citation_schema_version`·`evidence_type`으로 분해 → **SLI ④ 하류 정합성 통과율** | Part 3 정책 |

질의 시 `evidence.assemble`은 사용자 턴 트레이스 안에 있으므로 샘플링 정책도 [Part 5/03](../05-cross-cutting/03-observability-tracing.md)의 턴 정책을 따른다 — 메타 100%, `blocked`(근거 0건)·`degraded`(partial·metadata_only) 턴은 100% 보존, citation `text`는 어떤 경우에도 스팬에 넣지 않고 `evidence_id`로 본문 저장소를 역참조한다.

| 알림 | 조건 (예시) | 심각도 | 런북 (첫 3단계) |
|---|---|---|---|
| 근거 없는 답변 불가 (장애 모드: 메타데이터 저장소·ACL 서비스 다운) | `evidence.assemble.errors{dependency}` > 1%가 5분 지속, 또는 `zero_evidence` 비율 > 5% | P1 | ① 실패 의존(`meta` / `acl`)과 03·10 가용성 알림 대조 ② fail-closed 유지 확인(`acl_unchecked_exposed`=0) ③ Part 3에 "근거 확인 불가" 저하가 전달되는지 확인 후 온콜 |
| 답변 지연 (장애 모드: 조립 지연 급증) | `evidence.assemble.duration_ms` p95 > 400ms(SLO 2배)가 5분 지속 | P2 | ① `{join}` 분해로 느린 조인(meta_join / acl_check / conflict_lookup) 식별 ② `evidence_partial` 저하가 작동하고 계약 ②로 전달되는지 확인 ③ 캐시 적중률·k 상한 조정 |
| 특정 소스 근거 실종 (장애 모드: 필수 필드 누락 폭증) | `evidence.assemble.excluded_ratio{source,reason=missing_field}`가 7일 베이스라인 대비 +5%p 1시간 | P2 | ① 누락 필드 종류 확인 ② 03 `index.meta.gate` 규칙·스키마 버전 변경 이력 대조 ③ 상류 수정 후 영향 문서 근거 단위 재생성 |
| 하이라이트 어긋남 (장애 모드: locator 불일치) | `evidence.verify.locator_valid_ratio` < 99.9%(일 샘플), 또는 UI 하이라이트 실패 신고 ≥ 3건/일 | P2 | ① 불일치 근거의 `parser.version`·`chunk_version`이 생성 시점과 다른지 확인 ② `locator_verified=false` 저하가 적용 중인지 확인 ③ 해당 문서 재처리 큐 투입 |
| 등급 역전 (장애 모드: 등급 오산정) | 택소노미 버전 변경 후 `grade_distribution` diff에서 상향(비공식 → A) ≥ 1, 또는 샘플 감사·신고 1건 | P1 | ① 소스 등급 택소노미 변경 이력·승인 확인 ② 택소노미 롤백 후 배치 재계산 ③ 상향된 근거가 노출된 턴 범위를 `grade_rules_version`으로 산정 |
| 저하 비율 급변 (장애 모드: 상충 탐지기 다운 · 한국어 문장 분할 실패) | `evidence.conflict.mode{metadata_only}` > 50%가 30분 지속, 또는 `evidence.build.chunk_fallback_ratio{doc_type}` > 20% | P3 | ① 탐지기 다운이면 고위험 컬렉션(규정·가격)을 등급 A·B만 노출로 전환 ② 분할 실패가 특정 문서 유형(목록·인용)에 집중되면 분할기 규칙 보강 ③ 복구 후 배치 재판정·재분할 |

대시보드에서 이 단계를 보는 뷰: ② Knowledge 패널 — 조립 p95(조인별), 제외율(소스 × 사유), locator 유효율, 등급 분포(택소노미 버전별), 저하 플래그 비율; 질의 시 지표는 Retrieval 패널에 `citation_schema_version`으로도 표시 / ③ 비용 패널 — 상충 탐지(NLI) 호출 비용(1천 쌍당) / 감사 추적으로 넘기는 필드(관측 로그와 분리): 거버넌스 접점 표의 `evidence_id`·버전 3종·등급 + 규칙 버전 + 입력 요인·`conflicts_with` + 탐지 수단·`acl_checked` 결과 + 주체 집합 해시·저하 플래그 — 관측 스팬은 집계 가능한 ID·코드만 갖고, 근거별 등급 산정 입력 요인은 감사 저장소에 / 이 단계의 관측이 없으면 못 보는 장애: **비공식 문서가 A등급으로 노출되는 등급 오산정** — 등급은 결정론 함수라 오류 로그가 없고 조립도 성공으로 끝난다. `grade_distribution{source_grade,grade}`를 택소노미·규칙 버전별로 기록하지 않으면 사용자 신고 전까지 모른다.

### 거버넌스 접점 — 정책 · 가드레일 · 승인 · 감사

| 항목 | 이 단계에서의 답 |
|---|---|
| 위험 등급 (1~5, 가역성 × 영향 범위) | 근거 노출은 **등급 2**(여러 사용자에게 보임, 가역). 단 `acl_checked`는 **보안 게이트**라 등급과 무관하게 규칙만·fail-closed(캐시 적중 응답도 예외 아님, 불균형 ② 반영). 소스 등급 택소노미 변경(공식 소스 지정)은 등급 3 → 데이터 소유자 승인 1인 |
| 가드레일 위치 (입력 / 출력 / 행동) | **출력**: ① citation 완전성 게이트(필수 필드 미충족 → 차단) ② 등급 D 기본 제외·C "참고" 표시(저하) ③ 생성물(`derived_from`)은 검증 근거 불허(원본으로 치환) ④ `acl_checked` 미설정 근거 렌더 차단. 행동 가드레일 해당 없음(외부 행동 없음) |
| 게이트 출력 (통과 / 차단 / 승인 / 저하)과 판정 수단 | 통과 / 차단(필수 필드·ACL) / 저하(등급 C·D, `assembly=partial`, `conflict_check=metadata_only`). 저하 플래그 둘은 Part 3 응답을 거쳐 계약 ② 봉투([Part 4/03](../04-dynamic-ui/03-component-registry-schema.md))의 `degradation[]`에 각각 `evidence_partial`·`conflict_check_metadata_only`로 **전달**되고, 표시 variant·고지 문구는 Part 4가 맡는다(불균형 ④ 반영). 판정은 **규칙만**(등급 함수·필드 검증). 내용 모순 판정은 NLI·판정 모델이되 결과는 "둘 다 제시"로 이어질 뿐 **하나를 고르지 않는다** |
| 소유자 (A) | 아키텍트(citation 스키마 계약 — Part 2·3·4에 걸침), 데이터 소유자(소스 등급 택소노미·노출 정책). AI 엔지니어가 R, Part 3·4 담당이 C |
| 감사 추적에 남길 필드 | `evidence_id`, `doc_id`·`doc_version`·`pipeline_version`, `confidence.grade` + 등급 규칙 버전 + 입력 요인, `conflicts_with` + 탐지 수단·버전, `acl_checked` 결과 + 주체 집합 해시(요청 단위), citation 스키마 버전, 저하 플래그 |

### 운영 준비도 체크
- [ ] 필수 필드 미충족 근거가 노출되지 않고(fail-closed), 근거가 0건일 때 Part 3에 신호가 가는가
- [ ] `acl_checked`가 요청 단위로 설정되고 미설정 근거는 렌더가 차단되는가. 캐시 적중 응답도 주체 집합 해시 키 또는 재확인으로 같은 원칙을 지키는가(불균형 ② 반영)
- [ ] 등급이 색인 시 선계산되고 신선도 이벤트·규칙 변경 시 배치 재계산되는가
- [ ] locator 유효율 샘플 검증이 주기적으로 돌고 불일치는 저하 + 재처리인가
- [ ] 상충 탐지기 다운 시 메타데이터 결정론 상충만으로 저하 운영이 가능한가
- [ ] citation 스키마 변경이 추가 전용이고 Part 3·4와 계약 테스트가 CI에 있는가
- [ ] 등급 매트릭스·소스 등급 택소노미가 코드 밖 설정이고 변경에 승인·시행일이 있는가
- [ ] 한국어 문장 분할 실패 시 청크 단위 폴백이 자동인가
- [ ] 운영 안정성 표의 탐지 신호마다 원천 스팬·지표(`evidence.assemble.errors{dependency}`, `duration_ms{join}`, `excluded_ratio{source,reason}`, `evidence.verify.locator_valid_ratio`, `grade_distribution` diff, `conflict.mode`, `chunk_fallback_ratio`)가 있고 `citation_schema_version`·`grade_rules_version`으로 분해되는가
- [ ] 알림마다 런북이 있고, `evidence.assemble` 스팬이 `request_id`·`agent.evidence_ids[]`·`agent.acl_checked`를 1/07 공통 속성 그대로 갖는가

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 온톨로지 | `evidence_type`·소스 등급·주제 범위의 어휘를 온톨로지에서. 트리플 근거의 관계명이 온톨로지 관계라야 경로를 자연어로 렌더할 수 있다. 벡터 경로와 그래프 경로가 같은 어휘로 합류하는 지점 | 그래프 또는 구조화 메타데이터 보유 | `[2차]` |
| 생성물의 근거화 | VLM 캡션·LLM 요약·contextual 접두가 인덱스에 섞이며 "원문 vs 생성"의 구분이 citation 스키마의 필수 요소가 됨 | 멀티모달·맥락 보강 사용 시 | `[2차]` |
| Agentic RAG | 검색이 에이전트 루프 "안"으로 들어오면서 근거 단위가 한 번이 아니라 여러 턴에 걸쳐 누적. `evidence_id`의 턴 간 안정성과 중복 제거가 필요 | 에이전트형 검색 | `[2차]` |
| LLM Wiki(사전 컴파일형 지식 베이스) | LLM이 쓴 종합 페이지가 근거 후보에 들어오며 "원문 vs 파생"의 구분이 등급(E·F)과 필드(`derived_from[]`·`compiled_at`·`compiler_version`·`stale`)로 스키마에 들어감. 환각 누적을 막는 수단은 **원문 치환 검증 + 재컴파일 트리거** | 코퍼스 소규모·변화 느림·종합 질문 반복([Part 3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) 하이브리드) | `[2차]` |

## 관련 페이지

- [02 청킹 전략](02-chunking-strategies.md) — parent-child가 청크·문장 단위의 기반
- [03 메타데이터와 출처](03-metadata-provenance.md) — citation 필드의 원천
- [05 지식그래프 구축](05-knowledge-graph-construction.md) — 트리플 근거와 근거 문장 필수 규칙
- [07 신선도와 동기화](07-freshness-and-sync.md) — `valid_to`·톰스톤이 등급과 제외에 반영
- [10 접근 제어와 거버넌스](10-access-control-governance.md) — `acl_checked`
- [Part 3/04 생성과 근거 강제](../03-agent-workflow-rag/04-generation-grounding.md) — 검증기 계약
- [Part 4/06 HITL UI 패턴](../04-dynamic-ui/06-hitl-ui-patterns.md) — 렌더 계약
- [Part 3/01 RAG 아키텍처 패턴](../03-agent-workflow-rag/01-rag-architecture-patterns.md) — 질의 시 합성 vs 사전 컴파일(LLM Wiki), [Part 6/agent-memory](../06-trends/agent-memory.md) — 운영 지식용 메모리와 파생 근거
- [Part 5/04 거버넌스와 설명가능성](../05-cross-cutting/04-governance-explainability.md), [시나리오 A](../07-appendix-scenarios/01-enterprise-document-qa.md)

## 참고 문헌

- `[검증]` W3C PROV — 생성물·파생 관계(`wasDerivedFrom`)의 표준 어휘
- `[검증]` NLI(자연어 추론) 기반 사실 일관성 평가 — SummaC, 관련 함의 모델 연구 일반
- `[검증]` LLM 자가 보고 확신도의 비보정 — 보정(calibration) 연구 다수
- `[검증]` Renkhoff et al., IEEE TAI 2024 (arXiv 2401.03188) — 뉴로심볼릭 V&V. 부분별 검증은 부분적으로 충분, 결합 전체는 미성숙
- `[2차]` RAGAS의 faithfulness·context precision 지표 — 근거-답변 정합성 측정의 실무 기준. judge 파싱 실패가 NaN으로 조용히 제외되는 문제 보고
- `[2차]` 근거 하이라이트·citation UI의 실무 패턴 — 문서 QA 제품들의 공통 설계
- `[2차]` Agentic RAG — 검색이 루프 "앞"이 아니라 "안"으로
- `[2차]` Karpathy LLM Wiki(2026-04) 해설 — LLM이 쓴 위키 페이지는 파생 근거, 출처는 페이지 단위. 원 gist 미확인. 규모 한계는 `[검증]` arXiv 2604.11243
