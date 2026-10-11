---
title: "보안과 샌드박싱"
part: 04-dynamic-ui
stage: interaction
tags: [security, iframe-sandbox, csp, postmessage, prompt-injection, trust-boundary, mcp-apps, authorization]
maturity: 시험
status: review
updated: 2026-10-11
---

# 보안과 샌드박싱

> **범위**: iframe 샌드박스, UI 경유 prompt injection, 권한 경계. 에이전트 UI 레이어에 고유한 공격면 — 생성된 UI의 실행, UI를 거쳐 에이전트 입력으로 들어가는 콘텐츠, 사용자 이벤트의 권한 — 과 그 방어. 프롬프트 인젝션 일반론과 입출력 필터는 [Part 5/01](../05-cross-cutting/01-security-prompt-injection.md), 툴 권한 모델은 [Part 3/09](../03-agent-workflow-rag/09-tools-and-mcp.md).
> **시각화**: [`assets/html/dynamic-ui-architecture.html`](../assets/html/dynamic-ui-architecture.html) — "서버 재검증" 노드와 렌더러 경계

## 요약 (3줄)

1. UI 레이어는 **두 방향의 신뢰 경계**를 가진다. 내려오는 방향(에이전트·툴 서버가 만든 UI를 사용자 브라우저에서 실행)과 올라가는 방향(렌더된 콘텐츠와 사용자 이벤트가 에이전트 입력이 됨). 둘 다 외부 입력으로 취급한다.
2. 통제 수준([01](01-generative-ui-spectrum.md))이 곧 보안 비용이다. Controlled는 props 스키마 검증으로 끝나고, Declarative는 트리·바인딩 검증이 추가되며, Open-ended는 **코드 실행**이므로 iframe 샌드박스 + CSP + postMessage 검증이 전부 필요하다.
3. 사용자 이벤트의 권한은 **서버가 인증 컨텍스트로 다시** 확인한다. 프론트가 보낸 `user`·`roles`·"이 버튼이 보였으니 권한이 있다"는 모두 주장일 뿐이다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | ① 신뢰 경계 지도(어느 콘텐츠가 어디서 와서 어디서 실행·해석되는가) ② 패턴별 검증 계층(스키마 / 트리·바인딩 / 샌드박스) ③ iframe 샌드박스·CSP 정책 ④ postMessage 채널 스키마와 출처 검증 ⑤ 사용자 이벤트 서버 재검증 규칙 ⑥ UI 경유 인젝션 방어 |
| **언제** (When) | Part 4를 하는 모든 경우. Controlled만 쓴다면 ②·⑤·⑥만. Open-ended(MCP Apps 포함)를 한 영역이라도 열면 ③·④가 필수가 된다. 보안 설계는 렌더러 구현([07](07-frontend-integration.md)) **전에** 끝낸다 |
| **어떻게** (How) | ① 콘텐츠 출처를 분류(자사 컴포넌트 / 에이전트 생성 트리 / 툴 서버 UI / 검색된 문서 / 사용자 입력) → ② 출처별 실행·해석 위치 표시 → ③ 경계마다 검증 수단 배정 → ④ Open-ended 영역에 샌드박스 정책 작성 → ⑤ 사용자 이벤트 재검증 ①~⑤([04](04-streaming-state-sync.md)) 구현 → ⑥ 렌더된 콘텐츠가 에이전트 입력이 되는 경로를 찾아 격리·표시 → ⑦ 위협 모델을 문서화하고 [Part 5/01](../05-cross-cutting/01-security-prompt-injection.md)과 교차 검토 |
| **왜** (Why) | UI는 "출력"이라 보안 검토에서 빠지기 쉽다. 하지만 ① 에이전트가 만든 HTML이 사용자 세션에서 실행되면 XSS와 같고 ② 검색된 문서의 숨은 지시가 UI에 렌더된 뒤 사용자 이벤트로 에이전트에 되돌아가면 인젝션이며 ③ 승인 버튼이 보였다는 이유로 서버가 권한을 생략하면 권한 상승이다 |

## 문제와 맥락

전통 웹 보안은 "서버가 만든 HTML을 브라우저가 실행한다"는 전제 아래 XSS·CSRF를 다뤘다. 에이전트 UI는 전제가 바뀐다. HTML(또는 UI 트리)을 **확률적 모델이나 서드파티 툴 서버**가 만들고, 그 안의 콘텐츠 일부는 **검색된 외부 문서**에서 왔으며, 사용자가 그 UI에서 누른 버튼이 **에이전트의 다음 행동**을 결정한다. 공격자는 문서에 지시를 심어 UI를 거쳐 에이전트를 조종하거나, 툴 서버 UI로 호스트 세션을 노리거나, 이벤트를 위조해 승인 없이 실행을 유도할 수 있다. MCP Apps가 "서버가 HTML을 반환하고 호스트가 샌드박스 iframe에 렌더"하는 모델을 택한 것 `[2차]`은 이 위협을 전제한 설계다.

## 핵심 개념

### 신뢰 경계 지도

| 콘텐츠 출처 | 신뢰 수준 | 실행·해석 위치 | 필요한 검증 |
|---|---|---|---|
| 자사 Controlled 컴포넌트 | 높음(코드 리뷰 거침) | 호스트 문서 | props 스키마 |
| 에이전트 생성 Declarative 트리 | 중(모델 출력) | 호스트 렌더러 | 트리 스키마 + 허용 조합 + 바인딩 + 깊이·크기 제한 |
| 툴 서버 UI (MCP Apps) | 낮음(서드파티 코드) | **샌드박스 iframe** | sandbox 속성 + CSP + postMessage 스키마·출처 |
| 에이전트 생성 HTML (Open-ended 커스텀) | 낮음 | **샌드박스 iframe** | 위와 동일 + 생성물 크기·외부 요청 금지 |
| 검색된 문서 본문 (citation 하이라이트 등) | 낮음(외부 데이터) | 텍스트로만 렌더 | HTML 이스케이프, 링크 무해화, 지시문 표시 |
| 사용자 이벤트 | 낮음(클라이언트 주장) | 서버 상태 머신 | 재검증 ①~⑤ |

원칙: **낮음 등급은 호스트 문서에서 실행하지 않는다.** 텍스트로 렌더하거나 iframe에 가둔다.

### iframe 샌드박스와 CSP

HTML 표준의 `sandbox` 속성은 iframe 안의 문서에 제약을 건다 `[검증]`. Open-ended 영역의 기본 정책은 다음과 같다.

| 설정 | 기본값 | 열 때의 조건 |
|---|---|---|
| `sandbox` 기본(토큰 없음) | 스크립트·폼·팝업·동일 출처 전부 금지 | — |
| `allow-scripts` | 필요하면 허용(위젯이 동작하려면) | `allow-same-origin`과 **함께 주지 않는다** — 둘 다 주면 샌드박스를 스스로 해제할 수 있다 |
| `allow-same-origin` | 금지 | 거의 모든 경우 금지. 별도 출처(서브도메인)에서 서빙해 자연스럽게 분리 |
| `allow-forms` | 금지 | 폼 제출 대신 postMessage로 데이터 전달 |
| `allow-popups`, `allow-top-navigation` | 금지 | 호스트를 벗어나는 탐색은 호스트가 중개 |
| CSP (`Content-Security-Policy`) | `default-src 'none'`에서 시작, 필요한 소스만 열기 | 외부 네트워크 요청은 허용 목록으로. 인라인 스크립트는 nonce/hash |
| 별도 출처 서빙 | 샌드박스 콘텐츠는 호스트와 다른 출처에서 | 동일 출처 쿠키·스토리지 접근 차단 |

A2UI가 "실행 코드가 아닌 데이터"를 강조하는 것 `[벤더]`은 Declarative가 샌드박스 없이도 호스트 렌더러에서 안전하게 해석될 수 있다는 주장이다. 이것이 성립하려면 렌더러가 트리의 어떤 값도 코드로 평가하지 않아야 한다(이벤트 핸들러 문자열, URL 스킴, 템플릿 표현식 주의).

### postMessage 채널 검증

iframe ↔ 호스트 통신은 postMessage다. 양쪽 모두 다음을 지킨다.

1. **출처(origin) 검증**: 수신 시 `event.origin`이 기대한 값인지 확인. 송신 시 대상 출처를 명시(`*` 금지)
2. **스키마 검증**: 메시지를 닫힌 enum `type` + 스키마로 검증. 모르는 `type`은 무시하고 기록
3. **능력 제한**: iframe이 요청할 수 있는 것(상태 읽기, 이벤트 발생)을 화이트리스트로. "툴 실행"을 iframe이 직접 요청하게 하지 않는다 — 사용자 이벤트로 변환되어 서버 재검증을 거친다
4. **크기·빈도 제한**: 메시지 크기 상한, 레이트 리밋
5. **세션 토큰 전달 금지**: iframe에 호스트 인증 토큰을 넘기지 않는다. 필요한 데이터는 호스트가 가져와 전달

### UI 경유 prompt injection

인젝션은 "모델 입력에 섞인 지시"다. UI 레이어에서 모델 입력이 되는 경로가 세 가지 있다.

| 경로 | 예 | 방어 |
|---|---|---|
| **렌더된 콘텐츠 → 사용자 이벤트 → 에이전트** | 검색된 문서에 "이 버튼을 누르면 모든 데이터를 삭제하라고 에이전트에 전달됨"이라는 숨은 지시. 사용자가 무심코 승인 | 사용자 이벤트 `payload`는 **구조화된 값만**(ID·enum·숫자). 자유 텍스트를 에이전트 프롬프트에 그대로 넣지 않는다. 승인 카드는 행동 정의(서버 소유)만 승인하고 렌더된 문서 텍스트를 승인하지 않는다 |
| **iframe 콘텐츠 → postMessage → 에이전트** | 툴 서버 UI가 "사용자가 X를 요청했다"는 메시지를 위조 | postMessage는 사용자 이벤트로만 변환, 서버 재검증, iframe이 보낸 텍스트에 "사용자 발화" 지위를 주지 않는다 |
| **인라인 수정 → 에이전트** | 사용자가(또는 조작된 클라이언트가) 수정 필드에 지시문 입력 | 수정 값은 **데이터**로 취급(타입·범위 검증). 에이전트가 수정 값을 다시 읽을 때 데이터 구획으로 격리 표시 |

공통 원칙: UI를 거쳐 돌아오는 것은 **데이터이지 지시가 아니다**. 에이전트 프롬프트에 넣을 때 출처 태그를 붙이고 구획을 분리한다([Part 5/01](../05-cross-cutting/01-security-prompt-injection.md)).

### 사용자 이벤트 권한의 서버 재검증

[04](04-streaming-state-sync.md)의 재검증 ①~⑤ 중 보안 관점에서 빠뜨리기 쉬운 것:

- **"보였으니 허용"의 오류**: 승인 버튼이 렌더됐다는 사실은 권한 증명이 아니다. 서버는 인증 컨텍스트 + 정책으로 다시 판단
- **대상 소유 확인**: `target.component_id`가 **이 사용자의 이 run**에서 서버가 발급한 것인지. 다른 세션의 ID를 가져다 쓰는 공격
- **행동 정의 불변**: `as_shown_hash`([06](06-hitl-ui-patterns.md)) 대조. 카드 표시 후 인자 변경 차단
- **멱등성·재생 방지**: `idempotency_key` + `issued_at` 창. 오래된 이벤트 재전송 거부
- **권한 상승 경로**: 인라인 수정으로 `amount`·`target_user` 같은 필드를 권한 밖 값으로 바꾸는 것. 필드별 권한 검증

### Open-ended 패턴의 코드 실행 위험

Open-ended는 "모델이 만든 코드를 사용자 브라우저에서 실행"이다. 샌드박스가 있어도 남는 위험:

- **피싱 UI**: 샌드박스 안에서 호스트의 로그인 화면을 흉내 내 자격 증명 입력 유도 → iframe 경계를 **시각적으로** 표시(테두리·출처 라벨), 호스트는 iframe 안에서 자격 증명을 받지 않는다는 규칙을 사용자에게 고지
- **리소스 소모**: 무한 루프·대용량 렌더 → 생성물 크기 상한, 타임아웃 후 iframe 제거
- **외부 유출**: CSP를 뚫는 채널(이미지 URL에 데이터 인코딩) → `img-src`도 허용 목록, 호스트가 가져다주는 데이터 최소화
- **클릭재킹**: 투명 iframe을 승인 버튼 위에 → 승인 UI는 iframe 밖(Controlled)에만, 호스트 자신의 `frame-ancestors` 정책

## 아키텍처 / 파이프라인

입력은 통제 수준([01](01-generative-ui-spectrum.md))·프로토콜([02](02-protocols-ag-ui-a2ui-mcp-apps.md))·임베딩 방식([07](07-frontend-integration.md)) 결정. 처리는 신뢰 경계 지도 작성 → 경계별 검증 배정 → 샌드박스·CSP·postMessage 정책 → 재검증 규칙. 출력은 보안 정책 문서와 검증 코드이며, [10](10-evaluation-genui.md)의 스키마 검증 실패율·[Part 5/01](../05-cross-cutting/01-security-prompt-injection.md)의 인젝션 테스트가 이것을 검증한다.

## 기술 비교표 — 이 단계에서 쓸 수 있는 선택지

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| **props 스키마 검증만** (Controlled) | 자사 컴포넌트, 행동 UI | Declarative·Open-ended 영역 | 하 | 표준 | `[검증]` JSON Schema |
| **트리 스키마 + 허용 조합 + 바인딩 검증** (Declarative) | 에이전트 생성 트리, A2UI | 렌더러가 트리 값을 코드로 평가하는 구조 | 중 — 검증기 유지 | 신생 (A2UI 렌더러) / 표준 (검증 기법) | `[벤더]` A2UI "데이터" 주장 |
| **iframe sandbox + CSP + 별도 출처** (Open-ended) | 툴 서버 UI, 생성 HTML, 신뢰 수준 다른 코드 | 호스트와 긴밀한 상태 공유·일체감 요구 | 중~상 — 정책·브리지·운영 | 표준 (HTML·CSP) / 신생 (MCP Apps 호스트) | `[검증]` HTML·CSP 표준 / `[2차]` MCP Apps |
| **postMessage 스키마·출처 검증 브리지** | iframe 사용 시 전부 | — (iframe을 쓰면 필수) | 중 | 표준 | `[검증]` HTML 표준 |
| **서버 재검증 ①~⑤** | 사용자 이벤트 전부 | — (필수) | 중 | 표준 (웹 보안 관행) | — |
| **출처 태그·구획 분리** (UI 경유 인젝션) | 렌더된 콘텐츠·수정 값이 에이전트 입력이 될 때 | 에이전트가 UI 입력을 전혀 읽지 않음 | 하~중 | 안정 (인젝션 방어 관행) | `[2차]` [Part 5/01](../05-cross-cutting/01-security-prompt-injection.md) |
| **클라이언트 측 검증만** | 프로토타입 | 운영 — 클라이언트는 조작 가능 | 하 | 해당 없음 | — |
| **HTML sanitizer로 생성 HTML을 호스트 문서에 삽입** | 아주 제한된 서식(굵게·링크)만 허용하는 텍스트 | 스크립트·폼·스타일이 필요한 위젯 — sanitizer 우회 역사 | 중 | 안정 (라이브러리) | `[2차]` 우회 사례 다수 |

### 선택 가이드

- **Controlled만 쓴다면** → props 스키마 + 서버 재검증 + 출처 태그. 이유: 코드 실행이 없으므로 샌드박스 불필요
- **Declarative 트리를 렌더한다면** → 트리·조합·바인딩 검증 + 렌더러가 값을 평가하지 않는지 코드 리뷰. 이유: "데이터"라는 주장 `[벤더]`은 렌더러 구현에 달렸다
- **서드파티 툴 UI나 생성 HTML을 보여 준다면** → iframe sandbox(`allow-same-origin` 없이) + CSP + 별도 출처 + postMessage 브리지. 이유: 코드 실행이므로 격리 외에 방법이 없다
- **생성 HTML을 "약간의 서식"으로만 쓴다면** → sanitizer보다 마크다운 → 안전 렌더. 이유: sanitizer 우회 역사가 길다
- **판단이 어려우면** → Open-ended를 열지 않는다. 이유: 보안 비용의 대부분이 거기서 나온다. 요구가 확인된 뒤 iframe으로만 연다

## 설계 시 고려사항 (체크리스트)

- [ ] 신뢰 경계 지도(출처 → 실행 위치 → 검증)가 문서로 있는가
- [ ] 낮음 등급 콘텐츠가 호스트 문서에서 실행되는 경로가 없는가
- [ ] iframe에 `allow-scripts`와 `allow-same-origin`이 함께 있지 않은가
- [ ] 샌드박스 콘텐츠가 호스트와 **다른 출처**에서 서빙되는가
- [ ] CSP가 `default-src 'none'`에서 출발해 필요한 것만 열렸는가. `img-src`까지 허용 목록인가
- [ ] postMessage 수신 시 출처를 검증하고 송신 시 대상 출처를 명시하는가(`*` 없음)
- [ ] iframe이 요청할 수 있는 능력이 화이트리스트이고, 툴 실행은 사용자 이벤트 → 서버 재검증 경로만 있는가
- [ ] 호스트 인증 토큰이 iframe에 전달되지 않는가
- [ ] 사용자 이벤트 `payload`가 구조화 값만 담고, 자유 텍스트가 에이전트 프롬프트에 직접 들어가지 않는가
- [ ] 서버가 `target` ID의 소유(이 사용자·이 run)를 확인하는가
- [ ] `as_shown_hash`·`idempotency_key`·`issued_at` 창 검증이 있는가
- [ ] 인라인 수정 필드별 권한·범위 검증이 있는가
- [ ] 승인 UI가 iframe 밖(Controlled)에만 있는가. 호스트 `frame-ancestors` 정책이 있는가
- [ ] iframe 경계가 사용자에게 시각적으로 표시되는가(피싱 UI 대비)
- [ ] 인젝션 테스트 케이스(문서 내 지시문 → UI → 이벤트)가 [Part 5/01](../05-cross-cutting/01-security-prompt-injection.md) 테스트에 포함되는가

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| 샌드박스 안의 피싱 UI | 사용자가 호스트로 착각해 자격 증명 입력 | 시각적 경계, 정책 고지, iframe 안 입력은 호스트 인증과 무관함을 보장 |
| sanitizer 우회 | 생성 HTML 삽입 시 XSS | 삽입 대신 iframe 또는 마크다운 |
| A2UI 렌더러의 값 평가 | "데이터"가 코드가 되는 순간 Declarative가 Open-ended로 | 렌더러 코드 리뷰, 평가 지점 금지 |
| MCP Apps 샌드박스 정책의 호스트별 편차 | 같은 UI가 호스트마다 다른 보안 수준 | 자사 호스트 정책을 스펙보다 엄격하게, 스펙 변경 추적 `[2차]` |
| UI 경유 인젝션의 간접성 | 문서 → UI → 사람 → 이벤트의 긴 경로라 테스트에서 빠짐 | 경로를 위협 모델에 명시, 엔드투엔드 인젝션 테스트 |
| 서버 재검증 누락 | "보였으니 허용" 가정 | 재검증을 미들웨어로 강제, 생략 불가 구조 |
| 보안이 사용성 저하 | 격리 때문에 iframe 높이·포커스·스타일 문제 | [07](07-frontend-integration.md)·[09](09-ux-accessibility.md)와 함께 설계 |

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 뉴로심볼릭 | 이 페이지의 검증 전부가 게이트 패턴(Rule ∧ Model). 모델·사람이 뭐라 하든 규칙(스키마·권한·CSP)이 최종 허가 | 항상 | `[2차]` [Part 1/04](../01-foundations/04-neuro-symbolic-ai.md) |
| MCP Apps 샌드박스 모델 | "서버 HTML → 호스트 샌드박스 iframe"이 서드파티 UI 수용의 표준 모델로. 2026-07-28 MCP 개정에 확장 편입 | MCP Apps 채택 | `[2차]` 공식 changelog 재확인 권장 |
| A2UI "코드가 아닌 데이터" | Declarative가 샌드박스 없이 안전하다는 주장의 근거. 렌더러 구현에 따라 성립 여부가 갈림 | A2UI 채택 | `[벤더]` |

## 관련 페이지

- [Part 5/01 보안과 프롬프트 인젝션](../05-cross-cutting/01-security-prompt-injection.md) — 인젝션 일반론, 입출력 필터, 출처 태그
- [Part 3/09 툴과 MCP](../03-agent-workflow-rag/09-tools-and-mcp.md) — 툴 권한 모델, [Part 2/10 접근 제어](../02-data-pipeline/10-access-control-governance.md)
- [01 Generative UI 스펙트럼](01-generative-ui-spectrum.md), [02 프로토콜 비교](02-protocols-ag-ui-a2ui-mcp-apps.md), [04 스트리밍·상태 동기화 — 서버 재검증](04-streaming-state-sync.md), [06 HITL UI — as_shown_hash](06-hitl-ui-patterns.md), [07 프론트엔드 통합 — 임베딩](07-frontend-integration.md)
- [Part 1/05 에이전트 프로토콜 — 프로토콜은 보안을 정의하지 않는다](../01-foundations/05-agentic-protocols.md)

## 참고 문헌

- `[검증]` WHATWG HTML Living Standard — iframe `sandbox` 속성, `postMessage`(출처 검증)
- `[검증]` W3C Content Security Policy — 지시어, nonce/hash, `frame-ancestors`
- `[2차]` MCP Apps 샌드박스 iframe 모델: CopilotKit 2026-01 지원 발표, MCP 2026-07-28 개정 해설. [연구 노트](../_meta/research-notes-2026-10.md) "프로토콜" 절
- `[벤더]` Google Developers Blog, A2UI v0.9 — "실행 코드가 아닌 데이터" 주장
- `[2차]` HTML sanitizer 우회 사례는 웹 보안 커뮤니티에 다수 보고. 특정 라이브러리 수치 인용하지 않음
