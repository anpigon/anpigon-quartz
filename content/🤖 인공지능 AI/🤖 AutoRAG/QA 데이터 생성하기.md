---
created: 2025-05-03 08:14:51
updated: 2025-05-03 11:33:30
tags:
  - AutoRAG
  - RAG
  - RAG평가
---
> 출처: https://docs.auto-rag.com/data_creation/qa_creation/qa_creation.html

## 1. 질문 생성(Query Generation)

`autorag.data.qa.query.llama_gen_query` 패키지는 LlamaIndex LLM을 사용하여 질문(query)을 생성하는 함수들을 제공합니다. 

이 패키지에서는 다음과 같은 함수들을 제공합니다:

1. `factoid_query_gen`
2. `concept_completion_query_gen`
3. `two_hop_incremental`
4. `custom_query_gen` 

### 각 함수의 상세 설명

#### 1. factoid_query_gen

이 함수는 사실 기반 질문을 생성합니다. 사실 기반 질문은 주어진 문맥에서 직접적인 사실 정보를 묻는 질문입니다.

**기능:**
- 주어진 문서 내용에서 사실 정보를 추출하여 질문 형태로 변환
- 간단하고 직접적인 답변이 가능한 질문 생성
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원

**사용 예시:**
```python
from autorag.data.qa.query.llama_gen_query import factoid_query_gen

new_qa = qa.batch_apply(factoid_query_gen, llm=llm, lang="ko")
```

**시스템 프롬프트:**
```Prompt
당신은 주어진 Text를 '사실 질문'으로 변환하는 AI입니다.

사실 질문(factoid questions)이란 사실적인 정보를 요구하는 질문으로, 쉽게 검증할 수 있는 답변을 필요로 합니다. 일반적으로 예/아니오 답변이나 간단한 설명을 요구하며, 날짜, 이름, 장소 또는 사건과 같은 구체적인 세부사항에 대해 묻는 질문입니다.

사실 질문의 예는 다음과 같습니다:
	•	프랑스의 수도는 어디입니까?
	•	전구를 발명한 사람은 누구입니까?
	•	위키피디아는 언제 설립되었습니까?

지침:
	1.	질문은 반드시 주어진 Text를 기반으로 작성되어야 합니다.
	2.	질문은 Text를 기반으로 가능한 한 구체적으로 작성되어야 합니다.
	3.	Text에서 사실적 정보를 요구하는 질문을 만들어야 합니다. 즉, Text를 기반으로 사실 질문을 만드세요.
	4.	질문에 “주어진 Text에서” 또는 “제공된 단락에서”와 같은 표현을 포함해서는 안 됩니다.
사용자는 질문의 출처가 Text라는 것을 모르기 때문에 반드시 그 출처를 언급해서는 안 됩니다.
	5.	파일 이름이나 파일 제목에 대한 질문을 하지 마세요. 파일의 내용에 대해 물어보세요.
예를 들어, '문서의 파일 이름은 무엇입니까?'와 같은 질문을 작성하지 마세요.
	6.	질문을 한국어로 작성하세요.
```

#### 2. concept_completion_query_gen

이 함수는 개념 완성 질문을 생성합니다. 개념 완성 질문은 특정 개념이나 용어의 정의, 설명, 특성 등을 묻는 질문입니다.

**기능:**
- 문서에서 중요한 개념이나 용어를 식별하고 이에 대한 질문 생성
- "~이란 무엇인가?", "~의 특징은 무엇인가?" 등의 형태로 질문 생성
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원

**사용 예시:**
```python
from autorag.data.qa.query.llama_gen_query import concept_completion_query_gen

new_qa = qa.batch_apply(concept_completion_query_gen, llm=llm, lang="ko")
```

**시스템 프롬프트:**
```Prompt
당신은 Text를 “개념 완성” 질문으로 변환하는 AI입니다.
"개념 완성" 질문은 개념의 본질이나 정체성에 대해 직접적으로 묻는 질문입니다.

다음 지시사항을 따르세요.
지시사항:
1.	질문은 반드시 주어진 Text를 기반으로 작성되어야 합니다.
2.	질문은 Text를 기반으로 가능한 한 자세하게 작성되어야 합니다.
3.	Text에서 제공된 정보를 묻는 질문을 생성하세요.
4.	Text의 특정 키워드를 반드시 질문에 포함하세요.
5.	질문에 “주어진 Text에서” 또는 “제공된 단락에서”와 같은 표현을 포함해서는 안 됩니다.
사용자는 질문의 출처가 Text라는 것을 모르기 때문에 반드시 그 출처를 언급해서는 안 됩니다.
6.	파일 이름이나 파일 제목에 대한 질문을 하지 마세요. 파일의 내용에 대해 물어보세요.
예를 들어, '문서의 파일 이름은 무엇입니까?'와 같은 질문을 작성하지 마세요.
7.	질문을 한국어로 작성하세요.
```

#### 3. two_hop_incremental

이 함수는 두 단계(two-hop) 추론이 필요한 질문을 생성합니다. 이는 두 개의 서로 다른 문서에서 정보를 결합하여 답변해야 하는 복잡한 질문입니다.

**기능:**
- 두 개의 서로 다른 문서에서 정보를 연결하는 질문 생성
- 첫 번째 문서에서 얻은 정보를 바탕으로 두 번째 문서에 대한 질문 생성
- 더 복잡하고 추론이 필요한 질문 생성
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원

**사용 예시:**
```python
from autorag.data.qa.query.llama_gen_query import two_hop_incremental

new_qa = qa.batch_apply(two_hop_incremental, llm=llm, lang="ko")
```

이 함수는 다음과 같은 형태의 출력을 생성합니다:

1. 답변(Answer): 질문에 대한 최종 답변
2. 한 단계 질문(One-hop question): 첫 번째 문서에 기반한 질문
3. 두 단계 질문(Two-hop question): 두 번째 문서를 포함하여 생성된 최종 질문

**시스템 프롬프트:**
```Prompt
Generate a multi-hop question for the given answer which requires reference to all of the given documents.

[USER]
Document 1: The Municipality of Nuevo Laredo is located in the Mexican state of Tamaulipas.
Document 2: The Ciudad Deportiva (Sports City ¨ ¨) is a sports
complex in Nuevo Laredo, Mexico. It is home to the Tecolotes de
Nuevo Laredo Mexican Baseball League team and ...

[ASSISTANT]
Answer: Tamaulipas
One-hop question (using Document 1): In which Mexican state is Nuevo Laredo located?
Two-hop question (using Document 2):  In which Mexican state can one find the Ciudad Deportiva, home to the Tecolotes de Nuevo Laredo?
```

#### 4. custom_query_gen

이 함수는 사용자 정의 프롬프트를 사용하여 질문을 생성합니다. 다른 함수들과 달리, 이 함수는 사용자가 직접 정의한 메시지 형식을 사용하여 더 유연한 질문 생성이 가능합니다.

**기능:**
- 사용자 정의 프롬프트를 사용한 맞춤형 질문 생성
- 기본 제공 함수로 커버되지 않는 특수한 형태의 질문 생성 가능
- 다양한 언어 및 형식 지원

**사용 예시:**
```python
from autorag.data.qa.query.llama_gen_query import custom_query_gen

messages = [  
    ChatMessage(  
        role=MessageRole.SYSTEM,  
        content="""As an expert AI assistant, create one question that can be directly answered from the provided **Text**..."""  
    )  
]  
new_qa = qa.batch_apply(custom_query_gen, llm=llm, messages=messages)
```

### 실제 사용 예시

AutoRAG에서는 이러한 함수들을 사용하여 QA 데이터셋을 생성하는 예시를 제공합니다: 

```python
import pandas as pd
from llama_index.llms.openai import OpenAI

from autorag.data.qa.filter.dontknow import dontknow_filter_rule_based
from autorag.data.qa.generation_gt.llama_index_gen_gt import (
	make_basic_gen_gt,
	make_concise_gen_gt,
)
from autorag.data.qa.schema import Raw, Corpus
from autorag.data.qa.query.llama_gen_query import factoid_query_gen
from autorag.data.qa.sample import random_single_hop

llm = OpenAI()
raw_df = pd.read_parquet("your/path/to/parsed.parquet")
raw_instance = Raw(raw_df)

corpus_df = pd.read_parquet("your/path/to/corpus.parquet")
corpus_instance = Corpus(corpus_df, raw_instance)

initial_qa = (
	corpus_instance.sample(random_single_hop, n=3)
	.map(
		lambda df: df.reset_index(drop=True),
	)
	.make_retrieval_gt_contents()
	.batch_apply(
		factoid_query_gen,  # query generation
		llm=llm,
	)
)
```

이 예시에서는 `factoid_query_gen` 함수를 사용하여 사실 기반 질문을 생성하고 있습니다.

### Notes

`autorag.data.qa.query.llama_gen_query` 패키지는 LlamaIndex LLM을 사용하는 질문 생성 함수를 제공하는 반면, 유사한 기능을 하는 `autorag.data.qa.query.openai_gen_query` 패키지도 있습니다. 이 패키지는 OpenAI API를 직접 사용하여 구조화된 출력을 통해 더 안정적인 결과를 제공할 수 있습니다. 두 패키지는 동일한 이름의 함수들을 제공하지만 내부 구현 방식이 다릅니다.

또한, 이 함수들은 주로 QA 데이터셋 생성 과정에서 사용되며, 특히 RAG(Retrieval-Augmented Generation) 시스템의 성능을 평가하기 위한 다양한 유형의 질문을 생성하는 데 유용합니다.

AutoRAG의 공식 문서에서도 이러한 질문 생성 함수들에 대한  자세한 설명과 예시가 제공됩니다: 
https://docs.auto-rag.com/data_creation/qa_creation/query_gen.html#question-types

##  2. 질문 최적화(Query Evolving)

`autorag.data.qa.evolve.llama_index_query_evolve` 패키지는 LlamaIndex LLM을 사용하여 질문(query)을 이볼빙시키는 함수들을 제공합니다. 

이 패키지에서는 다음과 같은 함수들을 제공합니다:

1. `conditional_evolve_ragas`
2. `reasoning_evolve_ragas`
3. `compress_ragas`
4. `llama_index_generate_base` (내부 유틸리티 함수)

### 각 함수의 상세 설명

#### 1. conditional_evolve_ragas

이 함수는 기존 질문을 조건부 질문으로 이볼빙시킵니다. 조건부 질문이란 특정 조건을 추가하여 질문의 복잡성을 높이는 방식입니다.

**기능:**
- 원래 질문에 조건을 추가하여 더 복잡하고 구체적인 질문으로 변환
- 검색 시스템의 성능을 테스트하기 위한 더 어려운 질문 생성
- 다양한 언어(영어, 한국어, 일본어 등) 지원

**사용 예시:**
```python
from autorag.data.qa.evolve.llama_index_query_evolve import conditional_evolve_ragas

new_qa = qa.batch_apply(
	conditional_evolve_ragas, 
	llm=llm, 
	lang="ko",
)
```

**시스템 프롬프트:**
```Prompt
제공된 질문에 조건에 관련한 내용을 추가하여 복잡성을 높이세요.
질문의 Context에 영향을 미치는 시나리오나 조건을 포함하여 질문을 더 복잡하게 만드는 것이 목표입니다.
질문을 다시 작성할 때 다음 규칙을 따르십시오.
1. 다시 작성된 질문은 100자를 넘지 않아야 합니다. 가능한 경우 약어를 사용하십시오.
2. 다시 작성된 질문은 합리적이어야 하며 사람이 이해하고 응답할 수 있어야 합니다.
3. 다시 작성된 질문은 현재 Context에서 완전히 답변할 수 있어야 합니다.
4. '제공된 글', '단락에 따르면?', 'Context에 의하면' 등의 문구는 질문에 나타날 수 없습니다.
5. 한국어로 질문을 작성하세요.

------
[USER]
Question: 식물의 뿌리 기능이 뭐야?
Context: 식물의 뿌리는 토양에서 물과 영양분을 흡수하고, 식물을 땅에 고정하며, 영양분을 저장합니다.

[ASSISTANT]
식물의 뿌리는 토양 영양분과 안정성에 대해 어떤 역할을 하나요?
------
[USER]
Question: 백신은 질병을 어떻게 예방하나요?
Context: 백신은 신체의 면역 반응을 자극하여 병원체를 인식하고 싸우는 항체를 생성함으로써 질병으로부터 보호합니다.
Output:

[ASSISTANT]
백신은 신체의 면역 체계를 어떻게 활용해서 질병을 예방합니까?
```

#### 2. reasoning_evolve_ragas

이 함수는 기존 질문을 추론이 필요한 복잡한 질문으로 이볼빙시킵니다. 이러한 질문들은 단순한 사실 검색보다 더 깊은 사고와 추론을 요구합니다.

**기능:**
- 원래 질문을 더 복잡하고 추론이 필요한 형태로 변환
- 답변을 추측하기 어려운 복잡한 질문 생성
- 다양한 언어(영어, 한국어, 일본어 등) 지원

**사용 예시:**
```python
from autorag.data.qa.evolve.llama_index_query_evolve import reasoning_evolve_ragas

new_qa = qa.batch_apply(
	reasoning_evolve_ragas, 
	llm=llm, 
	lang="ko",
)
```

**시스템 프롬프트:**
```Prompt
주어진 Context를 기반으로 기존 질문을 복잡하게 만들어 여러 논리적인 사고가 필요한 질문으로 다시 작성하세요.
질문에 답하려면 주어진 Context의 정보를 사용해 여러 논리적 사고나 추론을 해야 합니다.
질문을 다시 작성할 때 따라야 할 규칙:
1. 다시 작성된 질문은 Context에 있는 정보만으로 완전히 답변할 수 있어야 합니다.
2. 100자를 초과하는 질문을 작성하지 마세요. 가능한 경우 약어를 사용하세요.
3. 질문이 명확하고 모호하지 않도록 하세요.
4. '제공된 Context에 기반하여', '해당 단락에 따르면' 등의 문구는 질문에 포함되지 않아야 합니다.
5. 한국어로 질문을 작성하세요.

-----
[USER]
Question: 프랑스의 수도는 어디인가요?,
Context: 프랑스는 서유럽에 있는 나라입니다. 파리, 리옹, 마르세유를 포함한 여러 도시가 있습니다. 파리는 에펠탑과 루브르 박물관 같은 문화적 랜드마크로 유명할 뿐만 아니라 행정 중심지로도 알려져 있습니다.
Output: 

[ASSISTANT]
에펠탑과 행정 중심지, 두 단어는 어떤 도시를 가리키나요?
-----
[USER]
질문: Python에서 append() 메서드는 무엇을 하나요?
컨텍스트: Python에서 리스트는 하나의 변수에 여러 항목을 저장하는 데 사용됩니다. 리스트는 데이터를 저장하는 데 사용되는 4가지 내장 데이터 유형 중 하나입니다. append() 메서드는 리스트의 끝에 새로운 항목을 추가합니다.
출력:

[ASSISTANT]
리스트가 변수들을 모아 놓은 것을 나타낸다면, 어떤 메서드를 사용해야 항목을 하나 더 추가할 수 있습니까?
```

#### 3. compress_ragas

다른 이볼빙 함수들과 달리, 이 함수는 질문을 더 간결하고 명확하게 압축합니다. 때로는 LLM이 너무 복잡하거나 직접적인 질문을 생성할 수 있는데, 이 함수는 그런 질문을 더 간결하게 만들어 검색 성능을 향상시킵니다.

**기능:**
- 복잡한 질문을 더 간결하고 명확한 형태로 압축
- 원래 질문의 의미는 유지하면서 불필요한 정보 제거
- 검색 시스템의 성능 향상에 도움
- 다양한 언어(영어, 한국어, 일본어 등) 지원

**사용 예시:**
```python
from autorag.data.qa.evolve.llama_index_query_evolve import compress_ragas

new_qa = qa.batch_apply(compress_ragas, llm=llm, lang="ko")
```

**시스템 프롬프트:**
```Prompt
주어진 질문을 더 간접적이고 짧게 다시 작성하세요.
목표는 질문을 원래 질문의 본질을 유지하면서 너무 직설적이지 않게 만드는 것입니다.
약어 등을 사용하여 질문을 더 짧게 만드세요.

-----
[USER]
Question: 지구와 달 사이의 거리는 얼마입니까?
Output: 

[ASSISTANT]
달은 지구에서 얼마나 떨어져 있나요?
-----
[USER]
Question: 초콜릿 케이크를 굽기 위해 필요한 재료는 무엇입니까?
Output: 

[ASSISTANT]
초콜릿 케이크에 필요한 것은 무엇인가요?
```

#### 4. llama_index_generate_base (내부 유틸리티 함수)

이 함수는 다른 이볼빙 함수들의 기반이 되는 내부 유틸리티 함수입니다. 직접 사용하기보다는 다른 이볼빙 함수들의 구현에 사용됩니다.

**기능:**
- 질문과 컨텍스트를 받아 LLM에 전달하는 기본 로직 제공
- 메시지 형식 구성 및 LLM 응답 처리
- 비동기 처리 지원 llama_index_query_evolve.py:10-25

### 사용 방법

이 함수들은 일반적으로 QA 객체의 `batch_apply` 메서드를 통해 사용됩니다:

```python
from llama_index.llms.openai import OpenAI  
from autorag.data.qa.schema import QA  
from autorag.data.qa.evolve.llama_index_query_evolve import llama_index_generate_base  
  
llm = OpenAI()  
qa = QA(qa_df)  # qa_df는 질문이 포함된 데이터프레임  

async def custom_query_gen(
	row: Dict,
	llm: BaseLLM,
	lang: str = "en",
) -> Dict:
	CUSTOM_ PROMPT = {
		"ko": [
			ChatMessage (
				role=MessageRole.SYSTEM,
				content="""당신은 스타트업의 지원 프로그램 중 하나인 "글로벌 팁스 프로그램"에 대해 알려주는 어시스터트의 관리자입니다.
RAG 시스템을 관리를 위해, 사용자들의 예상 질문을 추출해야 합니다.
주어지는 문서는 글로벌 팁스 프로그램에 대한 안내 문서입니다. 
문서를 보고 예상되는 사용자들의 질문을 적으세요.
단, 반드시 하나의 질문을 실성하세요.
생성하는 질문에는 미사여구 없이, 해당 질이 포함되어야 합니다.
""",
			)
		]
	}
	
	return await llama_index_generate_base(
		row, llm, CUSTOM_PROMPT[lang]
	)

new_qa = qa.batch_apply(custom_query_gen, llm=llm, lang="ko")
```

위 코드에서 볼 수 있듯이, 각 함수는 원래 질문을 새로운 형태로 변환하며, 변환된 질문은 원래 질문과 다르지만 관련된 의미를 유지합니다.
### 실제 활용 사례

AutoRAG의 `advanced_create` 함수에서는 이러한 이볼빙 함수들을 사용하여 다양한 난이도의 질문을 생성합니다: [qa_create.py:54-86](https://github.com/Marker-Inc-Korea/AutoRAG/blob/93eaa4a2/api/src/qa_create.py#L54-L86)

이 코드에서는 데이터셋의 절반에 `reasoning_evolve_ragas`를 적용하여 복잡한 추론 질문을 생성하고, 나머지 절반에는 `compress_ragas`를 적용하여 간결한 질문을 생성합니다. 이렇게 하면 다양한 난이도의 질문이 포함된 균형 잡힌 데이터셋을 만들 수 있습니다.
### Notes

`autorag.data.qa.evolve.llama_index_query_evolve` 패키지는 LlamaIndex LLM을 사용하는 질문 이볼빙 함수를 제공하는 반면, 유사한 기능을 하는 `autorag.data.qa.evolve.openai_query_evolve` 패키지도 있습니다. 이 패키지는 OpenAI API를 직접 사용하여 구조화된 출력을 통해 더 안정적인 결과를 제공할 수 있습니다. 두 패키지는 동일한 이름의 함수들을 제공하지만 내부 구현 방식이 다릅니다.

또한, 이 함수들은 주로 QA 데이터셋 생성 과정에서 사용되며, 특히 RAG(Retrieval-Augmented Generation) 시스템의 성능을 평가하기 위한 다양한 난이도의 질문을 생성하는 데 유용합니다.

AutoRAG의 공식 문서에서도 이러한 이볼빙 함수들에 대한  자세한 설명과 예시가 제공됩니다: 
https://docs.auto-rag.com/data_creation/qa_creation/evolve.html

## 3. 답변 생성(Answer Generation)

`autorag.data.qa.generation_gt.llama_index_gen_gt` 패키지는 LlamaIndex LLM을 사용하여 질문에 대한 답변(ground truth)을 생성하는 함수들을 제공합니다. 

이 패키지에서는 다음과 같은 함수들을 제공합니다:

1. `make_basic_gen_gt`
2. `make_concise_gen_gt`
3. `make_custom_gen_gt`

### 각 함수의 상세 설명

#### 1. make_basic_gen_gt

이 함수는 질문에 대한 기본적인 답변을 생성합니다. 특별한 제약 없이 LLM이 주어진 컨텍스트(retrieval ground truth)와 질문을 바탕으로 자연스러운 답변을 생성합니다.

**기능:**
- 주어진 질문과 컨텍스트를 바탕으로 상세한 답변 생성
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 자연스럽고 완전한 문장 형태의 답변 생성

**사용 예시:**
```python
from autorag.data.qa.generation_gt.llama_index_gen_gt import make_basic_gen_gt

result_qa = qa.batch_apply(make_basic_gen_gt, llm=llm, lang="ko")
```

**시스템 프롬프트:**
```Prompt
당신은 주어진 질문에 대한 답을 제공된 Text 내에서 찾는 AI 비서입니다. 질문과 관련된 증거를 Text에서 찾아 적절한 답변을 작성하세요.
```

#### 2. make_concise_gen_gt

이 함수는 질문에 대한 간결한 답변을 생성합니다. 기본 답변보다 더 짧고 핵심만 담은 형태의 답변을 생성하며, 때로는 단어나 구문 수준의 매우 간결한 답변을 제공합니다.

**기능:**
- 주어진 질문에 대한 핵심 정보만 포함한 간결한 답변 생성
- 불필요한 설명이나 부가 정보 없이 핵심만 전달
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원

**사용 예시:**
```python
from autorag.data.qa.generation_gt.llama_index_gen_gt import make_concise_gen_gt

result_qa = qa.batch_apply(make_concise_gen_gt, llm=llm, lang="ko")
```

**시스템 프롬프트:**
```Prompt
당신은 주어진 질문에 대해 제공된 Text 내에서 답을 찾는 AI 비서입니다.
질문에 대한 답을 Text에서 찾아 적절한 답변을 작성하세요.
답변은 간결하고 질문에 관련된 내용만 포함해야 합니다.
불필요하게 길게 답변하지 말고, 명확하게 작성하세요.
완전한 문장이 아니어도 되며, 답은 단어나 요약일 수 있습니다.
```
#### 3. make_custom_gen_gt

이 함수는 사용자 정의 시스템 프롬프트를 사용하여 맞춤형 답변을 생성합니다. 다른 함수들과 달리, 이 함수는 사용자가 직접 정의한 시스템 프롬프트를 사용하여 더 유연한 답변 생성이 가능합니다.

**기능:**
- 사용자 정의 시스템 프롬프트를 사용한 맞춤형 답변 생성
- 기본 제공 함수로 커버되지 않는 특수한 형태의 답변 생성 가능
- 다양한 언어 및 형식 지원

**사용 예시:**
```python
from autorag.data.qa.generation_gt.llama_index_gen_gt import make_custom_gen_gt

system_prompt = """As an expert AI assistant focused on providing accurate and concise responses..."""  
result_qa = qa.batch_apply(make_custom_gen_gt, llm=llm, system_prompt=system_prompt)
```

### 실제 사용 예시

AutoRAG의 데이터 생성 튜토리얼에서는 이러한 함수들을 사용하여 QA 데이터셋을 생성하는 예시를 제공합니다: 

```Python
initial_qa = (
    corpus_instance.sample(random_single_hop, n=3)
    .map(
        lambda df: df.reset_index(drop=True),
    )
    .make_retrieval_gt_contents()
    .batch_apply(
        factoid_query_gen,  # query generation
        llm=llm,
    )
    .batch_apply(
        make_basic_gen_gt,  # answer generation (basic)
        llm=llm,
    )
    .batch_apply(
        make_concise_gen_gt,  # answer generation (concise)
        llm=llm,
    )
    .filter(
        dontknow_filter_rule_based,  # filter don't know
        lang="en",
    )
)
```

이 예시에서는 `make_basic_gen_gt`와 `make_concise_gen_gt` 함수를 순차적으로 적용하여 두 가지 유형의 답변을 생성하고 있습니다.

## Notes

`autorag.data.qa.generation_gt.llama_index_gen_gt` 패키지는 LlamaIndex LLM을 사용하는 답변 생성 함수를 제공하는 반면, 유사한 기능을 하는 `autorag.data.qa.generation_gt.openai_gen_gt` 패키지도 있습니다. 이 패키지는 OpenAI API를 직접 사용하여 구조화된 출력을 통해 더 안정적인 결과를 제공할 수 있습니다. 두 패키지는 동일한 이름의 함수들을 제공하지만 내부 구현 방식이 다릅니다.

이러한 답변 생성 함수들은 주로 QA 데이터셋 생성 과정에서 사용되며, 특히 RAG(Retrieval-Augmented Generation) 시스템의 성능을 평가하기 위한 기준 답변(ground truth)을 생성하는 데 유용합니다. 생성된 답변은 나중에 RAG 시스템이 생성한 답변과 비교하여 성능을 평가하는 데 사용됩니다.

AutoRAG의 공식 문서에서도 이러한 답변 생성 함수들에 대한 설명을 제공합니다: 
[Answer Generation](https://docs.auto-rag.com/data_creation/qa_creation/answer_gen.html)

각 답변 유형에 대한 자세한 설명과 예시도 제공됩니다:

-  [기본 생성 (Basic Generation)](https://docs.auto-rag.com/data_creation/qa_creation/answer_gen.html#basic-generation) 
- [간결 생성 (Concise Generation)](https://docs.auto-rag.com/data_creation/qa_creation/answer_gen.html#concise-generation)
- [커스텀 생성 (Custom Generation)](https://docs.auto-rag.com/data_creation/qa_creation/answer_gen.html#custom-generation)
## 4. dontknow 필터링(Filtering)

`autorag.data.qa.filter.dontknow` 패키지는 AutoRAG 프레임워크에서 QA 데이터셋 생성 과정 중 "모르겠습니다" 유형의 응답을 필터링하는 함수들을 제공합니다. 

이 패키지에서는 다음과 같은 함수들을 제공합니다:

1. `dontknow_filter_rule_based`
2. `dontknow_filter_llama_index`
3. `dontknow_filter_openai`
4. `Response` 클래스

### 각 함수의 상세 설명

#### 1. dontknow_filter_rule_based

이 함수는 규칙 기반 접근 방식을 사용하여 "모르겠습니다" 유형의 응답이 포함된 QA 쌍을 필터링합니다. 미리 정의된 "모르겠습니다" 표현 목록을 사용하여 필터링을 수행합니다.

**기능:**
- 미리 정의된 "모르겠습니다" 표현 목록을 사용하여 간단하고 빠른 필터링 수행
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 계산 비용이 적고 외부 API에 의존하지 않음

**사용 예시:**
```python
from autorag.data.qa.filter.dontknow import dontknow_filter_rule_based

filtered_qa = qa.filter(
	dontknow_filter_rule_based, 
	lang="ko"
)
```

#### 2. dontknow_filter_llama_index

이 함수는 LlamaIndex LLM을 사용하여 "모르겠습니다" 유형의 응답을 필터링합니다. OpenAI API 대신 다른 LLM 모델을 사용할 수 있는 옵션을 제공합니다.

**기능:**
- LlamaIndex LLM을 사용하여 필터링 수행
- 다양한 LLM 모델(OpenAI, Ollama 등) 지원
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 비동기 처리 지원

**사용 예시:**
```python
from llama_index.llms.openai import OpenAI  
from autorag.data.qa.filter.dontknow import dontknow_filter_llama_index

llm = OpenAI()  
filtered_qa = qa.batch_filter(
	dontknow_filter_llama_index, 
	llm=llm, 
	lang="ko",
)
```

**프롬프트:**
```Prompt
다음 문장은 어떠한 질문에 대한 대답입니다. 해당 문장이 질문에 대해서 '모른다고' 답한 것인지 판단하십시오.
만약 해당 문장이 '모른다고' 답한 것이라면, True를 반환하세요. 그렇지 않다면 False를 반환하세요.
```

#### 3. dontknow_filter_openai

이 함수는 OpenAI API를 사용하여 "모르겠습니다" 유형의 응답을 더 정교하게 필터링합니다. 단순한 규칙 기반 접근 방식보다 더 복잡한 경우도 처리할 수 있습니다.

**기능:**
- OpenAI API를 사용하여 더 정교한 필터링 수행
- 규칙 기반 접근 방식으로는 감지하기 어려운 복잡한 "모르겠습니다" 표현도 식별 가능
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 비동기 처리 지원

**사용 예시:**
```python
from openai import AsyncOpenAI  
from autorag.data.qa.filter.dontknow import dontknow_filter_openai

filtered_qa = qa.batch_filter(
	dontknow_filter_openai, 
	client=AsyncOpenAI(), 
	lang="ko",
)
```

**프롬프트:**
```Prompt
다음 문장은 어떠한 질문에 대한 대답입니다. 해당 문장이 질문에 대해서 '모른다고' 답한 것인지 판단하십시오.
만약 해당 문장이 '모른다고' 답한 것이라면, True를 반환하세요. 그렇지 않다면 False를 반환하세요.
```

#### 4. Response (클래스)

이 클래스는 LLM 기반 필터링 함수에서 사용되는 응답 구조를 정의합니다. 주로 내부적으로 사용되며, 응답이 "모르겠습니다" 유형인지 여부를 나타내는 플래그를 포함합니다.

**기능:**
- `is_dont_know` 플래그를 통해 응답이 "모르겠습니다" 유형인지 여부 표시
- LLM 기반 필터링 함수에서 구조화된 출력을 위해 사용

### 실제 사용 사례

AutoRAG의 데이터 생성 튜토리얼에서는 이러한 필터링 함수를 사용하여 QA 데이터셋의 품질을 향상시키는 [예시](https://docs.auto-rag.com/data_creation/qa_creation/filter.html#don-t-know-filter)를 제공합니다: 

```python
initial_qa = (
    # (⋯)
    .filter(
        dontknow_filter_rule_based,  # filter don't know
        lang="ko",
    )
)
```

위 예시에서는 `dontknow_filter_rule_based` 함수를 사용하여 "모르겠습니다" 응답이 포함된 QA 쌍을 필터링하고 있습니다.
### Notes

`autorag.data.qa.filter.dontknow` 패키지는 QA 데이터셋 생성 과정에서 "모르겠습니다" 유형의 응답을 필터링하는 다양한 방법을 제공합니다. 규칙 기반 접근 방식(`dontknow_filter_rule_based`)은 계산 비용이 적고 빠르지만 복잡한 표현을 감지하는 데 한계가 있을 수 있습니다. 반면, LLM 기반 접근 방식(`dontknow_filter_openai`, `dontknow_filter_llama_index`)은 더 정교한 필터링이 가능하지만 계산 비용이 더 높고 외부 API에 의존합니다.

사용 사례와 요구 사항에 따라 적절한 필터링 방법을 선택할 수 있으며, 이를 통해 RAG 시스템의 성능 평가에 사용되는 데이터셋의 품질을 향상시킬 수 있습니다.
## 5. passage_dependency 필터링(Filtering)

`autorag.data.qa.filter.passage_dependency` 패키지는 AutoRAG 프레임워크에서 QA 데이터셋 생성 과정 중 "passage-dependent" 질문을 필터링하는 함수들을 제공합니다. 

이 패키지에서는 다음과 같은 함수들을 제공합니다:

1. `passage_dependency_filter_llama_index`
2. `passage_dependency_filter_openai`
3. `Response` 클래스

### 각 함수의 상세 설명

#### 1. passage_dependency_filter_llama_index

이 함수는 LlamaIndex LLM을 사용하여 "passage-dependent" 질문을 필터링합니다. OpenAI API 대신 다른 LLM 모델을 사용할 수 있는 옵션을 제공합니다.

**기능:**
- LlamaIndex LLM을 사용하여 질문이 passage-dependent인지 판단
- 다양한 LLM 모델(OpenAI, Ollama 등) 지원
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 비동기 처리 지원

**사용 예시:**
```python
from llama_index.llms.openai import OpenAI  
from autorag.data.qa.filter.passage_dependency import passage_dependency_filter_llama_index

llm = OpenAI(temperature=0, model="gpt-4o-mini")  

filtered_qa = qa.batch_filter(
	passage_dependency_filter_llama_index, 
	llm=llm, 
	lang="ko",
)
```

**프롬프트:**
```Prompt
당신은 '단락 의존' 질문을 인식하는 분류기입니다.
'단락 의존'이란 어떤 단락이 선택 되는지 따라 답이 달라지는 질문을 의미합니다.
예를 들어, '주어진 표에 따르면 가장 높은 점수는 무엇인가요?'라는 질문은 단락 의존 질문입니다. 왜냐하면 표가 어떤 것인지에 따라 그 답이 달라지기 때문입니다.

반면에, 다음 문장들은 단락 의존적이지 않습니다.
'KBO 야구 역사상 한 경기에서 가장 높은 점수는 무엇인가요?' 또는 '프랑스의 수도는 무엇인가요?'
이러한 문장은 단락에 관계 없이 동일한 답을 가집니다.

입력된 질문이 단락 의존적이라면 True를 반환하고, 그렇지 않으면 False를 반환하세요.
```
#### 2. passage_dependency_filter_openai

이 함수는 OpenAI API를 사용하여 "passage-dependent" 질문을 필터링합니다. Passage-dependent 질문이란 선택된 문맥이나 패시지에 따라 답변이 달라질 수 있는 질문을 의미합니다.

**기능:**
- OpenAI API를 사용하여 질문이 passage-dependent인지 판단
- 영어("en"), 한국어("ko"), 일본어("ja") 등 다양한 언어 지원
- 비동기 처리 지원

**사용 예시:**
```python
from openai import AsyncOpenAI  
from autorag.data.qa.filter.passage_dependency import passage_dependency_filter_openai

filtered_qa = qa.batch_filter(
	passage_dependency_filter_openai, 
	client=AsyncOpenAI(), 
	lang="ko",
)
```

**프롬프트:**
```Prompt
당신은 '단락 의존' 질문을 인식하는 분류기입니다.
'단락 의존'이란 어떤 단락이 선택 되는지 따라 답이 달라지는 질문을 의미합니다.
예를 들어, '주어진 표에 따르면 가장 높은 점수는 무엇인가요?'라는 질문은 단락 의존 질문입니다. 왜냐하면 표가 어떤 것인지에 따라 그 답이 달라지기 때문입니다.

반면에, 다음 문장들은 단락 의존적이지 않습니다.
'KBO 야구 역사상 한 경기에서 가장 높은 점수는 무엇인가요?' 또는 '프랑스의 수도는 무엇인가요?'
이러한 문장은 단락에 관계 없이 동일한 답을 가집니다.

입력된 질문이 단락 의존적이라면 True를 반환하고, 그렇지 않으면 False를 반환하세요.
```
#### 3. Response 클래스

이 클래스는 LLM 기반 필터링 함수에서 사용되는 응답 구조를 정의합니다. 주로 내부적으로 사용되며, 질문이 "passage-dependent"인지 여부를 나타내는 플래그를 포함합니다.

**기능:**
- 질문이 "passage-dependent"인지 여부를 나타내는 플래그 포함
- LLM 기반 필터링 함수에서 구조화된 출력을 위해 사용 test_passage_dependency.py:18

### 필터링의 중요성

 [이 문서](https://docs.auto-rag.com/data_creation/qa_creation/filter.html#unanswerable-question-filtering)에서 설명하는 바와 같이 "모르겠습니다" 필터링은 QA 데이터셋 생성 과정에서 중요한 단계입니다.

이러한 필터링을 통해 RAG 시스템의 성능 평가에 사용되는 데이터셋의 품질을 향상시킬 수 있습니다.

### 실제 사용 사례

AutoRAG의 데이터 생성 문서에서는 이러한 필터링 함수를 사용하여 QA 데이터셋의 품질을 향상시키는 예시를 제공합니다: https://docs.auto-rag.com/data_creation/qa_creation/filter.html#llm-based-don-t-know-filter

#### OpenAI 사용 예시
```python
from openai import AsyncOpenAI
from autorag.data.qa.schema import QA
from autorag.data.qa.filter.passage_dependency import passage_dependency_filter_openai

en_qa = QA(en_qa_df)

result_en_qa = en_qa.batch_filter(
    passage_dependency_filter_openai, 
    client=AsyncOpenAI(), 
    lang="ko"
)
.map(
	lambda df: df.reset_index(drop=True)
)
```
#### LlamaIndex 사용 예시
```python
from llama_index.llms.ollama import Ollama
from autorag.data.qa.schema import QA
from autorag.data.qa.filter.passage_dependency import passage_dependency_filter_llama_index

llm = OpenAI(model="gpt-4o-mini", temperature=0)

en_qa = QA(en_qa_df)

result_en_qa = en_qa.batch_filter(
	passage_dependency_filter_llama_index, 
	llm=llm, 
	lang="ko"
)
.map(
    lambda df: df.reset_index(drop=True)  # reset index
)
```

### Notes

`autorag.data.qa.filter.passage_dependency` 패키지는 QA 데이터셋 생성 과정에서 "passage-dependent" 질문을 필터링하는 다양한 방법을 제공합니다. 이러한 질문들은 선택된 문맥에 따라 답변이 달라질 수 있기 때문에, 최고의 검색 시스템을 사용하더라도 정확한 패시지를 찾기 어려울 수 있습니다.

이 패키지는 `autorag.data.qa.filter.dontknow` 패키지와 유사한 인터페이스를 제공하지만, 다른 유형의 문제를 해결합니다. "dontknow" 필터는 답변할 수 없는 질문을 필터링하는 반면, "passage_dependency" 필터는 문맥에 따라 답변이 달라질 수 있는 질문을 필터링합니다.

두 가지 구현 방식(OpenAI와 LlamaIndex)을 제공하여 사용자가 자신의 요구 사항에 맞는 방법을 선택할 수 있도록 합니다. OpenAI 구현은 구조화된 출력을 통해 더 안정적인 결과를 제공할 수 있으며, LlamaIndex 구현은 다양한 LLM 모델을 지원합니다.

