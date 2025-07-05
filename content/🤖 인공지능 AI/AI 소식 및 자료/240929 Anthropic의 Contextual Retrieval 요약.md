---
created: 2024-09-29 09:33:17
modified: 2024-10-01 01:38:11
title: Anthropic의 Contextual Retrieval 요약
tags:
  - Anthropic
  - ContextualRetrieval
  - rag
  - BM25
permlink: ai-anthropic-contextual-retrieval-and
category: kr-dev
dg-publish: true
---

> - 원글: https://www.anthropic.com/news/contextual-retrieval
> - 코드: https://github.com/anthropics/anthropic-cookbook/blob/main/skills/contextual-embeddings/guide.ipynb

AI 모델이 특정 맥락에서 유용하려면 배경 지식에 대한 접근이 필요합니다. 이를 위해 개발자들은 주로 RAG(Retrieval-Augmented Generation)를 사용하여 지식 기반에서 관련 정보를 검색하고 사용자 프롬프트에 추가합니다. 그러나 전통적인 RAG 방식은 정보를 인코딩할 때 문맥을 제거하여 관련 정보를 제대로 검색하지 못하는 문제가 있습니다.

## Contextual Retrieval이란?
Contextual Retrieval은 이러한 문제를 해결하기 위한 방법으로, 두 가지 기술을 사용합니다:

1. **Contextual Embeddings**: 각 텍스트 조각(chunk)에 문맥 정보를 추가하여 임베딩합니다.
2. **Contextual BM25**: BM25 검색 기법에 문맥 정보를 반영합니다.

이를 통해 검색 실패율을 49%까지 감소시킬 수 있으며, 리랭킹(reranking)을 함께 사용하면 최대 67%까지 감소시킬 수 있습니다.

## 구현 방법
1. **프롬프트 개선**: Claude와 같은 언어 모델을 사용하여 각 텍스트 조각에 대한 간결한 문맥 설명을 생성합니다.
2. **문맥 추가 임베딩**: 생성된 문맥을 각 텍스트 조각 앞에 추가한 후 임베딩합니다.
3. **Contextual BM25 적용**: 문맥이 추가된 텍스트로 BM25 인덱스를 구축합니다.
4. **리랭킹 사용**: 검색된 상위 텍스트 조각들을 리랭킹 모델로 평가하여 가장 관련성이 높은 것들을 선택합니다.

## 성능 향상 결과
- **Contextual Embeddings**만으로도 검색 실패율이 35% 감소했습니다.
- **Contextual Embeddings와 Contextual BM25**를 함께 사용하면 검색 실패율이 49% 감소했습니다.
- 여기에 **재랭킹**을 추가하면 최대 67%까지 검색 실패율이 감소했습니다.

## 고려사항
- **청크 분할**: 텍스트를 어떻게 분할하느냐에 따라 성능이 달라질 수 있습니다.
- **임베딩 모델 선택**: 일부 모델은 Contextual Retrieval의 이점이 더 크게 나타납니다.
- **맞춤형 문맥 프롬프트**: 도메인에 특화된 프롬프트를 사용하면 더 나은 결과를 얻을 수 있습니다.
- **청크 수 결정**: 모델에 입력하는 텍스트 조각의 수는 성능에 영향을 미칩니다.

## 결론
Contextual Retrieval은 전통적인 RAG의 한계를 극복하여 대규모 지식 기반에서도 높은 정확도의 정보를 효율적으로 검색할 수 있게 해줍니다. 이는 AI 모델의 응답 성능을 직접적으로 향상시킵니다.

**같이 보면 좋은 글**
- [테디노트님의 Introducing Contextual Retrieval by Anthropic 리뷰](https://share.note.sx/1mr8qv5w#wgZzT3oyG5INDjghpaxfqBFDvjuc+4/Xz5JCVES/lG0)
- [성지코딩님의 ANTHROPIC에서 발표한 RAG성능 향상 꿀팁 정리](https://sjkoding.tistory.com/97)