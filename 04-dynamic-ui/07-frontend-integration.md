---
title: "프론트엔드 통합"
part: 04-dynamic-ui
stage: interaction
tags: [react, nextjs, vercel-ai-sdk, copilotkit, flutter, a2ui, android, ios, embedding, iframe, server-components]
maturity: 시험
status: review
updated: 2026-10-11
---

# 프론트엔드 통합

> **범위**: React/Next.js, Vercel AI SDK, CopilotKit, 모바일(Flutter/네이티브) 통합. 선택한 패턴([01](01-generative-ui-spectrum.md))·프로토콜([02](02-protocols-ag-ui-a2ui-mcp-apps.md))·레지스트리([03](03-component-registry-schema.md))를 실제 프론트엔드 스택에 어떻게 붙이는지 — 통합 방식과 제약 중심. 제품 성능 수치·벤치마크는 다루지 않는다. 컴포넌트 구현 세부(스타일링, 상태 관리 라이브러리)도 범위 밖.
> **시각화**: [`assets/html/dynamic-ui-architecture.html`](../assets/html/dynamic-ui-architecture.html) — "렌더러(웹/모바일)" 노드

## 요약 (3줄)

1. 프론트엔드 통합의 핵심 질문은 **"컴포넌트 선택(툴 호출)을 어디서 컴포넌트 인스턴스로 바꾸는가"** 다. 클라이언트 레지스트리에서(React + AG-UI/CopilotKit), 서버에서 렌더해 UI 조각을 스트리밍(Vercel AI SDK RSC), 플랫폼 렌더러가 선언적 트리를 해석(A2UI on Flutter/Android/iOS) — 세 위치 중 하나다.
2. 웹은 선택지가 많고 성숙하며, 모바일 네이티브는 **A2UI 렌더러가 사실상 유일한 개방 선택지**이고 v0.9 `[벤더]` 단계다. 기존 앱 임베딩은 위젯(같은 DOM) vs iframe(격리)의 트레이드오프다.
3. 어떤 스택이든 **레지스트리와 이벤트 스키마를 프레임워크 코드에서 분리**해 둔다. 프레임워크·SDK가 반년 단위로 바뀌는 영역이므로, 바뀌지 않는 것(카탈로그, 계약 ②·③)을 중심에 두고 어댑터로 감싼다.

## 이 단계에서 무엇을 · 언제 · 어떻게 · 왜

| 질문 | 답 |
|---|---|
| **무엇을** (What) | ① 플랫폼별 렌더러 선택과 통합 방식 ② 레지스트리 → 프레임워크 컴포넌트 바인딩 코드 ③ 이벤트 스트림 → 프레임워크 상태 어댑터 ④ 사용자 이벤트 → 계약 ③ 변환 ⑤ 임베딩 방식(해당 시) |
| **언제** (When) | 패턴·프로토콜·레지스트리 결정 후. 프론트엔드가 이미 정해져 있는 기존 제품에 에이전트를 넣는 경우가 대부분이므로, 실제로는 "우리 스택에서 가능한 패턴은 무엇인가"를 거꾸로 묻게 된다. 그 경우 이 페이지를 먼저 읽고 [01](01-generative-ui-spectrum.md)로 돌아간다 |
| **어떻게** (How) | ① 대상 플랫폼 열거(웹 / iOS / Android / 데스크톱 / 기존 앱 안) → ② 플랫폼별로 선택 가능한 통합 방식 확인(아래 비교표) → ③ 레지스트리 바인딩 계층 작성(카탈로그 `name` → 컴포넌트 참조) → ④ 이벤트 어댑터 작성(스트림 범주 → 프레임워크 상태) → ⑤ 사용자 이벤트 변환기 → ⑥ 폴백(텍스트) 렌더 확인 → ⑦ 접근성·보안 검증([08](08-security-sandboxing.md), [09](09-ux-accessibility.md)) |
| **왜** (Why) | 통합 계층 없이 프레임워크 API를 컴포넌트 안에서 직접 호출하면 ① SDK 버전 변경이 모든 컴포넌트에 번지고 ② 같은 카탈로그를 웹·모바일에서 재사용할 수 없으며 ③ 테스트가 프레임워크 런타임에 묶인다 |

## 문제와 맥락

에이전트 백엔드는 "컴포넌트 `ProductCard`를 props `{...}`로"라고 말한다. 이것이 화면에 그려지려면 어딘가에서 **문자열 이름이 실행 가능한 컴포넌트로** 바뀌어야 한다. 그 "어딘가"가 어디인지가 통합 방식을 가른다.

- **클라이언트 레지스트리**: 브라우저·앱이 이름 → 컴포넌트 맵을 갖고 있다. 백엔드는 데이터만 보낸다
- **서버 렌더 스트리밍**: 서버가 컴포넌트를 렌더해 UI 조각(React Server Components 류)을 보낸다. 클라이언트는 조각을 끼운다
- **플랫폼 렌더러**: 선언적 트리를 각 플랫폼의 렌더러(Flutter 위젯, Android Compose, iOS SwiftUI 등)가 해석한다. A2UI의 모델 `[벤더]`

세 방식은 번들 크기, 백엔드 결합도, 플랫폼 커버리지, 보안 경계가 다르다.

## 핵심 개념

### 통합 방식별 특성

| 방식 | 이름 → 컴포넌트 변환 위치 | 백엔드 결합 | 플랫폼 | 주요 제약 |
|---|---|---|---|---|
| **React + AG-UI 클라이언트** (CopilotKit 류 포함) | 클라이언트 레지스트리 | 느슨(이벤트 스키마만) | 웹 | 레지스트리가 번들에 포함 → 컴포넌트 수가 번들 크기 |
| **Next.js + Vercel AI SDK** (툴 → 컴포넌트) | 클라이언트 레지스트리 또는 서버 | 프레임워크 규약(같은 저장소) | 웹 | 백엔드가 Next.js(또는 호환 런타임)여야 함 |
| **Vercel AI SDK RSC 스트리밍** | 서버 렌더 | 강함(서버가 UI를 앎) | 웹 | 서버 컴포넌트 제약(클라이언트 상호작용은 클라이언트 컴포넌트로 분리), 캐싱·재연결 복잡 |
| **Flutter + A2UI 렌더러** | 플랫폼 렌더러 | 느슨(트리 스키마) | iOS·Android·웹·데스크톱 | "Basic" 세트 밖은 커스텀 위젯 등록, v0.9 변경 추적 `[벤더]` |
| **Android Jetpack / iOS + A2UI** | 플랫폼 렌더러 | 느슨 | 각 OS 네이티브 | Android Jetpack 라이브러리 제공 `[벤더]`, iOS는 렌더러 성숙도 `[실측]` 필요 |
| **기존 앱 임베딩 — 위젯** | 호스트 앱의 DOM/뷰 트리 안 | 호스트와 공유 | 웹·네이티브 | 스타일 충돌, 호스트 CSP·보안 컨텍스트 공유 |
| **기존 앱 임베딩 — iframe** | 별도 문서 | 격리 | 웹 | postMessage 브리지, 높이 동기화, 접근성 트리 분리 ([08](08-security-sandboxing.md)) |

### 레지스트리 바인딩 계층

```ts
// 카탈로그 name → 컴포넌트. 프레임워크 코드는 이 맵 뒤에 숨긴다
const registry: Record<CatalogName, ComponentBinding> = {
  ProductCard: { component: ProductCard, schema: ProductCardProps, version: "2.3.0" },
  ProductComparisonTable: { component: ComparisonTable, schema: ComparisonTableProps, version: "2.3.0" },
  ApprovalCard: { component: ApprovalCard, schema: ApprovalCardProps, version: "2.3.0", actions: ["approval", "rejection", "edit"] },
};
// 렌더 직전: 스키마 검증 → 실패 시 텍스트 폴백. 모르는 name → 텍스트 폴백
```

- `schema`는 [03](03-component-registry-schema.md)에서 생성된 것을 import한다. 손으로 쓰지 않는다
- 이름을 모르거나 버전이 메이저 불일치면 **텍스트 폴백**. 예외를 던지지 않는다
- 바인딩 계층은 프레임워크별로 하나씩(React용, Flutter용). 카탈로그는 공유

### 이벤트 어댑터

스트림 범주(생명주기 / 텍스트 / 툴 호출 / 상태, [04](04-streaming-state-sync.md)) → 프레임워크 상태 저장소로 변환하는 계층. AG-UI SDK나 CopilotKit이 이 어댑터를 제공하지만, **자체 어댑터 인터페이스로 한 번 감싼다.** SDK가 바뀌어도 컴포넌트는 모른다.

역방향(사용자 이벤트 → 계약 ③)도 같은 계층에서. 컴포넌트는 `onAction({type, payload})`만 호출하고, 봉투(`idempotency_key`, `run_id`, `client_state_seq`)는 어댑터가 채운다.

### 서버 컴포넌트 활용의 경계

RSC 류 서버 렌더 스트리밍은 "컴포넌트를 클라이언트 번들에 넣지 않아도 된다"는 장점이 있다. 하지만 에이전트 UI는 대부분 **상호작용**(승인·수정·선택)이 있으므로 그 부분은 결국 클라이언트 컴포넌트다. 실무 배치는 다음과 같다.

- 표시 전용(citation 리스트, 표, 차트의 정적 부분): 서버 렌더 가능
- 상호작용(승인 카드, 인라인 수정, 선택지): 클라이언트 컴포넌트 + 클라이언트 레지스트리
- 재연결·순서 보장([04](04-streaming-state-sync.md))은 서버 렌더 조각에도 적용되어야 하며, 프레임워크가 제공하는 범위를 확인한다

### 기존 앱에 임베딩

| 축 | 위젯(같은 문서) | iframe(격리) |
|---|---|---|
| 디자인 일관성 | 호스트 토큰 공유 가능 | 토큰을 iframe에 전달해야 함 |
| 보안 | 호스트와 같은 신뢰 경계 — 에이전트 UI 결함이 호스트에 영향 | 샌드박스로 격리. Open-ended 패턴은 **반드시** 이쪽 |
| 통신 | 함수 호출 | postMessage + 스키마 검증 |
| 접근성 | 하나의 접근성 트리 | 분리된 트리, 포커스 이동 처리 필요 |
| 적합 | 자사 Controlled 컴포넌트 | 서드파티 UI(MCP Apps), 신뢰 수준이 다른 코드 |

## 아키텍처 / 파이프라인

입력은 [03](03-component-registry-schema.md)의 카탈로그(+ 생성된 스키마), [04](04-streaming-state-sync.md)의 이벤트 스트림·사용자 이벤트 스키마, 플랫폼 목록. 처리는 플랫폼별 바인딩 계층 + 이벤트 어댑터. 출력은 렌더된 화면과 계약 ③ 이벤트. 이 단계 뒤에 [08](08-security-sandboxing.md)의 보안 검증과 [09](09-ux-accessibility.md)의 접근성 검증이 붙는다.

## 기술 비교표 — 이 단계에서 쓸 수 있는 선택지

| 기술 / 접근 | 적합한 상황 | 부적합한 상황 | 비용·복잡도 | 성숙도 | 검증 수준 |
|---|---|---|---|---|---|
| **React/Next.js + Vercel AI SDK** (툴 호출 → 컴포넌트, RSC 스트리밍 옵션) | 풀스택 단일 팀, Next.js(또는 호환 런타임) 백엔드, 웹만, 빠른 출시 | 백엔드가 Python/.NET/Java, 모바일 네이티브 필요, 백엔드 교체 가능성 | 하~중 | 안정 | — 프레임워크 문서, 수치 인용 안 함 |
| **React + CopilotKit** (AG-UI 클라이언트, 액션 정의 훅, 공유 상태) | 프론트·백 분리, 백엔드 프레임워크 다양(AG-UI 지원 백엔드), Controlled 패턴 | CopilotKit 추상화를 벗어난 커스텀 UI 흐름이 많음, 의존성 심사 엄격 | 중 | 신생→확산 (AG-UI 주도사) | `[2차]` `[벤더]` |
| **React + AG-UI SDK 직접** | CopilotKit 추상화 없이 이벤트 어댑터를 직접 통제, 기존 상태 관리와 통합 | 빠른 프로토타입(CopilotKit이 더 빠름) | 중~상 | 신생 | `[2차]` |
| **Flutter + A2UI 렌더러** | 멀티플랫폼 모바일+웹 단일 코드베이스, Declarative 패턴 | Controlled만 필요(과잉), 세트 밖 컴포넌트 다수, 플랫폼 네이티브 룩 엄격 | 중~상 | 신생 (v0.9) | `[벤더]` |
| **Android Jetpack / iOS 네이티브 + A2UI** | 네이티브 앱에 에이전트 UI, 플랫폼 디자인 가이드 준수 | iOS 렌더러 성숙도 미확인 상태에서 핵심 기능 의존 | 상 — 플랫폼별 렌더러·커스텀 컴포넌트 | 신생 (Android Jetpack 라이브러리 v0.9 `[벤더]`) | `[벤더]` |
| **네이티브 + 자체 Controlled 렌더러** | 컴포넌트 수 적음, A2UI 의존 회피, 네이티브 팀 보유 | 컴포넌트 많거나 Declarative 필요 | 중 | 안정 (일반 네이티브 개발) | — |
| **기존 앱 임베딩 — 위젯** | 자사 Controlled UI를 기존 웹앱·포털에 삽입 | 서드파티 UI, 신뢰 수준 다른 코드 | 하~중 | 안정 | — |
| **기존 앱 임베딩 — iframe** | 서드파티 UI(MCP Apps), 격리 필수, 여러 호스트에 같은 UI | 호스트와 긴밀한 상태 공유·디자인 일체감 | 중 — 브리지·높이 동기화·접근성 | 표준 (iframe) / 신생 (MCP Apps 호스트) | `[검증]` HTML 표준 / `[2차]` MCP Apps |

### 선택 가이드

- **Next.js 풀스택 단일 팀이고 웹만이라면** → Vercel AI SDK. 이유: 통합이 끝나 있다. 단 "툴 호출 = 컴포넌트" 구조와 레지스트리 분리는 유지해 AG-UI 전환 여지를 둔다
- **백엔드가 Python/.NET이고 프론트가 React라면** → CopilotKit 또는 AG-UI SDK 직접. 이유: 이벤트 프로토콜이 언어 경계를 넘는다. MS .NET SDK(2026-09-25) `[2차]`로 .NET 쪽 선택지도 생겼다
- **iOS·Android 네이티브 앱에 Declarative UI를 그려야 한다면** → A2UI 렌더러를 `[실측]` 후 채택, 그 전엔 자체 Controlled 렌더러. 이유: 개방 선택지가 A2UI뿐이지만 v0.9이고 멀티플랫폼 주장은 `[벤더]`
- **기존 포털에 서드파티 툴 UI를 넣어야 한다면** → iframe + [08](08-security-sandboxing.md) 샌드박스. 이유: 신뢰 경계가 다르다
- **판단이 어려우면** → React 클라이언트 레지스트리 + 이벤트 어댑터 인터페이스를 먼저 만들고, SDK(Vercel AI SDK 또는 CopilotKit)는 어댑터 뒤에 둔다. 이유: 어댑터만 바꾸면 SDK를 바꿀 수 있다

## 설계 시 고려사항 (체크리스트)

- [ ] 레지스트리 바인딩 계층이 프레임워크 코드와 분리되어 있는가. 컴포넌트가 SDK API를 직접 호출하지 않는가
- [ ] 바인딩의 `schema`가 [03](03-component-registry-schema.md)에서 생성된 것을 import하는가
- [ ] 모르는 컴포넌트·메이저 버전 불일치·스키마 실패가 모두 **텍스트 폴백**으로 떨어지는가(예외 아님)
- [ ] 이벤트 어댑터가 SDK를 감싸고 있어 SDK 교체 시 컴포넌트가 바뀌지 않는가
- [ ] 사용자 이벤트 봉투(`idempotency_key`, `run_id`, `client_state_seq`)를 어댑터가 채우고 컴포넌트는 `type`·`payload`만 아는가
- [ ] 서버 렌더를 쓴다면 상호작용 컴포넌트가 클라이언트 쪽에 있고 재연결·순서 보장이 적용되는가
- [ ] 모바일에서 A2UI를 쓴다면 "Basic" 세트 커버리지와 커스텀 위젯 등록 비용을 확인했는가
- [ ] 임베딩이라면 위젯/iframe 선택 근거(신뢰 경계)가 문서화되어 있는가
- [ ] 번들 크기 — 레지스트리의 컴포넌트가 지연 로딩되는가(컴포넌트 수가 수십 개일 때)
- [ ] 프레임워크·SDK 버전이 고정되어 있고 업그레이드 절차가 있는가 ([Part 5/05](../05-cross-cutting/05-llmops-agentops.md))

## 한계와 리스크

| 한계 | 영향 | 완화책 |
|---|---|---|
| 모바일 네이티브 개방 선택지가 A2UI뿐 | v0.9 변경·렌더러 품질에 종속 | `[실측]` 선행, 자체 Controlled 렌더러 병행 |
| SDK·프레임워크 빈번 변경 | 반년 단위 마이그레이션 | 어댑터 계층, 버전 고정 |
| 프레임워크 규약(Vercel AI SDK 류)의 백엔드 결합 | 백엔드 교체 시 프론트도 변경 | "툴 호출 = 컴포넌트" 구조 유지, 레지스트리 분리 |
| 서버 렌더 조각의 재연결·순서 보장 | 프레임워크가 다 제공하지 않을 수 있음 | 표시 전용에만 서버 렌더, 상호작용은 클라이언트 |
| 임베딩 위젯의 스타일·보안 컨텍스트 공유 | 호스트 CSS 충돌, 호스트 권한으로 실행 | Shadow DOM 류 스타일 격리, 서드파티는 iframe |
| 번들 크기 | 컴포넌트 수십 개가 초기 로딩에 포함 | 지연 로딩, 의도 그룹별 청크 |
| CopilotKit 의존 시 AG-UI 주도사 종속 | 로드맵 변경 영향 | AG-UI SDK 직접 사용 경로 확보 |

## 최신 트렌드 적용 (최근 1년)

| 트렌드 | 이 단계에서의 역할 | 적용 조건 | 검증 수준 |
|---|---|---|---|
| 온톨로지 | 해당 없음 — 프레임워크 통합은 도메인 의미와 무관한 층. 온톨로지는 카탈로그([03](03-component-registry-schema.md))와 매핑([05](05-intent-to-ui-mapping.md))에서 소비된다 | — | — |
| 뉴로심볼릭 | 렌더 직전의 스키마 검증(바인딩 계층)이 검증기 패턴의 마지막 관문. 서버 검증을 통과했어도 클라이언트가 한 번 더 확인하고 폴백 | 항상 | `[2차]` |
| 결정 모델 (Jev / Decisions API) | 해당 없음 — 렌더러는 판정하지 않는다. 판정은 백엔드([05](05-intent-to-ui-mapping.md))에서 끝나 `ui_intent`로 도착한다 | — | — |
| A2UI 모바일 렌더러 | v0.9의 Android Jetpack 라이브러리·Flutter 렌더러로 모바일 Declarative가 가능해짐 | 모바일 Declarative | `[벤더]` |
| AG-UI 대형 벤더 SDK | Microsoft .NET SDK(2026-09-25), AWS Bedrock AgentCore(2026-03)로 백엔드 쪽 선택지 확대 → React 프론트의 백엔드 중립성 강화 | 해당 스택 | `[2차]` `[벤더]` |
| MCP Apps 호스트 | 호스트(IDE·챗·자체 앱)가 서드파티 UI를 iframe으로 수용하는 모델이 표준화되는 중. CopilotKit 2026-01 지원 | 서드파티 툴 UI | `[2차]` |

## 관련 페이지

- [01 Generative UI 스펙트럼](01-generative-ui-spectrum.md), [02 프로토콜 비교](02-protocols-ag-ui-a2ui-mcp-apps.md), [03 컴포넌트 레지스트리](03-component-registry-schema.md), [04 스트리밍·상태 동기화](04-streaming-state-sync.md)
- [08 보안·샌드박싱](08-security-sandboxing.md) — iframe 임베딩, [09 UX·접근성](09-ux-accessibility.md) — 접근성 트리
- [Part 3/14 프레임워크 비교](../03-agent-workflow-rag/14-frameworks-landscape.md) — 백엔드 쪽, [Part 5/05 LLMOps / AgentOps](../05-cross-cutting/05-llmops-agentops.md) — 버전·배포

## 참고 문헌

- `[2차]` `[벤더]` AG-UI: CopilotKit 문서, Microsoft .NET SDK(2026-09-25), AWS Bedrock AgentCore 지원(2026-03). [연구 노트](../_meta/research-notes-2026-10.md) "프로토콜" 절
- `[벤더]` Google Developers Blog, A2UI v0.9 (2026-04-17) — Flutter·Android Jetpack 렌더러, 멀티플랫폼 주장
- `[2차]` MCP Apps 호스트 모델: CopilotKit 2026-01 지원 발표
- Vercel AI SDK·Next.js RSC는 프레임워크 공식 문서 기준. 성능 수치는 이 위키의 규칙에 따라 인용하지 않는다
