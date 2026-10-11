---
title: "Part 0. Overview — 전체 조망"
part: 00-overview
stage: all
tags: [overview]
maturity: 채택
status: review
updated: 2026-10-11
---

# Part 0. Overview — 전체 조망

> 이 Part는 위키 전체를 **한 장의 그림, 한 그루의 결정 트리, 한 장의 레이더**로 압축한다. 세부 Part로 들어가기 전에 여기서 "내 프로젝트가 어디에 해당하는지"를 먼저 찾는다.

## 이 Part의 역할

| 질문 | 답 |
|---|---|
| **무엇을** | AI Agent 시스템의 전체 구조, 기술 선택의 출발점, 최근 1년 트렌드의 위치 |
| **언제** | 프로젝트 착수 시, 또는 특정 단계에서 길을 잃었을 때 |
| **어떻게** | 레퍼런스 아키텍처에서 내 시스템의 범위를 표시 → 결정 트리로 각 단계의 기본값 결정 → 레이더에서 시험해 볼 트렌드 선택 |
| **왜** | 단계별 기술을 개별 최적화하면 레이어 간 정합성(근거 데이터 ↔ 생성 ↔ UI 표시)이 깨진다. 전체를 먼저 봐야 한다 |

## 챕터

| 페이지 | 내용 | 시각화 |
|---|---|---|
| [01 레퍼런스 아키텍처](01-reference-architecture.md) | Knowledge / Reasoning·Action / Interaction 3계층과 횡단 관심사. 각 단계 → 담당 페이지 매핑 | [reference-architecture.html](../assets/html/reference-architecture.html) |
| [02 종합 기술 선택 결정 트리](02-decision-guide.md) | 요구사항 변수(데이터 유형, 규모, 지연, 근거 요구, 규제, 팀 역량)에서 출발해 단계별 기본값에 도달하는 질문 트리 | decision-tree.html (예정) |
| [03 트렌드 레이더 2026](03-trend-radar-2026.md) | 2025-10 ~ 2026-10 트렌드를 채택 / 시험 / 관찰 / 보류로 배치, 각 트렌드가 적용되는 단계 링크 | trend-radar.html (예정) |
| [04 용어집](04-glossary.md) | 한/영 병기 용어 정의 | — |

## 3-Layer 모델 요약

```
┌──────────────────────────────────────────────────────────┐  ┐
│ Interaction   의도→UI 매핑 · 컴포넌트 레지스트리 · 스트리밍 · HITL UI │  │
│               (AG-UI / A2UI / MCP Apps)                   │  │
├──────────────────────────────────────────────────────────┤  │ Cross-cutting
│ Reasoning /   질의 이해·라우팅 → 검색·리랭킹 → 생성·근거 강제 │  │ 보안 · 비용 ·
│ Action        → 툴/MCP → 가드레일                          │  │ 관측성 ·
│               [결정 레이어: System One] [뉴로심볼릭 검증기]  │  │ 거버넌스 ·
│               [메모리·컨텍스트] [오케스트레이션]              │  │ LLMOps
├──────────────────────────────────────────────────────────┤  │
│ Knowledge     수집·파싱 → 청킹 → 메타·출처 → 임베딩·인덱싱    │  │
│               → KG 구축 → 온톨로지 → 근거 데이터 → 동기화·ACL │  │
└──────────────────────────────────────────────────────────┘  ┘
```

레이어 사이의 계약이 핵심이다.

- **Knowledge → Reasoning**: "근거 단위(evidence)"의 스키마. 청크인가, 트리플인가, 둘 다인가. [근거 데이터 설계](../02-data-pipeline/08-grounding-data-design.md)가 정의하고 [생성과 근거 강제](../03-agent-workflow-rag/04-generation-grounding.md)가 소비한다
- **Reasoning → Interaction**: "렌더 가능한 응답"의 스키마. 자유 텍스트인가, 컴포넌트 선택인가, 선언적 UI 트리인가. [의도→UI 매핑](../04-dynamic-ui/05-intent-to-ui-mapping.md)과 [컴포넌트 레지스트리](../04-dynamic-ui/03-component-registry-schema.md)가 정의한다
- **Interaction → Reasoning**: 사용자 행위 이벤트(승인, 수정, 선택)가 에이전트 상태로 돌아오는 경로. [스트리밍과 상태 동기화](../04-dynamic-ui/04-streaming-state-sync.md)

## 트렌드 세 축이 레이어에 걸치는 방식

| 트렌드 | Knowledge | Reasoning / Action | Interaction |
|---|---|---|---|
| **온톨로지 / KG** | 근거 데이터의 구조, 엔티티 해소, 시맨틱 레이어 | 제약 기반 검증, 멀티홉 추론, 설명 생성 | 도메인 개념 → UI 컴포넌트 매핑 |
| **뉴로심볼릭** | 추출 결과의 규칙 검증(SHACL 등) | LLM 출력의 기호 검증기, 규칙엔진 결합 | 렌더 전 UI 트리의 제약 검증 |
| **결정 모델 (Jev / Decisions API)** | 문서 분류, 품질 판정, 중복 판정을 저비용으로 | intent 라우팅, 게이팅, 다음 행동 선택, 승인 필요 여부 판정 | 어떤 컴포넌트를 띄울지 선택 |

## 관련 페이지

- [작성 가이드](../_meta/writing-guide.md) — 이 위키의 규칙
- [Part 7. 시나리오](../07-appendix-scenarios/00-overview.md) — 이 조망을 실제 프로젝트에 적용한 예
