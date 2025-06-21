---
dg-publish: true
title: KoNLPy의 Kkma 분석기와 Ollama를 사용하여 RAG 구축하기
tags:
  - KoNLPy
  - Kkma
  - Ollama
  - langchain
  - 한국어
  - 자연어처리
  - FAISS
  - rag
created: 2025-05-03 04:42:41
updated: 2025-05-06 01:22:56
---
## 소개

자연어 처리(NLP)에서 텍스트 분할은 중요한 전처리 단계입니다. 특히 한국어와 같은 특수한 언어 구조를 가진 경우, 적절한 도구의 선택이 중요합니다. 이 글에서는 한국어 텍스트 처리를 위한 KoNLPy의 Kkma 분석기와 LangChain을 활용하는 방법을 소개합니다.

## 한국어 형태소 분석기: KoNLPy의 Kkma

KoNLPy(Korean Natural Language Processing in Python)는 한국어 자연어 처리를 위한 파이썬 패키지입니다. 그 중 Kkma(Korean Knowledge Morpheme Analyzer)는 상세한 형태소 분석을 제공합니다.

### Kkma의 주요 특징
- 문장을 단어로, 단어를 형태소로 분해
- 각 토큰의 품사 식별
- 긴 텍스트를 개별 문장으로 분할 가능

> [!tip]
> Kkma는 한국어 형태소 분석으로 유명하지만, 처리 속도에 영향을 미칠 수 있다는 점에 유의해야 합니다. Kkma는 빠른 텍스트 처리보다 분석의 깊이가 우선시되는 RAG에 가장 적합합니다.

## 실습: '운수 좋은 날' 텍스트 분석하기

이 실습에서는 현진건의 단편소설 '운수 좋은 날'을 사용하여 한국어 텍스트 처리 과정을 살펴보겠습니다.

- [저작물명: 운수 좋은 날](https://gongu.copyright.or.kr/gongu/wrt/wrt/view.do?wrtSn=9002094&menuNo=200019)
- 저작(권)자: 현진건 (저작물 84 건)
- 출처: 한국저작권위원회

### 1. 환경 설정

먼저 필요한 라이브러리를 설치하고 환경을 설정합니다.

```sh
pip install konlpy langchain-text-splitters langchain-community faiss-cpu langchain-core langchain-huggingface
```

```python
import os

# 토크나이저 병렬 처리 활성화
os.environ['TOKENIZERS_PARALLELISM'] = 'true' 
```

### 2. 텍스트 불러오기

```python
with open("../data/현진건-운수_좋은_날+B3356-개벽.txt", encoding='utf-8') as f:
	korean_document = f.read()
```

```python
print(korean_document[:100])
```
	'운수 좋은날\n\n현진건\n\n 새침하게 흐린 품이 눈이 올 듯하더니 눈은 아니 오고 얼다가 만 비가 추\n적추적 내리는 날이었다.\n 이날이야말로 동소문 안에서 인력거꾼 노릇을 하는 김첨지'

### 3. KonlpyTextSplitter 생성 및 텍스트 분할

```python
from langchain_text_splitters import KonlpyTextSplitter

konlpy_text_splitter = KonlpyTextSplitter(
	chunk_size=500,
	chunk_overlap=20,
)
```
>  [KonlpyTextSplitter](https://python.langchain.com/api_reference/text_splitters/konlpy/langchain_text_splitters.konlpy.KonlpyTextSplitter.html)

```python
splited_texts = konlpy_text_splitter.split_text(korean_document) 

print(splited_texts[0])
print(len(splited_texts[0]))
```
	운수 좋은 날 현진건 새침하게 흐린 품이 눈이 올 듯하더니 눈은 아니 오고 얼다가 만 비가 추 적 추적 내리는 날이었다.
	이날이야말로 동소문 안에서 인력거꾼 노릇을 하는 김 첨지에게는 오래간만에도 닥친 운수 좋은 날이었다.
	문안에( 거기도 문 밖은 아니지만) 들어간 답 시는 앞집 마마님을 전찻길까지 모셔 다 드린 것을 비롯으로 행여나 손님이 있을까
	하고 정류장에서 어정 어정하며 내리는 사람 하나하나에게 거의 비는 듯한 눈결을 보내고 있다가 마침내 교원인 듯한 양복쟁이를 동광학교( 東光 學校 )까지 태워 다 주기로 되었다.
	첫 번에 삼십 전 , 둘째 번에 오십 전 - 아침 댓바람에 그리 흉치 않은 일이 었다.
	그야말로 재수가 옴붙어서 근 열흘 동안 돈 구경도 못한 김 첨지는 십 전짜리 백동화 서 푼, 또는 다섯 푼이 찰깍 하고 손바닥에 떨어질 제 거의 눈물을 흘릴 만큼 기뻤었다.
	더구나 이날 이때에 이 팔십 전이라는 돈이 그에게 얼마나 유용한 지 몰랐다.
	489

### 4. 임베딩 모델 생성

여기서는 다국어 지원이 우수한 BAAI/bge-m3 모델을 사용합니다.

```python
from langchain_huggingface import HuggingFaceEmbeddings

model_name = "BAAI/bge-m3"
model_kwargs = {'device': 'mps'} # cpu 또는 cuda
encode_kwargs = {'normalize_embeddings': True}
embeddings = HuggingFaceEmbeddings(
	model_name=model_name,
	model_kwargs=model_kwargs,
	encode_kwargs=encode_kwargs
)
```

### 5. 벡터 스토어 생성 및 문서 임베딩

```python
import faiss
from langchain_community.docstore.in_memory import InMemoryDocstore
from langchain_community.vectorstores import FAISS
from uuid import uuid4
from langchain_core.documents import Document


# 벡터 스토어 생성
index = faiss.IndexFlatL2(len(embeddings.embed_query("hello world")))

vector_store = FAISS(
	embedding_function=embeddings,
	index=index,
	docstore=InMemoryDocstore(),
	index_to_docstore_id={},
)

# 문서 임베딩
documents = [
	Document(
		page_content=text, id=str(uuid4()), metadata={"source": "운수 좋은 날.txt"}
	)
	for text in splited_texts
]

vector_store.add_documents(documents=documents)

print(documents[:1])
```
	[Document(id='1900ca64-4633-459d-8553-c0e13c931dc7', metadata={'source': '운수 좋은 날.txt'}, page_content='운수 좋은 날 현진건 새침하게 흐린 품이 눈이 올 듯하더니 눈은 아니 오고 얼다가 만 비가 추 적 추적 내리는 날이었다.\n\n이날이야말로 동소문 안에서 인력거꾼 노릇을 하는 김 첨지에게는 오래간만에도 닥친 운수 좋은 날이었다.\n\n문안에( 거기도 문 밖은 아니지만) 들어간 답 시는 앞집 마마님을 전찻길까지 모셔 다 드린 것을 비롯으로 행여나 손님이 있을까\n\n하고 정류장에서 어정 어정하며 내리는 사람 하나하나에게 거의 비는 듯한 눈결을 보내고 있다가 마침내 교원인 듯한 양복쟁이를 동광학교( 東光 學校 )까지 태워 다 주기로 되었다.\n\n첫 번에 삼십 전 , 둘째 번에 오십 전 - 아침 댓바람에 그리 흉치 않은 일이 었다.\n\n그야말로 재수가 옴붙어서 근 열흘 동안 돈 구경도 못한 김 첨지는 십 전짜리 백동화 서 푼, 또는 다섯 푼이 찰깍 하고 손바닥에 떨어질 제 거의 눈물을 흘릴 만큼 기뻤었다.\n\n더구나 이날 이때에 이 팔십 전이라는 돈이 그에게 얼마나 유용한 지 몰랐다.'),

### 6. 벡터 스토어 쿼리하기

#### 유사도 검색
```python
results = vector_store.similarity_search("김첨지의 직업은 무엇인가?", k=1)

for res in results:
	print(f"* {res.page_content} [{res.metadata}]")
```
	* 그 중에서 손님을 물색하는 김 첨지의 눈엔 양머리에 뒤축 높은 구두를 신고 망토까지 두른 기생 퇴물인 듯 난봉 여학생인 듯 한 여편네의 모양이 띄었다.
	그는 슬근슬근 그 여자의 곁으로 다가들었다.
	“ 아씨, 인력거 아니 타시랍시요.
	” 그 여학생인지 만 지가 한참은 매우 때깔을 빼며 입술을 꼭 다문 채 김 첨지 를 거들떠보지도 않았다.
	김 첨지는 구걸하는 거지나 무엇같이 연해 연방 그 의 기색을 살피며, “ 아씨, 정거장 애 들보 담 아주 싸게 모셔 다 드리겠습니다.
	댁이 어디 신 가 요.
	” 하고 추근추근하게도 그 여자의 들고 있는 일본식 버들 고리짝에 제 손을 대 었다.
	“ 왜 이래, 남 귀치 않게.” 소리를 벽력 같이 지르고는 돌아선다.
	김 첨지는 어랍시요 하고 물러섰다.
	전차는 왔다.
	김 첨지는 원망스럽게 전차 타는 이를 노리고 있었다.
	그러나 그의 예감( 豫感) 은 틀리지 않았다. [{'source': '운수 좋은 날.txt'}]

#### 스코어(score)로 유사도 검색
```python
results = vector_store.similarity_search_with_score("김첨지의 직업은 무엇인가?", k=1)

for res, score in results:
	print(f"* [SIM={score:3f}] {res.page_content} [{res.metadata}]")
```
	* [SIM=0.921228] 그 중에서 손님을 물색하는 김 첨지의 눈엔 양머리에 뒤축 높은 구두를 신고 망토까지 두른 기생 퇴물인 듯 난봉 여학생인 듯 한 여편네의 모양이 띄었다.
	그는 슬근슬근 그 여자의 곁으로 다가들었다.
	“ 아씨, 인력거 아니 타시랍시요.
	” 그 여학생인지 만 지가 한참은 매우 때깔을 빼며 입술을 꼭 다문 채 김 첨지 를 거들떠보지도 않았다.
	김 첨지는 구걸하는 거지나 무엇같이 연해 연방 그 의 기색을 살피며, “ 아씨, 정거장 애 들보 담 아주 싸게 모셔 다 드리겠습니다.
	댁이 어디 신 가 요.
	” 하고 추근추근하게도 그 여자의 들고 있는 일본식 버들 고리짝에 제 손을 대 었다.
	“ 왜 이래, 남 귀치 않게.” 소리를 벽력 같이 지르고는 돌아선다.
	김 첨지는 어랍시요 하고 물러섰다.
	전차는 왔다.
	김 첨지는 원망스럽게 전차 타는 이를 노리고 있었다.
	그러나 그의 예감( 豫感) 은 틀리지 않았다. [{'source': '운수 좋은 날.txt'}]

#### 리트리버(Retriever)로 변환하여 쿼리하기
```python
retriever = vector_store.as_retriever(search_type="mmr", search_kwargs={"k": 1})
```

```python
retriever.invoke("김첨지의 직업은 무엇인가?")
```
	[Document(metadata={'source': '운수 좋은 날.txt'}, page_content='그 중에서 손님을 물색하는 김 첨지의 눈엔 양머리에 뒤축 높은 구두를 신고 망토까지 두른 기생 퇴물인 듯 난봉 여학생인 듯 한 여편네의 모양이 띄었다.\n\n그는 슬근슬근 그 여자의 곁으로 다가들었다.\n\n“ 아씨, 인력거 아니 타시랍시요.\n\n” 그 여학생인지 만 지가 한참은 매우 때깔을 빼며 입술을 꼭 다문 채 김 첨지 를 거들떠보지도 않았다.\n\n김 첨지는 구걸하는 거지나 무엇같이 연해 연방 그 의 기색을 살피며, “ 아씨, 정거장 애 들보 담 아주 싸게 모셔 다 드리겠습니다.\n\n댁이 어디 신 가 요.\n\n” 하고 추근추근하게도 그 여자의 들고 있는 일본식 버들 고리짝에 제 손을 대 었다.\n\n“ 왜 이래, 남 귀치 않게.” 소리를 벽력 같이 지르고는 돌아선다.\n\n김 첨지는 어랍시요 하고 물러섰다.\n\n전차는 왔다.\n\n김 첨지는 원망스럽게 전차 타는 이를 노리고 있었다.\n\n그러나 그의 예감( 豫感) 은 틀리지 않았다.')]

> [!info]
> FAISS 벡터 스토어를 검색하는 다른 방법도 다양하게 있습니다. 이러한 방법의 전체 목록은 [FAISS API 문서](https://python.langchain.com/api_reference/community/vectorstores/langchain_community.vectorstores.faiss.FAISS.html)를 참조하세요.

### 7. RAG(Retrieval-Augmented Generation) 체인 만들기

RAG는 검색 기반 생성 모델로, 질문에 대한 답변을 생성할 때 관련 문서를 검색하여 참조합니다.

```python
from langchain_ollama import ChatOllama
from operator import itemgetter
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate

# 모델 생성
llm = ChatOllama(model="qwen2.5-7b-instruct-kowiki:q8_0")

# 프롬프트 작성
RAG_TEMPLATE = """
당신은 질문-답변 작업을 위한 도우미입니다.

다음은 질문에 답하기 위해 사용할 맥락입니다:

<context>
{context}
</context>

위의 맥락을 주의 깊게 생각해보세요.

이제 사용자의 질문을 검토하세요:

{question}

위의 맥락만을 사용하여 이 질문에 대한 답변을 제공하세요.

최대 세 문장으로 답변하고 간결하게 유지하세요.

반드시 자연스러운 한국어로 답변하세요.

답변:"""

rag_prompt = ChatPromptTemplate.from_template(RAG_TEMPLATE)

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# 리트리버 생성
retriever = vector_store.as_retriever(search_type="mmr", search_kwargs={"k": 5})

# 체인 생성
chain = (
    {
    "question": RunnablePassthrough(),
    "context": itemgetter("question") | retriever | format_docs,
    }
    | rag_prompt
    | llm
    | StrOutputParser()
)
```

### 8. RAG 체인 테스트

```python
print(1, chain.invoke({"question": "김첨지의 직업은 무엇인가?"}))
print(2, chain.invoke({"question": "학생의 목적지는 어디인가요?"}))
print(3, chain.invoke({"question": "학생은 김첨지에게 얼마를 지불하였나요?"}))
```
	1 '김 첨지는 인력거꾼입니다. 그는 동소문 안에서 인력거를 태우며 돈을 벌었습니다.'
	2 '학생은 남대문 정거장까지 가려고 합니다. 그는 비가 와서 짐을 놔두고 집으로 돌아가는 길에 인력거를 타게 됩니다. 그러다 남대문 정거장까지의 거리에 대해 물어봅니다.'
	3 '학생은 처음에 1원 50전을, 둘째로는 더 주고 80전을 지불했습니다.'

## 결론

이 글에서는 KoNLPy의 Kkma 분석기와 LangChain을 활용하여 한국어 텍스트를 처리하고 분석하는 방법을 살펴보았습니다. 이러한 도구들을 활용하면 한국어 자연어 처리 작업을 효과적으로 수행할 수 있습니다. 특히 RAG 모델을 구현함으로써 텍스트에 대한 질의응답 시스템을 구축할 수 있었습니다.

한국어 NLP 작업에서 적절한 도구와 모델을 선택하는 것이 중요합니다. KoNLPy와 같은 전문화된 라이브러리와 LangChain과 같은 강력한 프레임워크를 결합하면, 복잡한 한국어 텍스트 처리 작업을 효율적으로 수행할 수 있습니다.

---

> 참고: https://python.langchain.com/docs/how_to/split_by_token/#token-splitting-for-korean-with-konlpys-kkma-analyzer
