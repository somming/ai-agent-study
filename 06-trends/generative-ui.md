---
title: "Generative UI"
part: 06-trends
stage: interaction
tags: [trend, generative-ui, genui, ag-ui, a2ui, mcp-apps, controlled, declarative, open-ended]
maturity: 시험
status: review
updated: 2026-10-11
first_seen: 2026-01          # MCP Apps 지원 발표(2026-01)와 A2UI v0.9(2026-04)로 3패턴이 각각 대표 프로토콜을 갖게 된 시점
---

# Generative UI

> **범위**: 3패턴 정착, 프레임워크 지원, 모바일 확산

> **한 줄 정의**: 에이전트가 텍스트 대신(또는 텍스트와 함께) **UI를 생성·선택·갱신**하는 상호작용 방식. 최근 1년에 **Controlled / Declarative / Open-ended** 세 패턴으로 정리되었고, 각 패턴이 대표 프로토콜(AG-UI / A2UI / MCP Apps)을 갖게 되면서 "어느 패턴을 쓸 것인가"가 설계 결정이 되었다.
> **레이더 위치**: **시험** — 3패턴 분류는 실무에 유용하나 AG-UI 주도사(CopilotKit)의 프레이밍 `[2차]`이고, A2UI는 v0.9, MCP Apps는 2026-07 스펙 편입 직후다. 변경 빈도가 높아 "채택"은 Controlled 패턴에서 `[실측]`이 쌓인 뒤로 미룬다. Part 4 전체가 이 트렌드의 적용 단계다.

## 무엇이 바뀌었나 (최근 1년)

| 시점 | 사건 | 검증 수준 |
|---|---|---|
| 2026-01 | **MCP Apps**: MCP 서버가 HTML UI 반환, 호스트가 샌드박스 iframe 렌더. CopilotKit 지원 발표 | `[2차]` |
| 2026-03 | AWS Bedrock AgentCore, AG-UI 지원 | `[2차]` |
| 2026-04-17 | **A2UI v0.9**: "Standard" 컴포넌트 세트 → "Basic" 개명, Agent SDK, Android Jetpack 라이브러리 — **모바일 네이티브 렌더** 확산의 근거 | `[벤더]` |
| 2026-07-28 | MCP 개정에 **MCP Apps 확장** 편입 | `[2차]` |
| 2026-09-25 | Microsoft .NET AG-UI SDK | `[2차]` |
| 2026 | **3패턴 프레이밍** 정착: Controlled(AG-UI) / Declarative(A2UI, Open-JSON-UI) / Open-ended(MCP Apps, 커스텀). "대부분 팀이 프레임워크 기본값 때문에 Controlled로 흘러간다" | `[2차]` CopilotKit 자체 프레이밍 |

2025년까지 Generative UI는 "LLM이 React 컴포넌트를 고른다"는 프레임워크별 기능이었다. 바뀐 것은 ① 패턴의 어휘가 생겼고, ② 패턴마다 프로토콜이 붙었으며, ③ 웹을 넘어 **모바일 네이티브 렌더**가 주장되기 시작했다는 것이다. 프로토콜 비교는 [Part 1/05](../01-foundations/05-agentic-protocols.md)와 [agentic-protocols-convergence](agentic-protocols-convergence.md), 스펙트럼 상세는 [Part 4/01](../04-dynamic-ui/01-generative-ui-spectrum.md)에 있다.

## 왜 주목받나

1. **텍스트의 한계**: 표·폼·차트·선택지를 산문으로 설명하는 것은 느리고 오류가 많다. 상담·커머스·운영 대시보드에서 "보여주고 고르게 하는" 상호작용이 필요하다.
2. **HITL의 UI화**: 승인·수정·선택을 요구하는 Human-in-the-Loop가 채팅 안에서 버튼·폼으로 처리되어야 한다 ([Part 4/06](../04-dynamic-ui/06-hitl-ui-patterns.md)). Generative UI는 HITL의 전달 수단이다.
3. **툴 생태계의 요구**: 서드파티 툴이 자기 위젯을 가진 채 호스트에 꽂히고 싶어 한다. MCP Apps의 "툴이 UI를 가져오는" 모델이 이 요구에 답한다.
4. **멀티플랫폼 압력**: 웹 전용 React 컴포넌트 생성은 모바일 앱에 쓸 수 없다. A2UI의 "실행 코드가 아닌 데이터" 접근이 여기서 나왔다 `[벤더]`.

## 핵심 메커니즘

### 3패턴

| 패턴 | 누가 UI를 결정하나 | 전송·표현 | 대표 프로토콜 | 안전성 | 표현력 | 2026 위상 |
|---|---|---|---|---|---|---|
| **Controlled** | 개발자가 미리 등록한 컴포넌트 중 **에이전트가 선택**, 인자만 채움 | 타입드 이벤트(상태 패치) | **AG-UI** | 높음 — 레지스트리 밖 UI 불가 | 레지스트리에 한정 | **사실상 기본값** `[2차]` |
| **Declarative** | 에이전트가 **UI 트리(JSON)** 를 생성, 렌더러가 플랫폼별로 그림 | 선언적 데이터 | **A2UI**, Open-JSON-UI | 중간 — 트리 검증 필요, 실행 코드 아님 | 컴포넌트 세트 내 자유 조합 | v0.9, 변경 중 |
| **Open-ended** | 에이전트 또는 **툴 서버**가 HTML/코드를 생성 | HTML → 샌드박스 iframe | **MCP Apps**, 커스텀 | 낮음 — 샌드박스 의존 | 사실상 무제한 | 스펙 편입 직후 |

### 왜 Controlled가 기본값이 되었나

CopilotKit의 프레이밍에 따르면 "대부분 팀이 프레임워크 기본값 때문에 Controlled로 흘러간다" `[2차]`. 이 설명은 AG-UI 주도사의 것이지만, 구조적 이유도 분명하다: ① 컴포넌트 레지스트리([Part 4/03](../04-dynamic-ui/03-component-registry-schema.md))가 있으면 LLM 출력 검증이 **enum 선택 + 인자 스키마**로 환원되어 가장 쉽고, ② 디자인 시스템·접근성·보안을 기존 컴포넌트가 이미 책임지며, ③ 선택 문제는 [decision-models](decision-models.md)의 결정 모델이 맡기에 적합한 닫힌 선택지다. **Controlled는 "덜 생성적"이지만 운영 가능성이 가장 높다.**

### 두 가지 다른 UI 출처

- **에이전트가 UI를 결정** (Controlled, Declarative): UI는 에이전트 설계자의 의도를 표현한다. 레지스트리·컴포넌트 세트가 통제 지점
- **툴 서버가 UI를 가져옴** (MCP Apps): UI는 **툴 제공자**의 것이다. 호스트는 내용을 모르고 샌드박스만 제공한다. 서드파티 확장에 유리하지만 **보안·UX 일관성의 통제권이 호스트에서 툴로** 넘어간다 ([Part 4/08](../04-dynamic-ui/08-security-sandboxing.md))

### 모바일 확산

A2UI는 UI 트리가 실행 코드가 아닌 데이터이므로 웹·Flutter·Android·iOS에서 **네이티브 렌더**가 가능하다고 주장하며, v0.9에서 Android Jetpack 라이브러리를 추가했다 `[벤더]`. Controlled 패턴도 플랫폼별 레지스트리를 두면 멀티플랫폼이 되지만 컴포넌트를 플랫폼마다 구현해야 한다. "한 번 생성, 어디서나 렌더"는 Declarative의 약속이고, 아직 `[벤더]`다.

## 벤더 주장 vs 검증된 사실

| 주장 | 출처 | 검증 수준 | 우리의 판단 |
|---|---|---|---|
| Generative UI는 Controlled / Declarative / Open-ended 3패턴으로 정리된다 | CopilotKit | `[2차]` CopilotKit 자체 프레이밍 | 분류 자체는 유용해 **어휘로 채택**. 단 AG-UI 주도사가 Controlled를 중심에 놓는 프레이밍임을 인식 |
| 대부분 팀이 프레임워크 기본값 때문에 Controlled로 흘러간다 | CopilotKit | `[2차]` | 구조적 이유(검증 용이·기존 컴포넌트 재사용)로 설득력 있음. 우리도 Controlled에서 시작 |
| A2UI는 데이터라서 웹·Flutter·Android·iOS 네이티브 렌더가 된다 | Google Developers Blog, A2UI v0.9 (2026-04-17) | `[벤더]` | 구조적으로 가능하나 **렌더러 간 동일성·접근성·성능은 미검증**. v0.9에서 컴포넌트 세트 개명 → 호환성 파괴 진행 중 |
| MCP Apps로 서드파티 툴이 자기 UI를 안전하게 가져올 수 있다 | MCP 커뮤니티, CopilotKit (2026-01) | `[2차]` | "안전"은 샌드박스 iframe 구현에 전적으로 의존. 호스트 샌드박스 정책을 `[실측]`으로 검증해야 함 |
| AG-UI가 Microsoft·AWS 채택으로 표준이 되고 있다 | Microsoft (2026-09-25), AWS (2026-03) | `[2차]` `[벤더]` | 채택은 사실. 거버넌스는 스타트업 단일 — [agentic-protocols-convergence](agentic-protocols-convergence.md) |
| Generative UI가 사용자 과업 완료율을 높인다 | 프레임워크 벤더 일반 | `[벤더]` | 노트에 독립 측정 없음. 텍스트 대비 효과는 우리 시나리오에서 `[실측]` ([Part 4/10](../04-dynamic-ui/10-evaluation-genui.md)) |

## 이 위키의 어느 단계에 적용되나

Part 4 전체가 적용 단계다. 각 페이지의 "최신 트렌드 적용" 섹션이 구체 방법을 다룬다.

| 단계 페이지 | 역할 | 적용 조건 |
|---|---|---|
| [Part 4/01 Generative UI 스펙트럼](../04-dynamic-ui/01-generative-ui-spectrum.md) | 3패턴의 상세 비교·선택 가이드 | 모든 GenUI 설계 |
| [Part 4/02 AG-UI · A2UI · MCP Apps 상세 비교](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) | 패턴별 프로토콜 선택 | 프로토콜 채택 |
| [Part 4/03 컴포넌트 레지스트리와 스키마](../04-dynamic-ui/03-component-registry-schema.md) | Controlled의 핵심 산출물. 선택지·인자 스키마 | Controlled |
| [Part 4/04 스트리밍과 상태 동기화](../04-dynamic-ui/04-streaming-state-sync.md) | AG-UI 이벤트·상태 패치 구현 | 스트리밍 UI |
| [Part 4/05 의도 → UI 매핑](../04-dynamic-ui/05-intent-to-ui-mapping.md) | 어떤 컴포넌트를 고를지 — 결정 모델·온톨로지 적용 지점 | Controlled / Declarative |
| [Part 4/06 HITL UI 패턴](../04-dynamic-ui/06-hitl-ui-patterns.md) | 승인·수정 UI | 행동하는 에이전트 |
| [Part 4/07 프론트엔드 통합](../04-dynamic-ui/07-frontend-integration.md) | 렌더러·프레임워크 통합, 모바일 | 멀티플랫폼 |
| [Part 4/08 보안·샌드박싱](../04-dynamic-ui/08-security-sandboxing.md) | Open-ended의 iframe 샌드박스, Declarative 트리 검증 | Declarative / Open-ended |
| [Part 4/09 UX·접근성](../04-dynamic-ui/09-ux-accessibility.md) | 생성된 UI의 접근성·일관성 | 모든 GenUI |
| [Part 4/10 Generative UI 평가](../04-dynamic-ui/10-evaluation-genui.md) | 텍스트 대비 효과, 렌더 실패율, 선택 정확도 | 채택 판단 |

## 한계와 미해결 질문

| 한계 / 질문 | 왜 문제인가 | 현재 상태 |
|---|---|---|
| **미성숙·변경 빈도** | A2UI v0.9 개명, MCP Apps 스펙 편입 직후. 반년 단위 마이그레이션 | 버전 고정, 어댑터 계층, Controlled 우선 |
| **3패턴 프레이밍의 출처** | AG-UI 주도사의 분류. Declarative·Open-ended가 상대적으로 불리하게 그려질 수 있음 | 각 패턴을 자체 기준으로 재평가 |
| **Open-ended의 보안** | 툴 서버 HTML을 호스트가 렌더. 샌드박스 이탈·피싱 UI 위험 | 샌드박스 정책 검증, [Part 4/08](../04-dynamic-ui/08-security-sandboxing.md) |
| **UX 일관성** | MCP Apps는 툴마다 다른 디자인. Declarative도 렌더러별 차이 | 디자인 토큰 주입 가능 여부 미확인 |
| **접근성** | 생성된 UI가 접근성 기준을 지키는지 자동 보장 없음 | Controlled는 컴포넌트가 책임, 나머지는 검증 필요 |
| **멀티플랫폼 주장 미검증** | A2UI 렌더러 간 동일성 | `[실측]` 없음 |
| **효과 측정 부재** | 텍스트 대비 과업 완료율 개선의 독립 측정 없음 | [Part 4/10](../04-dynamic-ui/10-evaluation-genui.md)에서 설계 |
| 결정 모델의 컴포넌트 선택 | 닫힌 선택지라 적합해 보이나 사례 미확인 | `[2차]` [decision-models](decision-models.md) |

## 우리 프로젝트에서의 채택 기준

패턴별로 기준이 다르다. **Controlled → Declarative → Open-ended** 순으로 올리며, 앞 패턴의 `[실측]` 없이 다음으로 가지 않는다.

**Controlled ("시험" → "채택")**
1. **텍스트 대비 효과 `[실측]`**: 같은 시나리오([Part 7/02](../07-appendix-scenarios/02-commerce-assistant-genui.md) 등)에서 텍스트 응답 vs Controlled UI의 과업 완료율·소요 시간·오류율 비교.
2. **선택 정확도 `[실측]`**: 레지스트리 컴포넌트 선택의 정답률(골든셋). 판정 수단(프롬프트 LLM / 결정 모델) 비교 포함.
3. **렌더 실패율 `[실측]`**: 인자 스키마 불일치·누락으로 렌더가 실패하는 비율과 폴백(텍스트) 동작 확인.
4. **레지스트리 운영**: 컴포넌트 추가·변경 절차와 접근성 체크리스트가 돌아가고 있다.

**Declarative (Controlled 채택 후)**
5. **트리 검증기**: 생성된 UI 트리를 스키마·허용 컴포넌트·값 범위로 검증하는 단계가 있고 거부율이 측정된다 ([Part 1/04](../01-foundations/04-neuro-symbolic-ai.md) 검증기 패턴).
6. **렌더러 동일성 `[실측]`**: 대상 플랫폼(웹 + 모바일 1종 이상)에서 같은 트리의 렌더 결과·접근성 비교. A2UI `[벤더]` 주장의 우리 검증.
7. **버전 고정과 마이그레이션 비용 기록**: v0.9 → 다음 버전 전환에 든 비용.

**Open-ended / MCP Apps (명시적 요구가 있을 때만)**
8. **샌드박스 검증 `[실측]`**: 호스트 iframe 정책이 스크립트·네트워크·스토리지 접근을 의도대로 차단하는지 테스트. 피싱 UI 시나리오 포함.
9. **UX 일관성 기준**: 서드파티 UI에 적용할 최소 디자인·접근성 기준과 심사 절차.

## 참고 문헌

- `[2차]` CopilotKit, "AG-UI and A2UI explained" — 3패턴 프레이밍(Controlled / Declarative / Open-ended), "대부분 팀이 Controlled로 흘러간다". CopilotKit은 AG-UI 주도사
- `[벤더]` Google Developers Blog, A2UI v0.9 (2026-04-17) — "Standard" → "Basic" 개명, Agent SDK, Android Jetpack 라이브러리, 멀티플랫폼 네이티브 렌더 주장; InfoQ (2026-07)
- `[2차]` MCP Apps CopilotKit 지원 발표 (2026-01); MCP 2026-07-28 개정의 Apps 확장 편입
- `[2차]` The New Stack, "Agent UI standards multiply: MCP Apps and Google's A2UI"
- `[2차]` AG-UI 채택: Microsoft .NET SDK (2026-09-25), AWS Bedrock AgentCore (2026-03), ThoughtWorks Technology Radar
- 관련 트렌드: [agentic-protocols-convergence](agentic-protocols-convergence.md), [decision-models](decision-models.md) (컴포넌트 선택), [ontology-renaissance](ontology-renaissance.md) (선택지·슬롯 값의 출처)

_검증 수준 표기: `[검증]` `[벤더]` `[2차]` `[실측]` — 규칙은 [작성 가이드](../_meta/writing-guide.md) 참조. 이 페이지의 모든 날짜·수치는 [조사 노트](../_meta/research-notes-2026-10.md)와 동일하다._
