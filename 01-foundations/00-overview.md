---
title: "Part 1. Foundations — 공통 기반"
part: 01-foundations
stage: all
tags: [overview, foundations]
maturity: 채택
status: draft
updated: 2026-10-11
---

# Part 1. Foundations — 공통 기반

> Part 2~4의 모든 페이지가 전제로 삼는 지식. 여기 없는 개념은 뒤 Part에서 설명 없이 쓴다.

## 이 Part의 역할

| 질문 | 답 |
|---|---|
| **무엇을** | LLM 동작 특성, 결정 모델과 생성 모델의 구분, 지식 표현 체계, 뉴로심볼릭 결합 패턴, 에이전트 프로토콜 지형, 평가 체계 |
| **언제** | 프로젝트 초반 팀 공통 어휘를 맞출 때. 뒤 Part에서 모르는 개념이 나올 때 |
| **어떻게** | 순서대로 읽는다. 특히 02(System 1/2)와 03(지식 표현)은 이 위키의 트렌드 세 축을 이해하는 열쇠 |
| **왜** | "온톨로지"를 데이터 스키마로만 이해하거나 "결정 모델"을 작은 LLM으로 오해하면 뒤의 설계 선택이 전부 어긋난다 |

## 챕터

| 페이지 | 핵심 질문 | 뒤 Part에서 쓰이는 곳 |
|---|---|---|
| [01 LLM 기초](01-llm-fundamentals.md) | 컨텍스트 윈도우·구조화 출력·함수 호출이 설계에 주는 제약은? 모델 라우팅의 기본 원칙은? | Part 3 전체, Part 5/02 |
| [02 System 1 vs System 2](02-system1-vs-system2.md) | 왜 "판정"과 "생성"을 다른 모델에 맡기는가? 결정 모델(Jev, Decisions API)은 LLM과 어떻게 다른가? | Part 2/09, Part 3/02·11·12, Part 4/05 |
| [03 지식 표현](03-knowledge-representation.md) | 택소노미·온톨로지·지식그래프의 차이는? RDF/OWL/SHACL과 Property Graph 중 무엇을 고르나? | Part 2/05·06·08, Part 3/10 |
| [04 뉴로심볼릭 AI](04-neuro-symbolic-ai.md) | 신경망과 기호 추론을 결합하는 패턴은 몇 가지이고 각각 언제 쓰나? | Part 2/09, Part 3/04·10·12 |
| [05 에이전트 프로토콜](05-agentic-protocols.md) | MCP·A2A·AG-UI·A2UI·MCP Apps는 각각 어느 레이어를 맡나? 경쟁인가 보완인가? | Part 3/07·09, Part 4/02 |
| [06 평가와 관측성 기초](06-evaluation-observability.md) | 골든셋·LLM-as-judge·트레이싱을 어떤 순서로 갖추나? | Part 2~4의 평가 페이지, Part 5/03 |

## 이 Part에서 비교하는 기술

| 챕터 | 비교 대상 |
|---|---|
| 01 | 구조화 출력 방식: JSON mode vs 함수 호출 vs 제약 디코딩(grammar) vs 결정 모델 |
| 02 | 판정 수단: 프롬프트된 LLM vs 소형 분류기(파인튜닝) vs 결정 모델(Jev, Decisions API) vs 규칙 |
| 03 | 지식 표현: 택소노미(SKOS) vs 온톨로지(OWL) vs Property Graph(LPG) vs 관계형 스키마; 제약 언어: SHACL vs ShEx vs 애플리케이션 코드 |
| 04 | 뉴로심볼릭 결합: Symbolic→Neural, Neural→Symbolic, Neural[Symbolic], Symbolic[Neural], Neural∪Symbolic 등 Kautz 분류 |
| 05 | 프로토콜: MCP(툴·데이터) vs A2A(에이전트 간) vs AG-UI(프론트-백 이벤트) vs A2UI(선언적 UI) vs MCP Apps(샌드박스 앱) |
| 06 | 평가: 규칙 기반 vs 참조 기반(정답셋) vs LLM-as-judge vs 사람 평가; 트레이싱: OTel GenAI 컨벤션 vs 벤더 전용 SDK |

## 최근 1년 트렌드가 이 Part에 미친 영향

- **결정 모델의 등장**(2026-09 Jev, 2026-09~10 OpenAI Decisions API)으로 02 챕터가 신설됐다. 이전에는 "작은 LLM으로 분류"가 유일한 선택지였다 `[2차]`
- **온톨로지의 재부상**: Gartner가 2026-04 보고서에서 뉴로심볼릭 AI(NSAI) 벤더를 식별했다는 벤더 발표 `[벤더]`, 온톨로지 3계층으로 650+ 에이전트를 운영했다는 arXiv 2604.00555 `[검증-단일저자]`. 03·04 챕터의 비중이 커졌다
- **프로토콜 수렴**: MCP Apps(2026-01), A2UI, AG-UI가 역할을 나눠 가지면서 05 챕터가 "비교"에서 "조합"으로 바뀌었다 `[2차]`

## 관련 페이지

- [Part 6. Trends](../06-trends/00-overview.md) — 위 트렌드의 심화
