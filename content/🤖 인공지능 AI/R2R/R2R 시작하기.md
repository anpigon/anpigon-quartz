---
title:
aliases: []
related:
created: 2025-07-02 05:27:32
modified: 2025-07-02 09:34:51
tags: [튜토리얼, Python, R2R]
---

> 원문: https://r2r-docs.sciphi.ai/self-hosting/quickstart

# 빠른 시작 (Quickstart)

R2R 시작하기는 쉽습니다.

### 1. 배포 확인 (Deployment Checks)

먼저 R2R 인스턴스가 로컬에 올바르게 배포되었는지 확인합니다:

```shell
curl http://localhost:7272/v3/health
# {"results":{"response":"ok"}}
```

### SDK 설치 (Install the SDK)

R2R은 Python 및 JavaScript SDK를 제공하여 상호 작용할 수 있습니다.

###### Python

```shell
pip install r2r
```

###### JavaScript

```shell
npm i r2r-js
```

### 파일 수집 (Ingesting files)

R2R에 파일을 수집하면, 서버는 작업을 수락하고, 파일을 처리 및 청크로 분할하며, 문서 요약을 생성합니다.

###### Python
```python
client.documents.create_sample(hi_res=True)
# 자체 문서를 수집하려면 `client.documents.create(file_path="/path/to/file")`을 사용하세요
```

###### JavaScript
```js
clients.documents.createSample({ ingestionMode: "hi-res" })
// 자체 문서를 수집하려면 `client.documents.create({filePath: </path/to/file>})`을 사용하세요
```

예시 출력:
```shell
IngestionResponse(message='Document created and ingested successfully.', task_id=None, document_id=UUID('e43864f5-a36f-548e-aacd-6f8d48b30c7f'))
```

### 파일 상태 확인 (Getting file status)

파일 수집이 완료되면, 문서를 목록화하여 상태를 확인할 수 있습니다.

###### Python
```python
client.documents.list()
```

###### JavaScript
```javascript
clients.documents.list()
```

###### Curl
```shell
curl -X GET https://api.sciphi.ai/v3/documents \
  -H "Content-Type: application/json"
```

예시 출력:
```json
[
  DocumentResponse(
    id=UUID('e43864f5-a36f-548e-aacd-6f8d48b30c7f'),
    collection_ids=[UUID('122fdf6a-e116-546b-a8f6-e4cb2e2c0a09')],
    owner_id=UUID('2acb499e-8428-543b-bd85-0d9098718220'),
    document_type=<DocumentType.PDF: 'pdf'>,
    metadata={'title': 'DeepSeek_R1.pdf', 'version': 'v0'},
    version='v0',
    size_in_bytes=1768572,
    ingestion_status=<IngestionStatus.SUCCESS: 'success'>,
    extraction_status=<GraphExtractionStatus.PENDING: 'pending'>,
    created_at=datetime.datetime(2025, 2, 8, 3, 31, 39, 126759, tzinfo=TzInfo(UTC)),
    updated_at=datetime.datetime(2025, 2, 8, 3, 31, 39, 160114, tzinfo=TzInfo(UTC)),
    ingestion_attempt_number=None,
    summary="이 문서는 DeepSeek-AI가 개발한 추론 모델 시리즈인 DeepSeek-R1에 대한 포괄적인 개요를 담고 있으며, DeepSeek-R1-Zero와 DeepSeek-R1을 포함합니다. DeepSeek-R1-Zero는 지도 미세 조정 없이 대규모 강화 학습(RL)을 활용하여 인상적인 추론 능력을 보여주지만, 가독성 및 언어 혼합과 같은 문제에 직면합니다. 성능 향상을 위해 DeepSeek-R1은 다단계 훈련과 콜드 스타트 데이터를 통합하여 다양한 추론 작업에서 OpenAI 모델과 유사한 결과를 달성합니다. 이 문서는 모델의 훈련 과정, 여러 벤치마크에 걸친 평가 결과, 그리고 추론 능력을 유지하면서 더 작고 효율적인 증류 모델의 도입을 상세히 설명합니다. 또한 현재 모델의 한계(예: 언어 혼합 및 프롬프트 민감도)를 논의하고, 소프트웨어 엔지니어링 작업에서 일반적인 능력과 효율성을 개선하기 위한 향후 연구 방향을 제시합니다. 이 연구 결과는 대규모 언어 모델에서 추론 능력을 개발하는 데 있어 RL의 잠재력과 소규모 모델을 위한 증류 기술의 효과를 강조합니다.", summary_embedding=None, total_tokens=29673)] total_entries=1
  ), ...
]
```

### 검색 실행 (Executing a search)

검색 쿼리를 수행합니다:

###### Python
```python
client.retrieval.search(
  query="What is DeepSeek R1?",
)
```

###### JavaScript
```javascript
client.retrieval.search({
  query: "What is DeepSeek R1?",
})
```

###### Curl
```shell
curl -X POST https://api.sciphi.ai/v3/retrieval/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is DeepSeek R1?"
  }'
```

검색 쿼리는 기본적인 유사도 검색을 사용하여 가장 관련성 높은 문서를 찾습니다. 사용 사례에 따라 [하이브리드 검색](https://r2r-docs.sciphi.ai/documentation/hybrid-search) 또는 [그래프 검색](https://r2r-docs.sciphi.ai/documentation/graphs)과 같은 고급 검색 방법을 사용할 수 있습니다.

예시 출력:
```json
AggregateSearchResult(
  chunk_search_results=[
    ChunkSearchResult(
      score=0.643,
      text="문서 제목: DeepSeek_R1.pdf
      텍스트: 70% 이상의 정확도를 달성할 수 있었습니다.
      DeepSeek-R1은 또한 모델이 형식 지침을 따르는 능력을 평가하기 위해 설계된 벤치마크인 IF-Eval에서도 인상적인 결과를 제공합니다. 이러한 개선은 지도 미세 조정(SFT) 및 RL 훈련의 최종 단계에서 지침 준수 데이터가 포함된 것과 관련될 수 있습니다. 또한 AlpacaEval2.0 및 ArenaHard에서 놀라운 성능이 관찰되었으며, 이는 DeepSeek-R1이 쓰기 작업 및 개방형 질문 답변에서 강점을 가지고 있음을 나타냅니다. DeepSeek-V3를 크게 능가하는 성능은 대규모 RL의 일반화 이점을 강조하며, 이는 추론 능력을 향상시킬 뿐만 아니라 다양한 도메인에서 성능을 개선합니다. 더욱이 DeepSeek-R1이 생성하는 요약 길이는 간결하며, ArenaHard에서 평균 689 토큰, AlpacaEval 2.0에서 2,218 문자를 기록합니다. 이는 DeepSeek-R1이 GPT 기반 평가에서 길이 편향을 도입하는 것을 피하며, 여러 작업에서 견고성을 더욱 공고히 함을 나타냅니다."
    ), ...
  ],
  graph_search_results=[],
  web_search_results=[],
  context_document_results=[]
)
```

### RAG (Retrieval-Augmented Generation)

RAG 응답을 생성합니다:

###### Python
```python
client.retrieval.rag(
  query="What is DeepSeek R1?",
)
```

###### JavaScript
```shell
client.retrieval.rag({
  query: "What is DeepSeek R1?",
})
```

###### Curl
```shell
curl -X POST https://api.sciphi.ai/v3/retrieval/rag \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is DeepSeek R1?"
  }'
```

예시 출력:
```json
RAGResponse(
  generated_answer='DeepSeek-R1은 강화 학습(RL)과 지도 미세 조정(SFT)을 활용하여 기능을 향상시키고 다양한 작업에서 인상적인 성능을 보여주는 모델입니다. 이 모델은 쓰기 작업, 개방형 질문 답변, IF-Eval, AlpacaEval2.0, ArenaHard와 같은 벤치마크에서 뛰어난 성능을 발휘합니다 [1], [2]. DeepSeek-R1은 여러 영역에서 이전 버전인 DeepSeek-V3를 능가하며, 다양한 도메인에서 추론 및 일반화 능력을 보여줍니다 [1]. 또한 SimpleQA와 같은 사실 기반 벤치마크에서 경쟁력 있는 결과를 달성하지만, 안전 RL 제약으로 인해 중국어 SimpleQA 벤치마크에서는 성능이 떨어집니다 [2]. 또한 DeepSeek-R1은 추론 능력을 소규모 모델로 전이시키는 증류 프로세스에 관여하며, 이 소규모 모델들은 벤치마크에서 매우 뛰어난 성능을 보입니다 [4], [6]. 이 모델은 영어와 중국어에 최적화되어 있으며, 향후 업데이트에서 언어 혼합 문제를 해결할 계획입니다 [8].',
  search_results=AggregateSearchResult(
    chunk_search_results=[ChunkSearchResult(score=0.643, text=Document Title: DeepSeek_R1.pdf ...)]
  ),
  citations=[Citation(index=1, rawIndex=1, startIndex=305, endIndex=308, snippetStartIndex=288, snippetEndIndex=315, sourceType='chunk', id='e760bb76-1c6e-52eb-910d-0ce5b567011b', document_id='e43864f5-a36f-548e-aacd-6f8d48b30c7f', owner_id='2acb499e-8428-543b-bd85-0d9098718220', collection_ids=['122fdf6a-e116-546b-a8f6-e4cb2e2c0a09'], score=0.6433466439465674, text='Document Title: DeepSeek_R1.pdf\n\nText: could achieve an accuracy of over 70%.\nDeepSeek-R1 also delivers impressive results on IF-Eval, a benchmark designed to assess a\nmodels ability to follow format instructions. These improvements can be linked to the inclusion\nof instruction-following...]
  metadata={'id': 'chatcmpl-B0BaZ0vwIa58deI0k8NIuH6pBhngw', 'choices': [{'finish_reason': 'stop', 'index': 0, 'logprobs': None, 'message': {'refusal': None, 'role': 'assistant', 'audio': None, 'function_call': None, 'tool_calls': None}}], 'created': 1739384247, 'model': 'gpt-4o-2024-08-06', 'object': 'chat.completion', 'service_tier': 'default', 'system_fingerprint': 'fp_4691090a87', ...}
)
```

### 스트리밍 RAG (Streaming RAG)

스트리밍 RAG 응답을 생성합니다:

###### Python
```shell
from r2r import (
    CitationEvent,
    FinalAnswerEvent,
    MessageEvent,
    SearchResultsEvent,
    R2RClient,
)

client = R2RClient("http://localhost:7272")

result_stream = client.retrieval.rag(
    query="What is DeepSeek R1?",
    search_settings={"limit": 25},
    rag_generation_config={"stream": True},
)

# can also do a switch on `type` field
for event in result_stream:
    if isinstance(event, SearchResultsEvent):
        print("Search results:", event.data)
    elif isinstance(event, MessageEvent):
        print("Partial message:", event.data.delta)
    elif isinstance(event, CitationEvent):
        print("New citation detected:", event.data.raw_index)
    elif isinstance(event, FinalAnswerEvent):
        print("Final answer:", event.data.generated_answer)
```

###### JavaScript

⋯

###### Curl
```shell
curl -X POST https://api.sciphi.ai/v3/retrieval/rag \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is DeepSeek R1?"
  }'
```

예시 출력:
```shell
Search results: id='run_1' object='rag.search_results' data={'chunk_search_results': [{'id': '1e40ee7e-2eef-524f-b5c6-1a1910e73ccc', 'document_id': '652075c0-3a43-519f-9625-f581e7605bc5', 'owner_id': '2acb499e-8428-543b-bd85-0d9098718220', 'collection_ids': ['122fdf6a-e116-546b-a8f6-e4cb2e2c0a09'], 'score': 0.7945216641038179, 'text': 'data, achieving strong performance across various tasks. DeepSeek-R1 is more powerful,\nleveraging cold-start data alongside iterative RL fine-tuning. Ultimately ...
...
Partial message: {'content': [MessageDelta(type='text', text={'value': 'Deep', 'annotations': []})]}
Partial message: {'content': [MessageDelta(type='text', text={'value': 'Seek', 'annotations': []})]}
Partial message: {'content': [MessageDelta(type='text', text={'value': '-R', 'annotations': []})]}
...
Final answer: DeepSeek-R1은 DeepSeek-AI 연구팀이 개발한 대규모 언어 모델입니다. 이 모델은 강화 학습(RL) 전에 다단계 훈련과 콜드 스타트 데이터를 사용하여 훈련된 추론 모델입니다. 이 모델은 MMLU, MMLU-Pro, GPQA Diamond, FRAMES를 포함한 다양한 벤치마크에서 특히 STEM 관련 질문에서 우수한 성능을 보여줍니다. ...
```

### RAG를 활용한 추론 에이전트 (agentic-rag)

R2R 추론 에이전트를 사용하면, 검색 증강 생성(RAG)이 단계별 추론과 결합되어 문서에서 더 높은 품질의 응답을 생성합니다.

###### Python
```python
streaming_response = client.retrieval.agentic-rag(
  message={"role":"user", "content": "What does deepseek r1 imply?"},
  rag_generation_config={
    "stream": True,
    "model": "anthropic/claude-3-5-sonnet-20241022",
  }
)

for chunk in streaming_response:
    print(chunk)
```

###### JavaScript
```javascript
// 1) 스트리밍 RAG 요청 시작
const resultStream = await client.retrieval.rag({
query: "What is DeepSeek R1?",
searchSettings: { limit: 25 },
ragGenerationConfig: { stream: true },
});

// 2) 비동기 이터레이터(스트리밍)를 받았는지 확인
if (Symbol.asyncIterator in resultStream) {
// 2a) 서버에서 오는 각 이벤트를 반복
for await (const event of resultStream) {
    switch (event.event) {
    case "search_results":
        console.log("검색 결과:", event.data);
        break;
    case "message":
        console.log("부분 메시지 델타:", event.data.delta);
        break;
    case "citation":
        console.log("새로운 인용 이벤트:", event.data);
        break;
    case "final_answer":
        console.log("최종 답변:", event.data.generated_answer);
        break;
    // ... 다른 이벤트 유형(예: tool_call / tool_result)이 있다면 케이스 추가
    default:
        console.log("알 수 없거나 처리되지 않은 이벤트:", event);
    }
}
} else {
// 2b) 스트리밍이 활성화되지 않았거나 서버가 SSE를 보내지 않은 경우,
//     단일 응답 객체를 받게 됩니다.
console.log("비스트리밍 RAG 응답:", resultStream);
}
```

예시 출력:
```shell
<Thought>함수 호출: local_search, 페이로드 {"query":"DeepSeek R1"}</Thought>
<Thought>검색 결과는 DeepSeek-R1에 대한 포괄적인 개요를 제공하며, 다양한 벤치마크 및 작업에서의 기능과 성능을 강조합니다. DeepSeek-R1은 DeepSeek-AI가 개발한 추론 모델로, 강화 학습(RL) 및 지침 준수 데이터를 활용하여 성능을 향상시킵니다. 이 모델은 쓰기, 개방형 질문 답변, 사실 기반 쿼리 처리와 같은 작업에서 뛰어난 성능을 발휘합니다. 이 모델은 여러 영역에서 이전 버전인 DeepSeek-V3를 능가하지만, 함수 호출 및 다중 턴 상호 작용과 같은 일부 복잡한 작업에서는 부족합니다. DeepSeek-R1은 또한 교육 작업 및 창의적 글쓰기에서 강력한 성능을 보여주며, 다재다능함과 견고성을 입증합니다. DeepSeek-R1의 주요 특징은 다음과 같습니다:- IF-Eval, AlpacaEval2.0, ArenaHard와 같은 벤치마크에서 인상적인 결과를 달성하여 쓰기 및 질문 답변 능력을 보여줍니다 [출처 1].- 이 모델은 추론 능력을 소규모 모델로 증류하는 데 교사로 사용되며, 이 소규모 모델들도 벤치마크에서 좋은 성능을 보입니다 [출처 2].- SimpleQA와 같은 사실 기반 벤치마크에서 DeepSeek-V3를 능가하지만, 언어 혼합 및 특정 복잡한 작업에서 한계가 있습니다 [출처 3, 5].- DeepSeek-R1은 코딩 작업에서 전문가 수준의 성능을 보여주며, MMLU 및 GPQA Diamond와 같은 교육 벤치마크에서 강력한 결과를 보여줍니다 [출처 6, 9].전반적으로 DeepSeek-R1은 추론 및 지침 준수에 중점을 둔 강력한 모델로, 광범위한 작업에서 경쟁력 있는 성능을 달성합니다.</Thought>
<Response>DeepSeek-R1은 DeepSeek-AI가 개발한 추론 모델로, 쓰기 작업, 개방형 질문 답변, 사실 기반 쿼리 처리에서 강력한 성능을 보이는 것으로 알려져 있습니다. 이 모델은 강화 학습과 지침 준수 데이터를 활용하여 기능을 향상시킵니다. 이 모델은 여러 영역에서 이전 버전인 DeepSeek-V3를 능가하며, 추론 능력을 소규모 모델로 증류하는 데 사용됩니다. 강점에도 불구하고, 함수 호출 및 언어 혼합과 같은 복잡한 작업에서는 한계가 있습니다. 전반적으로 DeepSeek-R1은 다양한 벤치마크에서 경쟁력 있는 성능을 가진 다재다능하고 견고한 모델입니다.
```

## 추가 기능 (Additional Features)

R2R은 문서 관리 및 사용자 경험을 향상시키기 위해 아래와 같은 추가 기능을 제공합니다.

### 그래프 (Graphs)

R2R은 문서 이해 및 검색을 향상시키는 강력한 개체 및 관계 추출 기능을 제공합니다. 이 기능은 R2R 내에서 지식 그래프를 구축하는 데 활용될 수 있습니다. 시스템은 자동으로 개체를 식별하고, 그들 간의 관계를 구축하며, 문서 컬렉션에서 풍부한 지식 그래프를 생성할 수 있습니다.

> [!tip] **[지식 그래프](https://r2r-docs.sciphi.ai/documentation/graphs)**
>
> 문서에서 개체와 관계를 자동으로 추출하여 지식 그래프를 형성합니다.

### 사용자 및 컬렉션 (Users and Collections)

R2R은 완전한 사용자 인증 및 관리 기능을 제공하여, 안전하고 기능이 풍부한 인증 시스템을 구현하거나 선호하는 인증 공급자와 통합할 수 있도록 합니다. 또한, 컬렉션은 사용자 및 문서에 대한 효율적인 접근 제어 및 구성을 가능하게 합니다.

> [!tip] **[사용자 인증 쿡북](https://r2r-docs.sciphi.ai/documentation/user-auth)**
>
> R2R의 내장 인증 기능을 사용하여 사용자 등록, 로그인, 이메일 인증 등을 구현하는 방법을 알아보세요.

> [!tip] **[컬렉션 쿡북](https://r2r-docs.sciphi.ai/documentation/collections)**
>
> R2R에서 세분화된 접근 제어 및 문서 구성을 위해 컬렉션을 생성, 관리 및 활용하는 방법을 알아보세요.

## 다음 단계 (Next Steps)

이제 R2R의 핵심 기능에 대한 기본적인 이해를 마쳤으므로, 더 고급 주제를 탐색할 수 있습니다:

- [문서 수집](https://r2r-docs.sciphi.ai/documentation/documents) 및 [문서 참조](https://r2r-docs.sciphi.ai/api-and-sdks/documents/documents)에 대해 자세히 알아보세요.
- [검색 및 RAG](https://r2r-docs.sciphi.ai/documentation/hybrid-search)와 [검색 참조](https://r2r-docs.sciphi.ai/api-and-sdks/retrieval/retrieval)에 대해 알아보세요.
- [지식 그래프](https://r2r-docs.sciphi.ai/documentation/graphs)와 같은 고급 기술을 시도하고 [그래프 참조](https://r2r-docs.sciphi.ai/api-and-sdks/graphs/graphs)를 참고하세요.
- 애플리케이션 권한을 보호하기 위한 [사용자 인증](https://r2r-docs.sciphi.ai/documentation/user-auth) 및 [사용자 API 참조](https://r2r-docs.sciphi.ai/api-and-sdks/users/users)에 대해 알아보세요.
- 세분화된 접근 제어를 위해 [컬렉션](https://r2r-docs.sciphi.ai/api-and-sdks/collections/collections)을 사용하여 문서를 구성하세요.
