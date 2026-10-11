---
title: "에이전트 프로토콜 수렴"
part: 06-trends
stage: interaction
tags: [trend, mcp, a2a, ag-ui, a2ui, mcp-apps, protocols, interoperability]
maturity: 시험
status: review
updated: 2026-10-11
first_seen: 2025-06          # A2A Linux Foundation 이관(2025-06)으로 "경쟁에서 조합으로" 서사가 시작. MCP 2026-07-28 개정으로 역할 분담이 굳어짐
---

# 에이전트 프로토콜 수렴

> **범위**: MCP·A2A·AG-UI·A2UI·MCP Apps의 역할 분담이 굳어지는 과정

> **한 줄 정의**: 2025~26년에 난립하던 에이전트 프로토콜이 서로 **경쟁하는 대안**이 아니라 **경계별로 역할을 나눠 조합**하는 구도로 정리된 흐름. MCP(에이전트 ↔ 툴·데이터) · A2A(에이전트 ↔ 에이전트) · AG-UI(백엔드 ↔ 프론트엔드 이벤트) · A2UI(UI 내용) · MCP Apps(툴 서버가 UI 반환)가 각자의 층을 맡는다.
> **레이더 위치**: **시험** — MCP와 A2A는 벤더 중립 거버넌스(오픈 스펙 / Linux Foundation)와 다벤더 SDK를 갖춰 안정권에 가깝다. UI 쪽 세 프로토콜(AG-UI, A2UI, MCP Apps)은 v0.9·스타트업 거버넌스·반년 단위 변경으로 미성숙하다. 전체를 "시험"에 두되 MCP·A2A는 "당사자 둘 이상" 경계에서 우선 적용한다.

## 무엇이 바뀌었나 (최근 1년)

| 시점 | 사건 | 검증 수준 |
|---|---|---|
| 2025-04 | Google, **A2A** 공개 | `[2차]` |
| 2025-06 | **A2A를 Linux Foundation으로 이관**. 100+ 기업 참여 | `[2차]` |
| 2026-01 | **MCP Apps**: MCP 서버가 HTML UI를 반환하고 호스트가 샌드박스 iframe에 렌더. CopilotKit 지원 발표 | `[2차]` |
| 2026-03 | **AWS Bedrock AgentCore가 AG-UI 지원** | `[2차]` |
| 2026-04-17 | **A2UI v0.9**: "Standard" 컴포넌트 세트 → "Basic" 개명, Agent SDK 추가, Android Jetpack 라이브러리 | `[벤더]` |
| 2026-05-21 | MCP 개정 RC | `[2차]` |
| 2026-07-28 | **MCP 개정 최종**: initialize 핸드셰이크·Mcp-Session-Id 제거 → **무상태 코어**, 라운드로빈 스케일링. **Multi Round-Trip Requests**, **Tasks 확장**(장기 실행), **MCP Apps 확장**(서버 렌더 UI). C# SDK v2.0 하위 호환 | `[2차]` 공식 changelog 재확인 권장 |
| 2026-09-25 | **Microsoft .NET AG-UI SDK** | `[2차]` |
| 2026 | AG-UI ThoughtWorks Technology Radar 등재 | `[2차]` |
| 2026 | A2A 1.0 도달 **정황**(A2UI가 "A2A 1.0 통합" 언급)이나 전용 발표 미확인 | `[2차]` |

다섯 프로토콜의 정의·경계·거버넌스 비교표는 [Part 1/05](../01-foundations/05-agentic-protocols.md)에 있다. 이 페이지는 **왜 이 구도가 "수렴"으로 불리는지, 각 사건이 역할 분담을 어떻게 굳혔는지**만 다룬다.

## 왜 주목받나

2025년 초의 질문은 "MCP냐 A2A냐"였다. 2026년 말의 답은 "둘 다, 다른 경계에서"다. 세 가지 사건이 이 전환을 만들었다.

1. **거버넌스의 중립화**: A2A가 Google 소유에서 Linux Foundation으로 옮겨가며 "Google 프로토콜 vs Anthropic 프로토콜" 구도가 무너졌다. 조직 간 연동의 기본 선택지로 A2A를 고르는 데 벤더 종속 우려가 줄었다 `[2차]`.
2. **MCP의 역할 확장**: 2026-07-28 개정으로 MCP는 "툴 호출 프로토콜"에서 **에이전트 런타임 프로토콜**로 넓어졌다. 무상태 코어(수평 확장), Tasks(장기 실행), Apps(UI)가 들어오면서 A2A와 겹치는 듯 보이지만, 실제로는 **MCP는 호스트 안쪽 자원**, **A2A는 호스트 바깥 에이전트**라는 경계가 더 뚜렷해졌다.
3. **UI 층의 분리**: AG-UI(이벤트 전송), A2UI(UI 내용), MCP Apps(툴 서버의 UI)가 **서로 다른 질문에 답한다**는 것이 2026년에 정리되었다. Microsoft(.NET SDK)와 AWS(AgentCore)가 AG-UI를 채택하면서 "스타트업 프로토콜"의 신뢰 문제가 일부 완화되었다 `[2차]`.

프로토콜은 **경계를 넘는 당사자가 둘 이상**일 때 가치가 있다 ([Part 1/05](../01-foundations/05-agentic-protocols.md) "프로토콜을 쓰지 않을 때"). 수렴 서사의 실무적 의미는 "다섯 개를 다 쓰라"가 아니라 "경계마다 하나씩, 필요한 경계에만"이다.

## 핵심 메커니즘

### 경계별 역할 분담

```
                        ┌─────────────── A2A ───────────────┐
                        │ (에이전트 ↔ 외부/타 조직 에이전트)   │
  사용자 ── 프론트엔드 ══ AG-UI ══ 에이전트 백엔드 ══ MCP ══ 툴·데이터 소스
             │  ▲                       │                     │
             │  └── A2UI (UI 트리 내용) ─┘                     └── MCP Apps (서버가 UI 반환)
             └── 렌더러 (React / Flutter / 네이티브)
```

| 경계 | 프로토콜 | 2026년에 굳어진 역할 | 상태 변화의 의미 |
|---|---|---|---|
| 에이전트 ↔ 툴·데이터 | **MCP** | 툴·리소스·프롬프트 발견·호출 + 장기 실행(Tasks) + 서버 UI(Apps) | 무상태화로 **세션 상태 책임이 호스트·서버로 이동**. 재설계 필요 |
| 에이전트 ↔ 에이전트 | **A2A** | Agent Card, 태스크 생명주기, 협상. 조직 간 연동 | LF 이관으로 벤더 중립. 1.0은 정황만 |
| 백엔드 ↔ 프론트엔드 | **AG-UI** | SSE 기반 타입드 이벤트(텍스트 델타, 툴 호출 시작/종료, 상태 패치), 양방향 | MS·AWS 채택으로 사실상 표준 후보. 거버넌스는 여전히 스타트업 |
| UI의 내용 | **A2UI** | 선언적 UI 트리(JSON). 실행 코드가 아닌 데이터 → 웹·Flutter·Android·iOS 네이티브 렌더 주장 | v0.9. 컴포넌트 세트 개명 등 **호환성 깨지는 변경** 진행 중 |
| 툴 서버 → 호스트 UI | **MCP Apps** | 서버가 HTML UI 반환, 호스트가 샌드박스 iframe 렌더 | MCP 2026-07-28 스펙의 **확장으로 편입** — 독립 프로토콜에서 MCP의 일부로 |

### "경쟁에서 조합으로"의 구체적 형태

- **AG-UI와 A2UI는 층이 다르다**: A2UI 트리를 AG-UI 이벤트에 실어 보낼 수 있다 `[2차]`. 전송과 내용의 분리
- **A2A와 MCP는 보완**: 에이전트를 "툴처럼" MCP로 부를 수도 있지만, 태스크 생명주기·협상·장기 실행이 필요하면 A2A
- **MCP Apps는 UI의 출처가 다르다**: AG-UI/A2UI는 **에이전트**가 UI를 결정, MCP Apps는 **툴 서버**가 자기 UI를 가져온다. 서드파티 툴이 위젯을 가진 채 호스트에 꽂히는 모델 ([generative-ui](generative-ui.md))

## 벤더 주장 vs 검증된 사실

| 주장 | 출처 | 검증 수준 | 우리의 판단 |
|---|---|---|---|
| MCP 2026-07-28 개정으로 무상태 코어·Multi Round-Trip·Tasks·Apps가 들어왔다 | 해설 블로그(blog.mcpservers.org, Stacklok, agentpedia) | `[2차]` | 개정 **전**에 작성된 소스가 섞여 있다. **공식 changelog 재확인 전까지 세부 항목은 잠정**. 무상태화 방향은 복수 소스 일치 |
| A2A가 Linux Foundation으로 이관, 100+ 기업 참여 | 기술 매체 복수 (2025-06) | `[2차]` | 복수 소스 일치. 벤더 중립 근거로 채택 |
| A2A 1.0 도달 | A2UI 자료의 "A2A 1.0 통합" 언급 | `[2차]` 정황 | **전용 발표 미확인**. "1.0"을 전제로 설계하지 않는다 |
| A2UI는 실행 코드가 아닌 데이터라 웹·Flutter·Android·iOS에서 네이티브 렌더된다 | Google Developers Blog (v0.9, 2026-04-17) | `[벤더]` | 구조적으로 가능한 주장. 그러나 v0.9에서 컴포넌트 세트 이름이 바뀌는 등 **안정성 미확보**. 렌더러별 동일성은 `[실측]` 필요 |
| AG-UI는 Microsoft .NET SDK(2026-09-25), AWS AgentCore(2026-03) 지원으로 산업 표준이 되고 있다 | InfoWorld, Microsoft, AWS 발표 | `[2차]` `[벤더]` | 대형 벤더 채택은 사실. 그러나 **거버넌스는 CopilotKit 단일 스타트업**. "표준"이 아니라 "사실상 표준 후보" |
| MCP C# SDK v2.0은 하위 호환이다 | Microsoft 발표 (2026-07-28) | `[벤더]` | 구 서버와의 혼재 기간 호환 문제는 `[실측]` 전까지 가정하지 않는다 |
| 프로토콜은 보완 관계다(MCP·A2A·AG-UI) | 해설 다수 | `[2차]` | 채택. 단 "셋 다 필요"가 아니라 "경계마다 판단" |

## 이 위키의 어느 단계에 적용되나

| 단계 페이지 | 역할 | 적용 조건 |
|---|---|---|
| [Part 1/05 에이전트 프로토콜](../01-foundations/05-agentic-protocols.md) | 다섯 프로토콜의 정의·비교표·선택 가이드(기반 지식) | 항상 참조 |
| [Part 3/09 툴과 MCP 통합](../03-agent-workflow-rag/09-tools-and-mcp.md) | MCP 2026-07-28 대응: 무상태 코어에서의 세션 상태 설계, Tasks로 장기 실행 | MCP 채택 |
| [Part 3/07 멀티 에이전트 오케스트레이션](../03-agent-workflow-rag/07-multi-agent-orchestration.md) | 내부 에이전트는 프레임워크, 조직 간은 A2A | 당사자 둘 이상 |
| [Part 4/02 AG-UI · A2UI · MCP Apps 상세 비교](../04-dynamic-ui/02-protocols-ag-ui-a2ui-mcp-apps.md) | UI 층 세 프로토콜의 선택·조합 | Generative UI 채택 |
| [Part 4/04 스트리밍과 상태 동기화](../04-dynamic-ui/04-streaming-state-sync.md) | AG-UI 이벤트(델타·상태 패치)의 구현 | 스트리밍 UI |
| [Part 4/08 보안·샌드박싱](../04-dynamic-ui/08-security-sandboxing.md) | MCP Apps iframe 샌드박스, A2UI 트리 검증 | MCP Apps / A2UI 채택 |
| [Part 5/01 보안·프롬프트 인젝션](../05-cross-cutting/01-security-prompt-injection.md) | 프로토콜은 보안을 정의하지 않음. MCP 서버·A2A 엔드포인트의 인증·권한·감사 | 모든 프로토콜 |

## 한계와 미해결 질문

| 한계 / 질문 | 왜 문제인가 | 현재 상태 |
|---|---|---|
| **AG-UI 거버넌스** | 재단이 아닌 스타트업(CopilotKit) 주도. 로드맵·라이선스 변경 위험 | MIT 라이선스로 포크 가능. MS·AWS 채택이 완화 요인이나 구조적 해결은 아님 |
| **MCP 대개정 전환기** | 구·신 서버 혼재, SDK 간 호환 | 2026-07-28 지원 SDK로 통일 + 호환 레이어 |
| **세션 상태의 책임 이동** | 무상태 코어에서 "누가 상태를 드나"를 호스트·서버 중 재설계 | Tasks 확장 + 애플리케이션 상태 저장소 |
| **UI 프로토콜 변경 빈도** | A2UI v0.9 개명 등 반년 단위 호환성 파괴 | 버전 고정, 어댑터 계층 |
| **A2A 1.0 불확실** | 정황만 있고 발표 없음 | 재확인 필요 |
| **보안 미정의** | 프로토콜이 인증·권한·감사를 정의하지 않아 새 공격면 | 별도 설계 필수 |
| **프로토콜 과잉** | 경계 하나인 시스템에 다섯 프로토콜 | "당사자 둘 이상" 기준 |
| 공식 changelog 미확인 | MCP 개정 세부가 2차 소스 기반 | 확인 후 검증 수준 조정 |

## 우리 프로젝트에서의 채택 기준

프로토콜별로 기준이 다르다. **MCP·A2A**는 경계가 확인되면 적용하고, **UI 세 프로토콜**은 [generative-ui](generative-ui.md)의 기준을 따른다.

1. **경계 확인**: 각 프로토콜을 쓰려는 경계에 **당사자가 둘 이상**인지 문서화한다. 하나뿐이면 쓰지 않는다 (함수 직접 등록 / 프레임워크 내장 스트리밍 / 단일 에이전트).
2. **MCP 2026-07-28 호환 `[실측]`**: 우리가 쓰는 호스트·서버·SDK가 개정 스펙으로 상호 운용되는지, 구 서버와의 혼재 시 어떤 실패가 나는지 테스트. 세션 상태의 저장 위치가 설계 문서에 명시되어 있다.
3. **공식 스펙 대조**: MCP 개정 세부(무상태·Multi Round-Trip·Tasks·Apps)를 공식 changelog로 확인해 이 페이지의 `[2차]`를 조정한다.
4. **A2A 적용 시**: 조직 간 연동 상대가 A2A를 지원하는지 확인. 1.0 여부와 무관하게 사용 버전을 고정한다.
5. **보안 설계 완료**: MCP 서버·A2A 엔드포인트의 인증·권한·감사가 프로토콜 바깥에서 설계·구현·테스트되어 있다 ([Part 5/01](../05-cross-cutting/01-security-prompt-injection.md)).
6. **어댑터 계층**: 프로토콜 호출이 어댑터 뒤에 있어 버전 변경·교체가 국소화된다. 특히 AG-UI는 거버넌스 리스크 때문에 필수.
7. **UI 프로토콜**: [generative-ui](generative-ui.md) 채택 기준 참조.

## 참고 문헌

- `[2차]` MCP 2026-07-28 개정 해설: blog.mcpservers.org, Stacklok 엔터프라이즈 가이드, agentpedia. RC 2026-05-21, 최종 2026-07-28. 공식 changelog(modelcontextprotocol.io) 재확인 권장
- `[벤더]` Microsoft, MCP C# SDK v2.0 발표 (2026-07-28)
- `[2차]` A2A Linux Foundation 이관: BleepingComputer, SD Times, heise (2025-06)
- `[2차]` AG-UI: InfoWorld; Microsoft .NET SDK (2026-09-25); AWS Bedrock AgentCore AG-UI 지원 (2026-03); ThoughtWorks Technology Radar
- `[벤더]` Google Developers Blog, A2UI v0.9 (2026-04-17); InfoQ (2026-07)
- `[2차]` The New Stack, "Agent UI standards multiply: MCP Apps and Google's A2UI"; CopilotKit, "AG-UI and A2UI explained" — CopilotKit은 AG-UI 주도사
- `[2차]` MCP Apps CopilotKit 지원 발표 (2026-01)
- 관련 트렌드: [generative-ui](generative-ui.md)

_검증 수준 표기: `[검증]` `[벤더]` `[2차]` `[실측]` — 규칙은 [작성 가이드](../_meta/writing-guide.md) 참조. 이 페이지의 모든 날짜·수치는 [조사 노트](../_meta/research-notes-2026-10.md)와 동일하다._
