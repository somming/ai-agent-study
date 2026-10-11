---
title: "관측성과 트레이싱"
part: 05-cross-cutting
stage: all
tags: [observability, tracing, opentelemetry, gen-ai-semconv, sampling, pii, dashboard, alerting]
maturity: 채택
status: review
updated: 2026-10-11
---

# 관측성과 트레이싱

> **범위**: 에이전트 트레이스 설계, OTel GenAI 컨벤션, 도구 비교. **운영** 관점 — 트레이스 스키마 표준화, 레이어 간 연결, 샘플링, PII, 알림·대시보드. 평가 수단과 골든셋 등 기초는 [Part 1/06](../01-foundations/06-evaluation-observability.md)을 전제로 하고 반복하지 않는다. RAG·에이전트 지표 정의는 [Part 3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md), UI 지표는 [Part 4/10](../04-dynamic-ui/10-evaluation-genui.md).

## 요약 (3줄)

1. 운영 관측성의 목표는 하나다. **사용자 턴 하나를 검색 ID → 판정·확신도 → 생성 citation → UI 렌더 이벤트 → 사용자 반응까지 한 trace_id로 끝까지 따라갈 수 있는가.** 레이어마다 따로 로그를 남기면 "어디서 틀렸나"를 세 시스템을 뒤져 맞춰야 한다.
2. OpenTelemetry(OTel) GenAI 시맨틱 컨벤션은 2026-10 현재 전 `gen_ai.*` 속성이 **Development** 등급이다. 그대로 쓰면 속성명 변경에 대시보드가 깨진다. **자체 스키마 + OTel 매핑 계층**이 운영 기본값이다.
3. 트레이스는 비용 계측([02](02-cost-latency-model-routing.md)), 감사 추적([04](04-governance-explainability.md)), 롤백 판단([05](05-llmops-agentops.md))의 공통 데이터 소스다. 세 용도가 같은 스팬 속성을 요구하므로 스키마를 한 번에 설계한다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | ① 트레이스 스키마(스팬 종류·필수 속성·ID 연결 규칙) ② OTel 매핑 계층 ③ 샘플링·보존·PII 정책 ④ 대시보드 지표 세트와 알림 규칙 ⑤ 레이어 간 전파 규약(프론트 → 백엔드 → 원격 에이전트) |
| **언제** (When) | **첫 배포 전**. 스키마 없이 로그를 쌓기 시작하면 나중에 ID를 이어 붙일 수 없다. 생략 가능한 것: 대시보드 정교화(실측 분포 후). 미룰 수 없는 것: trace_id 전파와 필수 속성 |
| **어떻게** (How) | ① 요청 1건의 스팬 트리를 그린다(세션 → 턴 → 에이전트 → 판정/검색/모델/툴/검증/UI) → ② 스팬별 필수 속성과 **연결 ID**(청크 ID, citation ID, 컴포넌트 인스턴스 ID, 승인 요청 ID)를 정한다 → ③ 자체 속성명으로 계측하고 OTel 익스포트 시 매핑 → ④ 본문(프롬프트·출력)은 opt-in·마스킹·별도 저장 → ⑤ 메타데이터 100%, 본문 샘플링, 실패·저확신·고위험 100% 보존 → ⑥ 지표·알림 정의 → ⑦ 레이어 간 정합성 검토 |
| **왜** (Why) | 에이전트 장애는 "500 에러"가 아니라 **조용한 품질 저하**로 온다(검색 recall 하락, 판정 보정 드리프트, citation 누락, 잘못된 컴포넌트 렌더). 트레이스 없이는 사용자 불만이 유일한 알림이고, 원인 분리가 불가능하다 |

## 문제와 맥락

전통 APM(Application Performance Monitoring)은 요청 → 서비스 → DB의 스팬을 추적한다. 에이전트는 두 가지가 다르다. 첫째, **스팬의 의미가 다르다** — 모델 호출 스팬은 토큰·모델 버전·프롬프트 버전, 검색 스팬은 반환 청크 ID와 점수, 판정 스팬은 선택과 확신도를 담아야 디버깅이 된다. 둘째, **트레이스가 세 레이어와 여러 프로세스를 넘는다** — 프론트엔드(UI 이벤트), 백엔드(에이전트 루프), 원격 MCP 서버·A2A 에이전트, 비동기 Tasks. 전파 규약이 없으면 끊긴다.

여기에 표준 문제가 겹친다. OTel GenAI 컨벤션은 `invoke_agent {name}` 스팬 아래 모델·툴 호출 자식 스팬 구조를 정의하고, 원격 에이전트는 CLIENT, 인프로세스는 INTERNAL 종류로, 본문 기록은 opt-in으로 두지만 `[2차]`, 모든 속성이 Development 등급이고 v1.42.0에서 별도 저장소(`semantic-conventions-genai`)로 분리되어 태그 릴리스가 없다 `[2차]`. 2026-06 "안정화" 주장 1건은 레지스트리에서 확인되지 않았다. 지금 OTel 속성명에 직접 대시보드를 묶으면 깨질 가능성이 높다.

## 핵심 개념

### 트레이스 스키마 — 스팬 종류와 필수 속성

| 스팬 | 레이어 | 필수 속성 | 연결 ID (다음 스팬으로 넘기는 것) |
|---|---|---|---|
| `turn` (= trace 루트) | 전부 | 요청 유형, 입력 길이, 총 지연, 총 비용, 최종 상태(성공·거부·폴백·사람 개입) | trace_id |
| `decision` (판정) | Reasoning | 판정 지점 ID, 수단(규칙·결정 모델·분류기·LLM), 질문 정의 버전, **선택 결과, 확신도(보정 확률)**, 임계값, 지연 | 선택 결과 → 라우팅 경로 |
| `retrieval` | Knowledge | 인덱스 버전, 쿼리(해시 또는 마스킹), 검색 방식, 후보 수, **반환 청크 ID 목록 + 점수**, 리랭크 여부, 지연 | 청크 ID → 생성 컨텍스트 |
| `model_call` (생성) | Reasoning | 모델 ID·버전, 프롬프트 버전, 입력·출력·캐시 토큰, 지연·TTFT, 종료 사유, 비용 | 생성된 **citation ID**(청크 ID 참조) → 검증·UI |
| `tool_call` | Reasoning | 툴 이름·스키마 버전, MCP 서버 ID·버전, 인자(마스킹), 결과 요약·크기, 성공 여부, 권한 게이트 결과, 지연 | 결과 참조 → 다음 판정 |
| `verification` | Reasoning | 검증기 종류(스키마·규칙·결정 모델·judge), 통과 여부, **거부 사유 코드**, 재생성 횟수 | 통과 여부 → 응답/폴백 |
| `ui_render` | Interaction | 컴포넌트 ID·카탈로그 버전, 선택 수단, 스키마 검증 결과, 렌더 성공, 렌더 지연, **참조한 citation ID** | 컴포넌트 인스턴스 ID → 사용자 이벤트 |
| `ui_event` | Interaction | 이벤트 종류(선택·승인·거절·수정), 대상 컴포넌트 인스턴스 ID, 서버 재검증 결과, 승인 요청 ID | 상태 머신 입력 → 다음 턴 |
| `remote_agent` (A2A) | Reasoning | 상대 에이전트 ID, 위임 범위, 종류 CLIENT, 응답 요약 | 상대 trace 링크 |

모든 스팬 공통: `trace_id, parent_span_id, 환경(dev/stg/prod), 릴리스 번들 버전([05](05-llmops-agentops.md)), 비용 태그([02](02-cost-latency-model-routing.md))`.

### 레이어 간 연결 — 한 트레이스로 끝까지

```
turn(trace_id=T)
 ├─ decision: route=complex, p=0.91 ─────────────────────┐
 ├─ retrieval: index=v12, chunks=[c17,c42,c88]           │
 ├─ model_call: model=M, prompt=v7, citations=[c17,c88] ◀┘
 ├─ verification: schema ok, grounding ok (c17,c88 ⊂ chunks)
 ├─ ui_render: component=ComparisonTable@v3, cites=[c17,c88], inst=U1
 └─ (다음 요청, 같은 T 전파) ui_event: approve, target=U1, revalidated=ok
```

연결 규칙 세 가지. ① **citation은 청크 ID를 그대로 쓴다**(Part 2의 citation 스키마를 재해석하지 않는다는 Part 4 원칙과 같다). 그래야 "인용된 근거가 실제로 검색된 것인가"를 트레이스에서 자동 검증할 수 있다. ② **UI 이벤트는 컴포넌트 인스턴스 ID를 들고 돌아온다.** 어떤 렌더에 대한 반응인지 모르면 과업 완료율을 못 센다. ③ **프론트엔드가 trace_id를 전파한다.** AG-UI류 이벤트 스트림에 trace_id를 실어 보내고, 사용자 이벤트 요청 헤더(W3C Trace Context)에 되돌려 보낸다. 원격 MCP 서버·A2A 에이전트에도 같은 헤더를 전파하고, MCP Tasks처럼 비동기로 완료되는 작업은 결과 수신 시 원래 trace에 **링크**로 잇는다.

### OTel 매핑 계층

```
계측 코드 ──(자체 속성: agent.decision.confidence 등)──▶ 내부 스키마
                                                          │
                                        매핑 테이블(버전 고정) ▼
                                     OTel 익스포터: gen_ai.* (Development)  ──▶ 벤더/OSS 백엔드
```

자체 속성명을 1차로 두고 OTel 속성으로 변환하는 어댑터를 한 곳에 둔다. 컨벤션이 바뀌면 매핑 테이블만 고친다. 매핑 대상 OTel 버전(저장소 커밋 또는 릴리스)을 설정값으로 고정하고 레지스트리를 직접 확인한다. 결정 모델 호출처럼 **출력 토큰이 없는 스팬**은 생성 모델 전제의 컨벤션에 딱 맞지 않으므로 자체 속성(선택·확신도·질문 정의 버전)이 주가 되고 OTel 쪽은 모델 ID·지연 정도만 매핑한다.

### PII 처리

- 프롬프트·출력·툴 인자·검색 쿼리 **본문은 opt-in**. 기본은 해시·길이·마스킹 요약만 스팬에
- 본문이 필요하면 별도 저장소(접근 제어·보존 기간 분리)에 두고 스팬에는 참조만 `[2차]`
- 마스킹은 계측 지점에서(수집 후가 아니라). 한국어 PII(주민번호·전화·주소 형식) 패턴 포함
- 사용자 ID는 익명화 ID로, 매핑 테이블은 분리 보관. 삭제 요청 시 트레이스·본문 저장소·캐시까지 전파([04](04-governance-explainability.md))

### 샘플링 전략

| 데이터 | 비율 | 이유 |
|---|---|---|
| 스팬 메타데이터(ID·버전·토큰·지연·판정·확신도) | 100% | 비용 계측·감사·지표의 원천. 양이 작다 |
| 본문(프롬프트·출력·툴 결과) | 샘플링(예: 수 %) + 조건부 100% | 저장 비용·PII |
| 실패·거부·폴백·저확신·사람 개입 턴 | 100% 보존(본문 포함, 마스킹) | 디버깅·골든셋 환류 대상 |
| 고위험 행동(외부 전송·결제·삭제) | 100% 보존 | 감사 추적 ([04](04-governance-explainability.md)) |

헤드 샘플링(요청 시작 시 결정)은 단순하지만 "실패한 턴을 100% 보존"을 못 한다. **테일 샘플링**(턴 종료 후 결과 보고 결정) 또는 "메타데이터 100% + 본문은 종료 시 조건부 보존" 구조가 필요하다.

### 대시보드 지표와 알림

| 지표군 | 지표 | 알림 조건 예 |
|---|---|---|
| 지연 | 단계별 p50/p95(판정·검색·리랭크·TTFT·총), 스트리밍 중단율 | p95가 예산([02](02-cost-latency-model-routing.md)) 초과 지속 |
| 오류·폴백 | 모델 API 오류율, 툴 실패율, MCP 서버별 오류율, 폴백 발동률, 상한 도달률 | 특정 MCP 서버 오류 급증, 폴백률 급증 |
| 비용 | 요청당 비용 분포, 기능·테넌트별 일 비용, 캐시 적중률, 모델별 호출 비중 | 요청당 평균 급증(루프), 적중률 급락(캐시 무효화) |
| 판정 품질 | 수단별 판정 분포, **확신도 분포·저확신 비율**, 임계값 미만 → 사람 경로 비율, 규칙 거부율 | 확신도 분포 이동(드리프트), 저확신 비율 급증 |
| 근거 품질 프록시 | citation 존재율, citation ⊂ 검색 결과 비율(자동 검증), 검색 0건 비율, 리랭크 후 상위 점수 분포 | citation 누락률 상승, 0건 비율 상승 |
| 검증 | 검증기별 통과·거부율, 거부 사유 코드 분포, 재생성 횟수 분포 | 특정 사유 급증(프롬프트·모델 변경 직후) |
| UI | 컴포넌트 선택 분포, 스키마 검증 실패율, 렌더 실패율, 사용자 수정·거절률, 승인 대기 시간 | 렌더 실패 급증(카탈로그 버전 불일치) |
| End-to-End | 과업 완료율, 턴 수, 사람 개입률, judge 샘플 점수 | 완료율 하락이 단계 지표보다 우선 |

알림은 **릴리스 번들 버전 변경 직후**에 민감도를 높인다(카나리 비교, [05](05-llmops-agentops.md)).

### 1/07 정의와의 연결 — 이 페이지는 어디에 놓이는가

[Part 1/07](../01-foundations/07-operations-vocabulary.md)의 통제 세 층에서 이 페이지는 **탐지(detective) 층의 기반 데이터**다. 관측 지표·알림·드리프트 모니터가 전부 여기서 정의한 스팬 속성에서 나온다. 1/07은 **관측성과 감사 추적을 구분**한다 — 같은 트레이스에서 나오되 목적(지금 어떻게 동작하나 vs 왜 그 결정이 났나), 소비자(엔지니어 vs 감사자), 보존(짧고 샘플링 가능 vs 길고 샘플링 불가·변조 방지)이 다르다. 이 페이지는 전자를 설계하고, 감사 레코드는 트레이스의 **부분집합을 별도 저장소·보존 정책으로 분리**해 [04](04-governance-explainability.md)가 설계한다. 위험 등급과의 접점은 샘플링 표에 있다: 고위험 행동(등급 3 이상)과 사람 개입 턴은 100% 보존 — 감사 요건이 샘플링 정책을 덮어쓴다. 거버넌스 구성 요소로는 "감사"의 원천이고, 1/07 체크리스트의 "SLI 후보와 SLO 초안"은 이 페이지의 지표 세트로 측정된다. 레이어 E2E SLI와 단계 SLI의 관계는 [06 공통 SLO 초안](06-operational-design-matrix.md)에 있다.

### 변화 대응 — 이 횡단 관심사가 바뀔 때

관측 체계에서 바뀌는 것은 두 속도로 나뉜다. **빠르게 바뀌는 것**은 외부 표준(OTel GenAI 컨벤션 — 2026-10 현재 전부 Development 등급, 속성명 변경 가능)과 벤더 SDK·프레임워크의 자동 계측 포맷이다. **느리게 바뀌어야 하는 것**은 자체 트레이스 스키마이고, **거의 바뀌지 않아야 하는 것**은 감사 레코드 스키마다. 세 속도를 한 스키마로 묶으면 OTel 속성명 하나가 바뀔 때 감사 레코드 파서가 깨진다. 1/07 "변화 유형별 흡수 지점" 표의 **프로토콜·SDK** 행이 이 페이지의 주된 변화 축이다.

| 변화 | 바뀌는 것 | 안정적으로 유지할 것 | 흡수 방식 | 전파 범위 |
|---|---|---|---|---|
| OTel GenAI 속성명·구조 변경 | 익스포터가 내보내는 `gen_ai.*` 이름 | **자체 트레이스 스키마**(`agent.decision.confidence` 등 내부 속성명) | **매핑 계층**의 테이블만 교체. 대시보드·알림은 내부 이름만 참조하므로 무영향. 매핑 대상 OTel 버전을 설정으로 고정 | 매핑 테이블 + 백엔드 익스포트 |
| 트레이스 스키마 자체 변경(새 스팬 종류, 필수 속성 추가) | 내부 스키마 | 스팬 종류 9개의 **연결 ID 규약**(청크 ID → citation ID → 컴포넌트 인스턴스 ID → 승인 요청 ID) | **트레이스 스키마 버전**: 속성 추가는 마이너(소비자는 tolerant reader), 삭제·의미 변경은 메이저 + 집계 쿼리 재작성 + 단절 마커. 트래젝토리 추출 어댑터([Part 3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md))와 평가 파이프라인이 소비자 | 집계·대시보드·평가 어댑터 |
| 감사 레코드 스키마 변경 | 감사 필드 | 재구성 다섯 요소(누가·근거·판정·확신도·결과·승인) | **별도 버전, 느리게**. 추가만 허용, 삭제 금지(과거 레코드 재해석 불가). 변경은 [04](04-governance-explainability.md) 변경 관리 승인 | 감사 저장소 파서 |
| 벤더 SDK·프레임워크 교체(자동 계측 포맷 변경) | 계측 지점의 원시 스팬 | 자체 스키마, trace_id 전파 규약 | 수집 측 **변환 어댑터** 교체. 프레임워크 트레이스 → 자체 스키마 변환은 [Part 3/14](../03-agent-workflow-rag/14-frameworks-landscape.md)의 어댑터 계층 | 어댑터만 |
| 새 스팬 원천(새 레이어 컴포넌트, 원격 A2A 에이전트, MCP Tasks) | 스팬 트리 | 전파 규약(W3C Trace Context 헤더, 비동기 결과는 링크) | 스팬 종류 추가(마이너) + 전파 규약 적용 확인 | 트리 + 상관 유실률 SLI |
| 샘플링·보존·PII 정책 변경(규제, 비용) | 비율·보존 기간·마스킹 패턴 | 메타데이터 100% / 실패·고위험·사람 개입 100% 원칙 | **정책을 설정으로**(비율·조건·보존 계층·패턴 사전) + 시행일. 수집기 설정 교체, 코드 배포 없음 | 수집기, 본문 저장소 |
| 지표 정의 변경(분모·분자·원인 코드 enum) | 대시보드·SLO·알림 | 지표 정의 문서 | 지표 버전 + 과거 재계산 또는 단절 마커([Part 4/10](../04-dynamic-ui/10-evaluation-genui.md)) | 모든 소비 대시보드 |

코드 밖 설정으로 둘 것: OTel 매핑 테이블(버전 고정), 샘플링 규칙·보존 계층, PII 패턴 사전(한국어 형식 포함), 알림 임계값, 지표 정의·원인 코드 enum. 포트·어댑터 뒤에 둘 외부 의존: OTel 익스포터, 벤더 관측 SDK, 프레임워크 자동 계측, 트레이스 백엔드, 수집기. 시점 재현: 모든 스팬에 **트레이스 스키마 버전 + 릴리스 번들 버전**이 있어야 과거 트레이스를 현재 집계 쿼리로 읽을 때 어떤 매핑을 적용할지 안다.

판단 기준: **감사 레코드 스키마는 트레이스 스키마보다 느리게, 트레이스 스키마는 OTel 매핑보다 느리게** 바꾼다. 세 속도가 역전되면(감사 필드를 OTel 속성명에 직접 묶는 등) 외부 표준의 변경이 규제 대응 데이터를 깨뜨린다. 2026-10 시점에서 OTel GenAI 속성은 아직 안정 등급이 아니므로 `[2차]`, 이 분리는 선택이 아니라 전제다.

### 단계별 관측 설계 색인 — 규약은 1/07, 적용은 각 단계 페이지

이 페이지는 **운영 도구와 횡단 정책**(스키마 표준화·매핑 계층·PII·샘플링·대시보드)을 다룬다. 어느 스팬을 어떤 이름으로, 어떤 속성과 지표로 내는지의 **규약**은 [Part 1/07 관측성 설계 요소](../01-foundations/07-operations-vocabulary.md)가 정의하고(스팬 계층 `session → turn → invoke_agent → <stage>.<operation>`, 공통 속성 `agent.*`와 버전 키 5종, 지표 3종, 알림→런북, 대시보드 3장), **단계마다 무엇을 기록·측정·알림하는지**는 각 단계 페이지의 "운영 설계 → 관측성" 소절이 답한다. 아래가 그 색인이다.

| 레이어 | 단계 페이지 (운영 설계 → 관측성 소절) | 대표 스팬 |
|---|---|---|
| Knowledge | [2/01](../02-data-pipeline/01-source-ingestion.md) · [2/02](../02-data-pipeline/02-chunking-strategies.md) · [2/03](../02-data-pipeline/03-metadata-provenance.md) · [2/04](../02-data-pipeline/04-embedding-indexing.md) · [2/05](../02-data-pipeline/05-knowledge-graph-construction.md) · [2/06](../02-data-pipeline/06-ontology-driven-modeling.md) · [2/07](../02-data-pipeline/07-freshness-and-sync.md) · [2/08](../02-data-pipeline/08-grounding-data-design.md) · [2/09](../02-data-pipeline/09-ai-inside-the-pipeline.md) · [2/10](../02-data-pipeline/10-access-control-governance.md) | `ingest.*`, `index.*`, `kg.*`, `sync.*`, `acl.resolve`, `evidence.assemble` (문서 단위 트레이스) |
| Reasoning / Action | [3/01](../03-agent-workflow-rag/01-rag-architecture-patterns.md) · [3/02](../03-agent-workflow-rag/02-query-understanding.md) · [3/03](../03-agent-workflow-rag/03-retrieval-reranking.md) · [3/04](../03-agent-workflow-rag/04-generation-grounding.md) · [3/05](../03-agent-workflow-rag/05-workflow-vs-agent.md) · [3/06](../03-agent-workflow-rag/06-agent-design-patterns.md) · [3/07](../03-agent-workflow-rag/07-multi-agent-orchestration.md) · [3/08](../03-agent-workflow-rag/08-state-memory-context.md) · [3/09](../03-agent-workflow-rag/09-tools-and-mcp.md) · [3/10](../03-agent-workflow-rag/10-neuro-symbolic-reasoning.md) · [3/11](../03-agent-workflow-rag/11-decision-layer.md) · [3/12](../03-agent-workflow-rag/12-guardrails-hitl.md) · [3/13](../03-agent-workflow-rag/13-evaluation-rag-agents.md) · [3/14](../03-agent-workflow-rag/14-frameworks-landscape.md) | `invoke_agent`, `routing.decide`, `retrieval.*`, `generation.*`, `tool.invoke`, `verify.*`, `decision.decide`, `gate.evaluate`, `approval.request/resolve`, `memory.*`, `agent.step/delegate/handoff` |
| Interaction | [4/01](../04-dynamic-ui/01-generative-ui-spectrum.md) · [4/02](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) · [4/03](../04-dynamic-ui/03-component-registry-schema.md) · [4/04](../04-dynamic-ui/04-streaming-state-sync.md) · [4/05](../04-dynamic-ui/05-intent-to-ui-mapping.md) · [4/06](../04-dynamic-ui/06-hitl-ui-patterns.md) · [4/07](../04-dynamic-ui/07-frontend-integration.md) · [4/08](../04-dynamic-ui/08-security-sandboxing.md) · [4/09](../04-dynamic-ui/09-ux-accessibility.md) · [4/10](../04-dynamic-ui/10-evaluation-genui.md) | `ui.map`, `ui.validate`, `ui.stream`, `ui.render`, `ui.event`, `ui.event.verify`, `approval.display/decide` (프론트엔드 보고, 같은 `request_id`) |
| 기반·종합 | [1/01](../01-foundations/01-llm-fundamentals.md) · [1/05](../01-foundations/05-agentic-protocols.md) · [1/06](../01-foundations/06-evaluation-observability.md) · [5/06 종합 매트릭스](06-operational-design-matrix.md) · 시나리오 [A](../07-appendix-scenarios/01-enterprise-document-qa.md)·[B](../07-appendix-scenarios/02-commerce-assistant-genui.md)·[C](../07-appendix-scenarios/03-ops-incident-agent.md)의 9.5절 | 모델 호출 `gen_ai.*` 매핑, `eval.run`, 프로토콜 속성 |

읽는 법: 어떤 장애가 탐지되지 않았다면 ① 그 단계 페이지의 운영 안정성 표에서 "탐지 신호"를 찾고 ② 같은 페이지 관측성 소절에서 그 신호를 내는 스팬·지표가 있는지 확인한다. 없으면 그 페이지의 결함이고, 있는데 안 보였다면 이 페이지의 수집·매핑·샘플링 문제다.

## 아키텍처 / 파이프라인

```
프론트(AG-UI 이벤트 + trace_id) ─▶ 백엔드 에이전트 루프(자체 스키마 계측) ─▶ 원격 MCP/A2A(헤더 전파)
                                            │
                                   매핑 계층(버전 고정) ─▶ OTel 익스포트 ─▶ 수집기(테일 샘플링·마스킹)
                                            │                                  ├─▶ 트레이스 백엔드(벤더 SDK UI 또는 OSS)
                                            │                                  ├─▶ 지표·알림
                                            │                                  ├─▶ 비용 집계 ([02])
                                            │                                  ├─▶ 감사 저장소(불변, 장기) ([04])
                                            └─ 본문 저장소(opt-in, 접근 제어)   └─▶ 실패 샘플 → 골든셋 환류 ([Part 1/06])
```

## 기술 비교표 — 이 단계에서 쓸 수 있는 선택지

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| 벤더 관측 SDK (Langfuse, LangSmith, Arize 등) 직접 계측 | 소규모 팀, 즉시 UI·평가 통합 필요, 프레임워크 자동 계측 활용 | 벤더 종속 회피, 기존 OTel 인프라와 통합, 결정 모델·UI 스팬처럼 벤더 스키마 밖 데이터 | 하~중 | 안정 | `[벤더]` |
| OTel GenAI 컨벤션 직접 계측 | 기존 OTel 인프라 보유, 벤더 중립 | 속성 안정성 필요(전부 Development), 대시보드를 속성명에 직접 묶는 운영 | 중 | 신생 | `[2차]` |
| **자체 스키마 + OTel 매핑 계층** | 운영 시스템 일반. 결정·검색·UI 스팬이 컨벤션에 없는 속성을 필요로 함 | 프로토타입(과잉), 계측 코드 소유권이 흩어진 조직(매핑 한 곳 유지 어려움) | 중 | 안정(패턴) | `[2차]` |
| 프레임워크 내장 트레이싱 (LangGraph, ADK 토큰 추적, Claude Agent SDK hooks 등) | 단일 프레임워크, 빠른 시작 | 프레임워크 밖 구간(프론트 UI, 원격 에이전트, 벡터DB) 연결, 프레임워크 교체 | 하 | 안정 | `[벤더]` `[2차]` |
| APM 벤더의 LLM 확장 (Datadog LLM Observability 등) | 이미 해당 APM 사용, 인프라 지표와 한 화면 | APM 미사용 조직(도입 비용), RAG 평가 자동화는 Ragas 연동·샘플링 권고 수준 | 중~상 (라이선스) | 안정 | `[벤더]` |
| 테일 샘플링 수집기 (OTel Collector 등) | 실패·저확신·고위험 100% 보존 필요 | 트래픽 매우 작음(헤드로 충분), 수집기 운영 역량 없음 | 중 | 안정 | `[검증]` OTel 표준 구성 요소 |

### 선택 가이드

- **아직 아무것도 없다면** → 벤더 SDK로 시작하되 OTel 익스포트를 켜고, 자체 연결 ID(청크·citation·컴포넌트 인스턴스)는 첫날부터 속성에 넣는다. 이유: 전환 비용 최소화, ID는 나중에 못 붙인다
- **기존 OTel 인프라가 있다면** → 자체 스키마 + 매핑 계층 + 테일 샘플링 수집기. 이유: 컨벤션 변경을 한 곳에서 흡수, 실패 100% 보존
- **Generative UI를 운영한다면** → 프론트 trace_id 전파와 `ui_render`/`ui_event` 스팬을 백엔드 스팬과 같은 백엔드에. 이유: 과업 완료율·수정률은 UI 이벤트 없이는 측정 불가
- **멀티 에이전트·MCP Tasks가 있다면** → 헤더 전파 + 비동기 결과의 trace 링크 규약을 먼저. 이유: 전파 없이는 트레이스가 에이전트 경계에서 끊긴다
- **판단이 어려우면** → 기본값: 메타데이터 100% + 본문 opt-in + 실패·저확신·고위험 100% 보존 + 벤더 SDK(OTel 익스포트 on). 이유: 비용·PII·디버깅 가능성의 균형

## 설계 시 고려사항 (체크리스트)

### 레이어별

- [ ] 스팬 트리와 필수 속성이 문서화되어 있고, 계측 코드가 그 스키마를 따르는가
- [ ] 모델·프롬프트·인덱스·툴 스키마·컴포넌트 카탈로그·결정 질문 정의 **버전**이 해당 스팬에 기록되는가 ([05](05-llmops-agentops.md))
- [ ] OTel 속성명에 대시보드·쿼리가 직접 묶여 있지 않고 매핑 계층을 거치는가. 매핑 대상 컨벤션 버전이 고정되어 있는가
- [ ] (Reasoning) 판정 스팬에 수단·선택·확신도·임계값이 있는가. 확신도 분포 드리프트 알림이 있는가
- [ ] (Reasoning) 검증 스팬의 거부 사유가 **코드화**되어 집계 가능한가(자유 텍스트 아님)
- [ ] (Reasoning) 원격 MCP 서버·A2A 에이전트에 trace 헤더가 전파되고, 비동기 결과가 원 trace에 링크되는가
- [ ] (Interaction) 프론트가 trace_id를 받고 사용자 이벤트에 되돌려 보내는가. `ui_event`가 컴포넌트 인스턴스 ID를 포함하는가
- [ ] 본문 기록이 opt-in이고, 마스킹이 계측 지점에서 이뤄지며, 한국어 PII 패턴을 포함하는가
- [ ] 실패·거부·폴백·저확신·고위험 턴이 100% 보존되는가(테일 샘플링 또는 조건부 보존)

### 레이어 간 정합성 검토

- [ ] 한 턴의 trace를 열었을 때 검색 청크 ID → 생성 citation → UI 컴포넌트 → 사용자 이벤트가 **끊김 없이** 보이는가. 끊기는 지점이 어느 레이어 경계인가
- [ ] Part 2의 citation 스키마 ID와 Part 3의 생성 citation, Part 4의 citation 렌더가 **같은 ID 체계**를 쓰는가(레이어마다 재매핑하면 자동 검증 불가)
- [ ] Part 3는 벤더 SDK로, Part 4는 프론트 분석 도구(제품 분석)로 따로 기록하고 있지 않은가 — 과업 완료율을 두 시스템을 조인해야 계산하는가
- [ ] Part 3의 판정 확신도는 기록하는데 Part 4의 UI 선택 판정 확신도는 기록하지 않는가(판정 수단마다 기록 깊이가 다름)
- [ ] 보안 거부([01](01-security-prompt-injection.md))·비용 상한([02](02-cost-latency-model-routing.md))·검증 거부가 모두 같은 거부 사유 코드 체계로 기록되는가
- [ ] 샘플링이 레이어별로 따로 결정되어 "검색 스팬은 있는데 같은 턴의 UI 스팬은 샘플링돼 없음"이 생기지 않는가(샘플링 결정은 턴 단위)

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| OTel GenAI 컨벤션 Development 상태 | 속성명 변경으로 대시보드·쿼리 깨짐 | 매핑 계층, 컨벤션 버전 고정, 레지스트리 직접 확인 |
| 결정 모델·UI 스팬이 컨벤션에 없음 | 벤더 UI에서 "모델 호출"로만 보이거나 누락 | 자체 속성 주도, 벤더 UI는 보조 |
| 본문 기록의 PII | 유출·규제 위반 | opt-in, 계측 지점 마스킹, 별도 저장소, 보존 기간 |
| 트레이스 저장 비용 | 본문 100% 기록 시 서빙 비용에 근접 | 메타데이터 100% / 본문 조건부, 테일 샘플링 |

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| OTel GenAI 컨벤션 저장소 분리 | v1.42.0에서 별도 저장소, 태그 릴리스 없음. `invoke_agent` 스팬 구조 포함. 운영에서는 매핑 계층 전제 | OTel 사용 | `[2차]` |
| AG-UI·A2UI 이벤트 스트림 | 타입드 이벤트(텍스트 델타·툴 호출 시작/종료·상태 패치)가 그대로 `ui_render`/`ui_event` 스팬의 원천. trace_id 전파 필드 필요 | GenUI 채택 | `[2차]` `[벤더]` |

## 관련 페이지

- [Part 1/06 평가와 관측성 기초](../01-foundations/06-evaluation-observability.md) — 전제. 평가 수단·골든셋·judge 한계
- [Part 3/13 RAG·에이전트 평가](../03-agent-workflow-rag/13-evaluation-rag-agents.md), [Part 4/10 Generative UI 평가](../04-dynamic-ui/10-evaluation-genui.md)
- [Part 2/08 근거 데이터 설계](../02-data-pipeline/08-grounding-data-design.md) — citation ID 체계, [Part 4/04 스트리밍과 상태 동기화](../04-dynamic-ui/04-streaming-state-sync.md) — 이벤트 전파
- [02 비용·지연](02-cost-latency-model-routing.md), [04 거버넌스](04-governance-explainability.md), [05 LLMOps](05-llmops-agentops.md) — 같은 트레이스를 소비하는 세 페이지

## 참고 문헌

- `[2차]` OTel GenAI 컨벤션 상태 — [연구 노트](../_meta/research-notes-2026-10.md) "평가·관측성" 절: 전 `gen_ai.*` Development, v1.42.0 저장소 분리·태그 릴리스 없음(2026-09 Dash0), 2026-06 안정화 주장은 레지스트리 미확인
- `[2차]` 에이전트 스팬 구조 — `invoke_agent {name}` 하위 모델·툴 스팬, 원격 CLIENT / 인프로세스 INTERNAL, 본문 opt-in
- `[벤더]` Datadog LLM Observability — Ragas 평가 자동 트레이싱, 샘플링 권고
- `[검증]` W3C Trace Context — 분산 트레이스 전파 헤더 표준. OTel Collector 테일 샘플링은 OTel 공식 구성 요소

_검증 수준 표기: `[검증]` `[벤더]` `[2차]` `[실측]` — 규칙은 [작성 가이드](../_meta/writing-guide.md) 참조_
