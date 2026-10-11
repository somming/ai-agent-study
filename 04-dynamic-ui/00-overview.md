---
title: "Part 4. Agent 기반 동적 UI 연동 설계"
part: 04-dynamic-ui
stage: interaction
tags: [overview, generative-ui, ag-ui, a2ui, mcp-apps]
maturity: 시험
status: draft
updated: 2026-10-11
---

# Part 4. Agent 기반 동적 UI 연동 설계

> **시각화**: `assets/html/dynamic-ui-architecture.html` (예정)
> Interaction Layer. 에이전트의 출력을 **텍스트가 아닌 UI로**, 사용자의 행위를 **프롬프트가 아닌 이벤트로** 주고받는 설계. 2026년 현재 이 레이어는 프로토콜이 수렴 중이라 Part 2·3보다 성숙도가 낮다. 성숙도 표기를 특히 유의해서 읽는다.

## 이 Part의 역할

| 질문 | 답 |
|---|---|
| **무엇을** | 에이전트 응답 → UI 컴포넌트 선택·조립·렌더 → 사용자 이벤트 → 에이전트 상태 복귀의 전체 루프 |
| **언제** | 사용자가 결과를 "읽는" 것이 아니라 "조작"해야 할 때(선택, 비교, 승인, 수정, 폼 입력). 순수 QA는 citation 렌더링(06 일부)만 필요 |
| **어떻게** | 01에서 통제 수준(Controlled / Declarative / Open-ended) 결정 → 02에서 프로토콜 → 03에서 컴포넌트 레지스트리 → 05에서 의도→UI 매핑 방식 → 04·06으로 스트리밍·HITL → 08 보안 → 10 평가 |
| **왜** | 텍스트 응답은 사용자가 다시 읽고 다시 입력해야 한다. UI는 그 왕복을 줄이지만, 에이전트가 UI를 "그릴" 자유를 줄수록 일관성·보안·접근성이 무너진다. 통제 수준이 핵심 결정이다 |

## 단계와 각 단계의 기술 선택지

| 순서 | 단계 | 페이지 | 이 단계에서 비교하는 기술 |
|---|---|---|---|
| 1 | GenUI 스펙트럼 | [01](01-generative-ui-spectrum.md) | **Controlled**(사전 정의 컴포넌트를 에이전트가 선택) / **Declarative**(에이전트가 UI 트리를 JSON으로 기술) / **Open-ended**(에이전트가 HTML·코드 생성) |
| 2 | 프로토콜 | [02](02-protocols-ag-ui-a2ui-mcp-apps.md) | AG-UI(이벤트 스트림, SSE) vs A2UI(Google, 선언적 스펙, 멀티플랫폼 렌더) vs MCP Apps(MCP 서버가 샌드박스 iframe UI 반환) vs Vercel AI SDK RSC vs 자체 프로토콜 |
| 3 | 컴포넌트 레지스트리 | [03](03-component-registry-schema.md) | JSON Schema 기반 카탈로그, 타입 생성(zod/pydantic), 디자인 시스템 토큰 연결, 버전 관리 |
| 4 | 스트리밍·상태 동기화 | [04](04-streaming-state-sync.md) | SSE vs WebSocket; 전체 상태 vs JSON Patch; optimistic UI; 부분 렌더 |
| 5 | 의도→UI 매핑 | [05](05-intent-to-ui-mapping.md) | LLM 툴 호출로 컴포넌트 선택 vs **결정 모델**로 선택 vs 온톨로지(도메인 개념→컴포넌트) 매핑 vs 규칙 |
| 6 | HITL UI | [06](06-hitl-ui-patterns.md) | 승인 카드, 인라인 수정, diff 뷰, 피드백 캡처, citation 렌더 |
| 7 | 프론트엔드 통합 | [07](07-frontend-integration.md) | React/Next.js + Vercel AI SDK, CopilotKit, Flutter(A2UI), 네이티브, 기존 앱 임베딩 |
| 8 | 보안·샌드박싱 | [08](08-security-sandboxing.md) | iframe 샌드박스, CSP, UI 경유 prompt injection, 이벤트 권한 검증 |
| 9 | UX·접근성 | [09](09-ux-accessibility.md) | 일관성 유지, 로딩·실패 상태, 접근성 트리, 다국어 |
| 10 | 평가 | [10](10-evaluation-genui.md) | UI 선택 정확도, 렌더 실패율, 과업 완료율, 사용자 수정률 |
| — | 결정 매트릭스 | [99](99-decision-matrix.md) | 요구사항 × 패턴 × 프로토콜 종합 |

## 이 Part를 관통하는 설계 원칙

1. **기본값은 Controlled다.** 프레임워크 기본값이 그렇기도 하지만, 디자인 시스템·접근성·보안을 지키면서 운영 가능한 유일한 패턴이기 때문이다. Declarative는 컴포넌트 조합이 폭발적으로 필요할 때, Open-ended는 샌드박스 안에서만
2. **"어떤 UI를 띄울지"는 판정 문제다.** 생성 모델의 툴 호출로도 되지만, 선택지가 고정된 컴포넌트 카탈로그라면 결정 모델이나 규칙이 더 빠르고 싸고 예측 가능하다
3. **UI 트리도 검증한다.** 에이전트가 만든 컴포넌트 트리는 스키마 검증 + 제약 검증(뉴로심볼릭 관점: 필수 필드, 허용 조합, 권한)을 거쳐 렌더한다
4. **사용자 이벤트는 신뢰 경계를 넘는다.** UI에서 돌아온 이벤트도 외부 입력이다. 서버에서 권한·스키마를 다시 검증한다
5. **citation 렌더링은 Part 2의 스키마를 그대로 쓴다.** UI 레이어에서 근거 형식을 재해석하지 않는다

## 최근 1년 트렌드가 이 Part에 미친 영향

| 트렌드 | 영향 받는 단계 | 요지 | 검증 수준 |
|---|---|---|---|
| 3패턴 정착 | 01 | CopilotKit이 Controlled / Declarative / Open-ended로 정리, 각각 AG-UI / A2UI·Open-JSON-UI / MCP Apps에 대응 | `[2차]` — CopilotKit 자체 프레이밍 |
| MCP Apps | 02 | MCP 확장으로 서버가 샌드박스 iframe UI 반환. CopilotKit 2026-01 지원 발표 | `[벤더]` |
| A2UI | 02, 07 | Google의 선언적 UI 스펙(Apache 2.0). 실행 코드가 아닌 데이터 전송이라 웹·Flutter·Android·iOS 네이티브 렌더 가능 주장 | `[벤더]` |
| AG-UI | 02, 04 | 프론트-백 양방향 이벤트 프로토콜(MIT). 툴 호출 시작/종료, 상태 패치를 SSE로 | `[2차]` |
| 결정 모델 | 05 | 컴포넌트 선택을 Jev/Decisions API 류로 판정하는 배치. 아직 사례 보고 드묾 | `[2차]` — 적용 사례 미확인 |
| 온톨로지 기반 UI 매핑 | 05 | 도메인 개념(상품, 주문, 인시던트)을 컴포넌트에 매핑하는 시맨틱 레이어 활용 | `[2차]` |

## 다른 Part와의 계약

- **Part 3 → Part 4**: 응답 스키마에 `ui_intent`(선택적)와 `citations`가 포함되어야 한다. 05가 이를 컴포넌트로 바꾼다
- **Part 4 → Part 3**: 사용자 이벤트(승인/거절/수정/선택)는 에이전트 상태 머신의 입력 이벤트로 정의된다. [Part 3/12 가드레일·HITL](../03-agent-workflow-rag/12-guardrails-hitl.md)과 함께 설계한다
- **Part 2 → Part 4**: citation 스키마 [Part 2/08](../02-data-pipeline/08-grounding-data-design.md)

## 관련 페이지

- [Part 1/05 에이전트 프로토콜](../01-foundations/05-agentic-protocols.md)
- [Part 6/generative-ui](../06-trends/generative-ui.md), [Part 6/agentic-protocols-convergence](../06-trends/agentic-protocols-convergence.md)
- [시나리오 B: 커머스 상담 + Generative UI](../07-appendix-scenarios/02-commerce-assistant-genui.md)
