---
title: "Part 2. AI 기반 데이터 파이프라인 및 근거 데이터 설계"
part: 02-data-pipeline
stage: knowledge
tags: [overview, data-pipeline, grounding]
maturity: 채택
status: review
updated: 2026-10-11
---

# Part 2. AI 기반 데이터 파이프라인 및 근거 데이터 설계

> **시각화**: [`assets/html/data-pipeline-architecture.html`](../assets/html/data-pipeline-architecture.html)
> Knowledge Layer. 에이전트가 **무엇을 근거로** 판단할지를 만드는 파이프라인. 이 Part의 산출물은 "검색 가능한 인덱스"가 아니라 **"답변에 인용할 수 있는 근거 단위"** 다.

## 이 Part의 역할

| 질문 | 답 |
|---|---|
| **무엇을** | 원천 데이터 → 근거 단위(evidence)로 변환하는 파이프라인과, 근거 단위의 스키마·출처·신선도·권한 |
| **언제** | 에이전트가 외부 지식에 의존하는 모든 경우. 모델 지식만으로 충분한 과업(순수 변환·요약)은 생략 가능 |
| **어떻게** | 수집 → 파싱 → 청킹 → 메타데이터 → 임베딩·인덱싱 (→ KG 구축 → 온톨로지 정렬) → 근거 스키마 확정 → 동기화·접근제어 |
| **왜** | RAG 품질 문제의 대부분은 생성이 아니라 **검색 대상 데이터**에서 생긴다. 근거가 없는 답은 환각이고, 출처 없는 근거는 감사 불가다 |

## 파이프라인 단계와 각 단계의 기술 선택지

| 순서 | 단계 | 페이지 | 이 단계에서 비교하는 기술 |
|---|---|---|---|
| 1 | 소스 수집·파싱 | [01](01-source-ingestion.md) | 커넥터(Airbyte/Fivetran류 vs 직접 구현), CDC, 문서 파서(Docling, Unstructured, LlamaParse, 벤더 OCR), 멀티모달 추출(VLM 캡셔닝) |
| 2 | 청킹 | [02](02-chunking-strategies.md) | fixed / recursive / semantic / late chunking / contextual retrieval / parent-child / 문서 구조 기반 |
| 3 | 메타데이터·출처 | [03](03-metadata-provenance.md) | 메타데이터 스키마 설계, W3C PROV, 버전 전략, lineage 도구 |
| 4 | 임베딩·인덱싱 | [04](04-embedding-indexing.md) | dense / sparse(BM25, SPLADE) / hybrid / multi-vector(ColBERT); 벡터DB(pgvector, Qdrant, Weaviate, Milvus, Elasticsearch, 매니지드); 양자화 |
| 5 | 지식그래프 구축 | [05](05-knowledge-graph-construction.md) | LLM 추출 vs 규칙 추출 vs 하이브리드; GraphRAG / LightRAG / HippoRAG 2 인덱싱; 엔티티 해소 |
| 6 | 온톨로지 모델링 | [06](06-ontology-driven-modeling.md) | 상향식(데이터→개념) vs 하향식(도메인→스키마); OWL vs LPG 스키마; SHACL 검증; 기존 표준 온톨로지 재사용 |
| 7 | 신선도·동기화 | [07](07-freshness-and-sync.md) | 전체 재색인 vs 증분 vs 이벤트 기반; 삭제 전파; 중복 제거 |
| 8 | **근거 데이터 설계 ★** | [08](08-grounding-data-design.md) | 근거 단위: 청크 vs 트리플 vs 문장 vs 하이브리드; citation 스키마; 신뢰도 등급; 반증 데이터 |
| 9 | 파이프라인 내부의 AI | [09](09-ai-inside-the-pipeline.md) | 분류·정제·라벨링에 LLM vs 소형 모델 vs 결정 모델(Jev, Decisions API) vs 규칙 |
| 10 | 접근 제어·거버넌스 | [10](10-access-control-governance.md) | 인덱스 분리 vs 메타데이터 필터 vs 쿼리 시 ACL 조인; PII 마스킹 시점 |
| — | 결정 매트릭스 | [99](99-decision-matrix.md) | 위 전체를 데이터 유형 × 요구사항으로 종합 |

## 이 Part를 관통하는 설계 원칙

1. **근거 단위를 먼저 정하고 파이프라인을 거꾸로 설계한다.** 답변에 어떤 형태의 인용이 붙어야 하는지(문서 링크? 문장? 그래프 경로?)가 청킹과 인덱싱 방식을 결정한다
2. **벡터 인덱스와 지식그래프는 택일이 아니다.** 비정형 서술은 벡터, 엔티티 관계·다중 홉은 그래프. 둘을 잇는 것이 온톨로지다
3. **파이프라인 안의 AI 호출은 비용 구조를 바꾼다.** 문서 100만 건을 LLM으로 분류하면 인덱싱 비용이 서빙 비용을 넘는다. 결정 모델·소형 모델·규칙의 자리를 먼저 본다
4. **신선도와 권한은 나중에 붙일 수 없다.** 메타데이터 스키마에 처음부터 넣는다

## 최근 1년 트렌드가 이 Part에 미친 영향

| 트렌드 | 영향 받는 단계 | 요지 | 검증 수준 |
|---|---|---|---|
| 온톨로지 르네상스 | 05, 06, 08 | GraphRAG가 임베딩+그래프 탐색에 머물렀다는 비판과 함께, 온톨로지로 추출 결과를 정렬·검증하는 흐름 | `[2차]` `[벤더]` |
| 뉴로심볼릭 | 05, 09 | LLM 추출 → SHACL/규칙 검증 → 그래프 적재의 2단계 구성 | `[2차]` |
| 결정 모델 (Jev / Decisions API) | 09, 07 | 문서 분류·품질 판정·중복 여부를 토큰 생성 없이 판정. 벤더는 대폭 저렴하다고 주장하나 독립 벤치마크 미확인 | `[벤더]` |
| Late chunking vs Contextual retrieval | 02 | 둘 다 2026 기본값으로 거론. 비교 연구(arXiv 2504.19754)는 contextual이 일관성↑ 비용↑, late가 효율↑ 관련성↓ | `[검증]` |
| LightRAG / HippoRAG 2 | 05 | GraphRAG의 인덱싱 비용 문제에 대한 대안. HippoRAG 2는 최난도 멀티홉(MuSiQue)에서 우위, 다른 벤치마크는 포맷 효과라는 재분석(arXiv 2607.11683) | `[검증]` |
| 하이브리드 검색 기본값화 | 04 | BM25 + dense + RRF가 단독보다 우수하다는 보고가 누적 | `[2차]` |

## 다음 Part로 넘기는 것

이 Part의 출력은 **근거 단위 스키마 + 인덱스(들) + 접근 정책**이다. [Part 3](../03-agent-workflow-rag/00-overview.md)의 검색·생성 단계가 이것을 소비하고, [Part 4](../04-dynamic-ui/00-overview.md)의 근거 표시 UI가 citation 스키마를 그대로 렌더한다.

## 관련 페이지

- [Part 1/03 지식 표현](../01-foundations/03-knowledge-representation.md)
- [Part 6/ontology-renaissance](../06-trends/ontology-renaissance.md), [Part 6/graphrag-evolution](../06-trends/graphrag-evolution.md)
- [시나리오 A: 사내 문서 QA](../07-appendix-scenarios/01-enterprise-document-qa.md) — 이 Part의 선택이 가장 많이 드러나는 예시
