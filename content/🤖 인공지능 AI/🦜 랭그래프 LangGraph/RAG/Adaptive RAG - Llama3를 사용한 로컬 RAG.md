---
aliases: []
related: 
created: 2024-10-02 03:07:17
title: "Adaptive RAG: Llama3를 사용한 로컬 RAG"
tags:
  - LangGraph
  - Llama3
  - rag
dg-publish: true
modified: 2024-10-04 08:04:55
---

> 원문: https://langchain-ai.github.io/langgraph/tutorials/rag/langgraph_adaptive_rag_local/

# Local RAG agent with Llama3

**참고 논문:**

- **Routing**: Adaptive RAG ([논문](https://arxiv.org/abs/2403.14403)). 질문을 다양한 검색 방식으로 라우팅
- **Fallback**: Corrective RAG ([논문](https://arxiv.org/pdf/2401.15884.pdf)). 문서가 쿼리와 관련이 없으면 웹 검색을 수행
- **Self-correction**: Self-RAG ([논문](https://arxiv.org/abs/2310.11511)). 할루시네이션을 올바르게 수정

![|800](https://i.imgur.com/l47fEVV.png)

## 필요한 패키지 설치
```python
%%capture --no-stderr
%pip install --quiet -U langchain langchain_community tiktoken langchain-nomic "nomic[local]" langchain-ollama scikit-learn langgraph tavily-python bs4
```

## 로컬 모델

### 임베딩 모델

[GPT4All Embeddings](https://blog.nomic.ai/posts/nomic-embed-text-v1):

`pip install langchain-nomic`

### LLM

[Ollama](https://x.com/ollama/status/1839007158865899651)와 [llama3.2](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/)를 사용: `ollama pull llama3.2:3b-instruct-fp16`

```python
### LLM
from langchain_ollama import ChatOllama

# 올라마 모델명
local_llm = 'llama3.2:3b-instruct-fp16'

# 일반적인 응답 모델
llm = ChatOllama(model=local_llm, temperature=0)

# json으로 출력 모델
llm_json_mode = ChatOllama(model=local_llm, temperature=0, format='json')
```

### Tavuky API Key 설정

웹검색을 위해 LLM과 RAG에 최적화된 검색 엔진인 [Tavily](https://tavily.com/)를 사용합니다.

```python
import os
import getpass
from dotenv import load_dotenv

load_dotenv()

def _set_env(var: str):
    if not os.environ.get(var):
        os.environ[var] = getpass.getpass(f"Enter {var}: ")

_set_env("TAVILY_API_KEY")
os.environ['TOKENIZERS_PARALLELISM'] = 'true'
```

### 트레이싱(Tracing)

트레이싱에는 [LangSmith](https://www.langchain.com/langsmith)를 사용합니다.

```python
_set_env("LANGCHAIN_API_KEY")
os.environ["LANGCHAIN_TRACING_V2"] = 'true'
os.environ["LANGCHAIN_PROJECT"] = 'local-llama32-rag'
```

### 웹검색 도구(Web Search Tool)

```python
### Search
from langchain_community.tools.tavily_search import TavilySearchResults

web_search_tool = TavilySearchResults(k=3);
```

### 벡터스토어(Vectorstore)

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import WebBaseLoader
from langchain_community.vectorstores import SKLearnVectorStore
from langchain_nomic.embeddings import NomicEmbeddings

urls = [
    "https://lilianweng.github.io/posts/2023-06-23-agent/",
    "https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/",
    "https://lilianweng.github.io/posts/2023-10-25-adv-attack-llm/",
]

# 문서 로더
docs = WebBaseLoader(urls).load()

# 문서 분할
text_splitter = RecursiveCharacterTextSplitter.from_tiktoken_encoder(
    chunk_size = 1000,
    chunk_overlap=200,
)
docs_splits = text_splitter.split_documents(docs)

# 벡터스토어 생성
vectorstore = SKLearnVectorStore.from_documents(
    embedding = NomicEmbeddings(model="nomic-embed-text-v1.5", inference_mode="local"), 
    documents = docs_splits,
)

# 리트리버 생성
retriever = vectorstore.as_retriever(k=3)
```

    USER_AGENT environment variable not set, consider setting it to identify your requests.

```python
retriever.invoke("agent memory")
```

    [Document(metadata={'id': 'fa10df4b-5401-4ec0-a1c4-eb57454eeee0', 'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'title': "LLM Powered Autonomous Agents | Lil'Log", 'description': 'Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies,⋯ 
    

### 컴포넌트

웹검색 또는 벡터스토어 검색으로 라우터하는 프롬프트를 작성합니다.

```python
### Router
import json
from langchain_core.messages import HumanMessage, SystemMessage

# Prompt 
router_instructions = """You are an expert at routing a user question to a vectorstore or web search.

The vectorstore contains documents related to agents, prompt engineering, and adversarial attacks.
                                    
Use the vectorstore for questions on these topics. For all else, and especially for current events, use web-search.

Return JSON with single key, datasource, that is 'websearch' or 'vectorstore' depending on the question."""

# 번역
"""
당신은 사용자 질문을 벡터 스토어 또는 웹 검색으로 라우팅하는 전문가입니다.
벡터스토어에는 에이전트, 프롬프트 엔지니어링 및 적대적 공격과 관련된 문서가 포함되어 있습니다.    
이러한 주제에 대한 질문은 벡터스토어를 사용하세요. 그 외의 모든 질문, 특히 최신 이슈에 대해서는 웹 검색을 사용하세요.
질문에 따라 single key, datasource 즉 'websearch' 또는 'vectorstore'가 포함된 JSON을 반환합니다.
"""
```

```python
# Test router
question = HumanMessage(content="What are the types of agent memory?")
test_vector_store = llm_json_mode.invoke([SystemMessage(router_instructions), question])
json.loads(test_vector_store.content)
```
    {'datasource': 'vectorstore'}


평가자를 위한 프롬프트를 작성합니다.

```python
### 리트리버 평가자 

# 문서 평가자 프롬프트 
doc_grader_instructions = """You are a grader assessing relevance of a retrieved document to a user question.

If the document contains keyword(s) or semantic meaning related to the question, grade it as relevant."""

# 번역
"""
평가자는 검색된 문서와 사용자 질문의 관련성을 평가합니다.
문서에 질문과 관련된 키워드나 의미론적 의미가 포함되어 있으면 관련성이 있는 것으로 평가합니다.
"""

# 최종 평가자 프롬프트
doc_grader_prompt = """Here is the retrieved document: \n\n {document} \n\n Here is the user question: \n\n {question}. 

This carefully and objectively assess whether the document contains at least some information that is relevant to the question.

Return JSON with single key, binary_score, that is 'yes' or 'no' score to indicate whether the document contains at least some information that is relevant to the question."""

# 번역
"""
검색된 문서는 다음과 같습니다: \n\n {document} \n\n 다음은 사용자 질문입니다: \n\n {question}. 
이렇게 하면 문서에 질문과 관련된 정보가 최소한 일부라도 포함되어 있는지 여부를 신중하고 객관적으로 평가합니다.
문서에 질문과 관련된 정보가 적어도 일부 포함되어 있는지 여부를 나타내는 'yes' 또는 'no' 결과인 single key, binary_score가 포함된 JSON을 반환합니다.
"""
```

```python
# Test
question = "What is Chain of thought prompting?" # 생각의 연결 고리란 무엇인가요?

docs = retriever.invoke(question) # 벡터스토어에서 관련 문서 검색
doc_txt = docs[1].page_content
doc_grader_prompt_formatted = doc_grader_prompt.format(document=doc_txt, question=question)

result = llm_json_mode.invoke([
    SystemMessage(content=doc_grader_instructions), 
    HumanMessage(content=doc_grader_prompt_formatted)
])
json.loads(result.content)
```
    {'binary_score': 'yes'}

### Generate

**RAG 프롬프트**
```python
### Generate

# RAG Prompt
rag_prompt = """You are an assistant for question-answering tasks. 

Here is the context to use to answer the question:

{context} 

Think carefully about the above context. 

Now, review the user question:

{question}

Provide an answer to this questions using only the above context. 

Use three sentences maximum and keep the answer concise.

Answer:"""

# 번역
"""
당신은 질문 답변 작업의 어시스턴트입니다. 
다음은 질문에 답변할 때 사용할 컨텍스트입니다:
{context} 

위의 컨텍스트를 주의 깊게 생각해 보세요. 
이제 사용자 질문을 검토합니다:
{question}

위의 컨텍스트만을 사용하여 이 질문에 대한 답변을 제공하세요. 
답변은 최대 세 문장으로 간결하게 작성하세요.
Answer:
"""

# Post-processing
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)
```

```python
# Test
question = "What is Chain of thought prompting?" # 생각의 연결 고리란 무엇인가요?

docs = retriever.invoke(question)
docs_txt = format_docs(docs)

rag_prompt_formatted = rag_prompt.format(context=docs_txt, question=question)
generation = llm.invoke([HumanMessage(content=rag_prompt_formatted)])
print(generation.content)
```
    Chain of Thought (CoT) prompting is a technique used in natural language processing to generate human-like responses by iteratively asking questions and refining the search space through external search queries, such as Wikipedia APIs. CoT prompting involves decomposing problems into multiple thought steps, generating multiple thoughts per step, and evaluating each state using a classifier or majority vote. The goal is to find an optimal instruction that leads to the desired output, which can be achieved by optimizing prompt parameters directly on the embedding space via gradient descent or searching over a pool of model-generated instruction candidates.

### Hallucination Grader

**할루시네이션 평가 프롬프트**

```python
### Hallucination Grader 

# Hallucination grader instructions 
hallucination_grader_instructions = """

You are a teacher grading a quiz. 

You will be given FACTS and a STUDENT ANSWER. 

Here is the grade criteria to follow:

(1) Ensure the STUDENT ANSWER is grounded in the FACTS. 

(2) Ensure the STUDENT ANSWER does not contain "hallucinated" information outside the scope of the FACTS.

Score:

A score of yes means that the student's answer meets all of the criteria. This is the highest (best) score. 

A score of no means that the student's answer does not meet all of the criteria. This is the lowest possible score you can give.

Explain your reasoning in a step-by-step manner to ensure your reasoning and conclusion are correct. 

Avoid simply stating the correct answer at the outset."""

# 번역
"""
당신은 퀴즈를 채점하는 교사입니다. 
사실과 학생 답안이 주어집니다. 
따라야 할 채점 기준은 다음과 같습니다:

(1) 학생 답안이 사실에 근거하고 있는지 확인합니다. 
(2) 학생 답안에 사실의 범위를 벗어난 "잘못된" 정보가 포함되어 있지 않은지 확인합니다.

Score:

yes는 학생의 답안이 모든 기준을 충족함을 의미합니다. 이 Score가 가장 높은(최고) Score입니다. 
no는 학생의 답변이 모든 기준을 충족하지 않음을 의미합니다. 이 Score는 교사가 줄 수 있는 가장 낮은 Score입니다.

추론과 결론이 올바른지 확인하기 위해 단계별로 추론을 설명합니다. 
처음부터 단순히 정답을 말하지 마십시오.
"""

# Grader prompt
hallucination_grader_prompt = """FACTS: \n\n {documents} \n\n STUDENT ANSWER: {generation}. 

Return JSON with two two keys, binary_score is 'yes' or 'no' score to indicate whether the STUDENT ANSWER is grounded in the FACTS. And a key, explanation, that contains an explanation of the score."""

# 번역
"""
**사실**: \n\n {문서} \n\n **학생 답변**: {generation}. 

**학생 답변**이 **사실**에 근거가 있는지 여부를 나타내는 'yes' 또는 'no' 점수인 binary_score 두 개의 키가 있는 JSON을 반환합니다. 그리고 점수에 대한 설명이 포함된 키, 설명.”
"""

# Test using documents and generation from above 
hallucination_grader_prompt_formatted = hallucination_grader_prompt.format(documents=docs_txt, generation=generation.content)
result = llm_json_mode.invoke([SystemMessage(content=hallucination_grader_instructions)] + [HumanMessage(content=hallucination_grader_prompt_formatted)])
json.loads(result.content)
```
    {'binary_score': 'yes',
     'explanation': 'The student answer provides a clear and accurate description of Chain of Thought (CoT) prompting, its components, and its goals. It also mentions various techniques used in CoT prompting, such as external search queries, prompt tuning, and automatic prompt engineering. The answer demonstrates an understanding of the concept and its applications in natural language processing.'}
****

### Answer Grader
```python
### Answer Grader 

# Answer grader instructions 
answer_grader_instructions = """You are a teacher grading a quiz. 

You will be given a QUESTION and a STUDENT ANSWER. 

Here is the grade criteria to follow:

(1) The STUDENT ANSWER helps to answer the QUESTION

Score:

A score of yes means that the student's answer meets all of the criteria. This is the highest (best) score. 

The student can receive a score of yes if the answer contains extra information that is not explicitly asked for in the question.

A score of no means that the student's answer does not meet all of the criteria. This is the lowest possible score you can give.

Explain your reasoning in a step-by-step manner to ensure your reasoning and conclusion are correct. 

Avoid simply stating the correct answer at the outset."""

# Grader prompt
answer_grader_prompt = """QUESTION: \n\n {question} \n\n STUDENT ANSWER: {generation}. 

Return JSON with two two keys, binary_score is 'yes' or 'no' score to indicate whether the STUDENT ANSWER meets the criteria. And a key, explanation, that contains an explanation of the score."""

# Test 
question = "What are the vision models released today as part of Llama 3.2?"
answer = "The Llama 3.2 models released today include two vision models: Llama 3.2 11B Vision Instruct and Llama 3.2 90B Vision Instruct, which are available on Azure AI Model Catalog via managed compute. These models are part of Meta's first foray into multimodal AI and rival closed models like Anthropic's Claude 3 Haiku and OpenAI's GPT-4o mini in visual reasoning. They replace the older text-only Llama 3.1 models."

# Test using question and generation from above 
answer_grader_prompt_formatted = answer_grader_prompt.format(question=question, generation=answer)
result = llm_json_mode.invoke([SystemMessage(content=answer_grader_instructions)] + [HumanMessage(content=answer_grader_prompt_formatted)])
json.loads(result.content)
```
    {'binary_score': 'yes',
     'explanation': "The student's answer helps to answer the question by providing specific details about the vision models released as part of Llama 3.2. The answer mentions two vision models (Llama 3.2 11B Vision Instruct and Llama 3.2 90B Vision Instruct) and their availability on Azure AI Model Catalog via managed compute. Additionally, the student provides context about Meta's first foray into multimodal AI and compares these models to other visual reasoning models like Claude 3 Haiku and GPT-4o mini. This extra information is not explicitly asked for in the question, but it demonstrates a thorough understanding of the topic. The answer also correctly states that these models replace the older text-only Llama 3.1 models, which meets all the criteria specified in the question."}

# 그래프

[LangGraph](https://langchain-ai.github.io/langgraph/)를 사용하여 위의 워크플로우를 그래프로 작성합니다.

### 그래프 상태

그래프 [`state`](https://langchain-ai.github.io/langgraph/concepts/low_level/#state) 스키마에는 우리가 원하는 키가 포함되어 있습니다:

- 그래프의 각 노드에 전달
- 선택적으로 그래프의 각 노드에서 수정합니다.

```python
import operator
from typing_extensions import TypedDict
from typing import List, Annotated

class GraphState(TypedDict):
    """
    Graph state is a dictionary that contains information we want to propagate to, and modify in, each graph node.
    """
    question : str # User question
    generation : str # LLM generation
    web_search : str # Binary decision to run web search
    max_retries : int # Max number of retries for answer generation 
    answers : int # Number of answers generated
    loop_step: Annotated[int, operator.add] 
    documents : List[str] # List of retrieved documents
```

그래프의 각 [노드](https://langchain-ai.github.io/langgraph/concepts/low_level/#nodes)는 단순히 함수입니다:

(1) `state`를 입력으로 받습니다.
(2) `state`를 수정합니다.
(3) 수정된 `state`를 상태 스키마(딕셔너리)에 기록합니다.

각 [엣지](https://langchain-ai.github.io/langgraph/concepts/low_level/#edges)는 그래프의 노드 사이를 라우팅합니다.

```python
from langchain.schema import Document
from langgraph.graph import END


### Nodes
def retrieve(state):
    """
    Retrieve documents from vectorstore

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): New key added to state, documents, that contains retrieved documents
    """
    print("---RETRIEVE---")
    question = state["question"]

    # Write retrieved documents to documents key in state
    documents = retriever.invoke(question)
    return {"documents": documents}


def generate(state):
    """
    Generate answer using RAG on retrieved documents

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): New key added to state, generation, that contains LLM generation
    """
    print("---GENERATE---")
    question = state["question"]
    documents = state["documents"]
    loop_step = state.get("loop_step", 0)
    
    # RAG generation
    docs_txt = format_docs(documents)
    rag_prompt_formatted = rag_prompt.format(context=docs_txt, question=question)
    generation = llm.invoke([HumanMessage(content=rag_prompt_formatted)])
    return {"generation": generation, "loop_step": loop_step+1}


def grade_documents(state):
    """
    Determines whether the retrieved documents are relevant to the question
    If any document is not relevant, we will set a flag to run web search

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): Filtered out irrelevant documents and updated web_search state
    """

    print("---CHECK DOCUMENT RELEVANCE TO QUESTION---")
    question = state["question"]
    documents = state["documents"]
    
    # Score each doc
    filtered_docs = []
    web_search = "No" 
    for d in documents:
        doc_grader_prompt_formatted = doc_grader_prompt.format(document=d.page_content, question=question)
        result = llm_json_mode.invoke([SystemMessage(content=doc_grader_instructions)] + [HumanMessage(content=doc_grader_prompt_formatted)])
        grade = json.loads(result.content)['binary_score']
        # Document relevant
        if grade.lower() == "yes":
            print("---GRADE: DOCUMENT RELEVANT---")
            filtered_docs.append(d)
        # Document not relevant
        else:
            print("---GRADE: DOCUMENT NOT RELEVANT---")
            # We do not include the document in filtered_docs
            # We set a flag to indicate that we want to run web search
            web_search = "Yes"
            continue
    return {"documents": filtered_docs, "web_search": web_search}
    

def web_search(state):
    """
    Web search based based on the question

    Args:
        state (dict): The current graph state

    Returns:
        state (dict): Appended web results to documents
    """

    print("---WEB SEARCH---")
    question = state["question"]
    documents = state.get("documents", [])

    # Web search
    docs = web_search_tool.invoke({"query": question})
    web_results = "\n".join([d["content"] for d in docs])
    web_results = Document(page_content=web_results)
    documents.append(web_results)
    return {"documents": documents}


### Edges
def route_question(state):
    """
    Route question to web search or RAG 

    Args:
        state (dict): The current graph state

    Returns:
        str: Next node to call
    """

    print("---ROUTE QUESTION---")
    route_question = llm_json_mode.invoke([SystemMessage(content=router_instructions)] + [HumanMessage(content=state["question"])])
    source = json.loads(route_question.content)['datasource']
    if source == 'websearch':
        print("---ROUTE QUESTION TO WEB SEARCH---")
        return "websearch"
    elif source == 'vectorstore':
        print("---ROUTE QUESTION TO RAG---")
        return "vectorstore"


def decide_to_generate(state):
    """
    Determines whether to generate an answer, or add web search

    Args:
        state (dict): The current graph state

    Returns:
        str: Binary decision for next node to call
    """

    print("---ASSESS GRADED DOCUMENTS---")
    question = state["question"]
    web_search = state["web_search"]
    filtered_documents = state["documents"]

    if web_search == "Yes":
        # All documents have been filtered check_relevance
        # We will re-generate a new query
        print("---DECISION: NOT ALL DOCUMENTS ARE RELEVANT TO QUESTION, INCLUDE WEB SEARCH---")
        return "websearch"
    else:
        # We have relevant documents, so generate answer
        print("---DECISION: GENERATE---")
        return "generate"


def grade_generation_v_documents_and_question(state):
    """
    Determines whether the generation is grounded in the document and answers question

    Args:
        state (dict): The current graph state

    Returns:
        str: Decision for next node to call
    """

    print("---CHECK HALLUCINATIONS---")
    question = state["question"]
    documents = state["documents"]
    generation = state["generation"]
    max_retries = state.get("max_retries", 3) # Default to 3 if not provided

    hallucination_grader_prompt_formatted = hallucination_grader_prompt.format(documents=format_docs(documents), generation=generation.content)
    result = llm_json_mode.invoke([SystemMessage(content=hallucination_grader_instructions)] + [HumanMessage(content=hallucination_grader_prompt_formatted)])
    grade = json.loads(result.content)['binary_score']

    # Check hallucination
    if grade == "yes":
        print("---DECISION: GENERATION IS GROUNDED IN DOCUMENTS---")
        # Check question-answering
        print("---GRADE GENERATION vs QUESTION---")
        # Test using question and generation from above 
        answer_grader_prompt_formatted = answer_grader_prompt.format(question=question, generation=generation.content)
        result = llm_json_mode.invoke([SystemMessage(content=answer_grader_instructions)] + [HumanMessage(content=answer_grader_prompt_formatted)])
        grade = json.loads(result.content)['binary_score']
        if grade == "yes":
            print("---DECISION: GENERATION ADDRESSES QUESTION---")
            return "useful"
        elif state["loop_step"] <= max_retries:
            print("---DECISION: GENERATION DOES NOT ADDRESS QUESTION---")
            return "not useful"
        else:
            print("---DECISION: MAX RETRIES REACHED---")
            return "max retries"  
    elif state["loop_step"] <= max_retries:
        print("---DECISION: GENERATION IS NOT GROUNDED IN DOCUMENTS, RE-TRY---")
        return "not supported"
    else:
        print("---DECISION: MAX RETRIES REACHED---")
        return "max retries"
```

## Control Flow

```python
from langgraph.graph import StateGraph
from IPython.display import Image, display

workflow = StateGraph(GraphState)

# Define the nodes
workflow.add_node("websearch", web_search) # web search
workflow.add_node("retrieve", retrieve) # retrieve
workflow.add_node("grade_documents", grade_documents) # grade documents
workflow.add_node("generate", generate) # generate

# Build graph
workflow.set_conditional_entry_point(
    route_question,
    {
        "websearch": "websearch",
        "vectorstore": "retrieve",
    },
)
workflow.add_edge("websearch", "generate")
workflow.add_edge("retrieve", "grade_documents")
workflow.add_conditional_edges(
    "grade_documents",
    decide_to_generate,
    {
        "websearch": "websearch",
        "generate": "generate",
    },
)
workflow.add_conditional_edges(
    "generate",
    grade_generation_v_documents_and_question,
    {
        "not supported": "generate",
        "useful": END,
        "not useful": "websearch",
        "max retries": END,
    },
)

# Compile
graph = workflow.compile()
display(Image(graph.get_graph().draw_mermaid_png()))
```

![](https://i.imgur.com/T31QaYz.png)


```python
inputs = {"question": "What are the types of agent memory?", "max_retries": 3}
for event in graph.stream(inputs, stream_mode="values"):
    print(event)
```
    ---ROUTE QUESTION---
    ---ROUTE QUESTION TO RAG---
    {'question': 'What are the types of agent memory?', 'max_retries': 3, 'loop_step': 0}
    ---RETRIEVE---
    {'question': 'What are the types of agent memory?', 'max_retries': 3, 'loop_step': 0, 'documents': [Document(metadata={'id': '7222226d-772f-4696-b694-25fed7f3df27', 'source': 'https://lilianweng.github.io/posts/2023-06-23-agent/', 'title': "LLM Powered Autonomous Agents | Lil'Log", 'description': 'Building agents with LLM (large language model) as its core controller is a cool concept⋯ 
>   Trace: [https://smith.langchain.com/public/1e01baea-53e9-4341-a6d1-b1614a800a97/r](https://smith.langchain.com/public/1e01baea-53e9-4341-a6d1-b1614a800a97/r)

```python
# Test on current events
inputs = {"question": "What are the models released today for llama3.2?", "max_retries": 3}
for event in graph.stream(inputs, stream_mode="values"):
    print(event)
```
    ---ROUTE QUESTION---
    ---ROUTE QUESTION TO WEB SEARCH---
    {'question': 'What are the models released today for llama3.2?', 'max_retries': 3, 'loop_step': 0}
    ---WEB SEARCH---
    {'question': 'What are the models released today for llama3.2?', 'max_retries': 3, 'loop_step': 0, 'documents': [Document(metadata={}, page_content='Meta’s Llama 3.2 models now available on watsonx, including multimodal 11B and 90B models | IBM Meta Llama 3.2 models now available on watsonx, including multimodal 11B and 90B models IBM is announcing the ⋯
>   Trace: [https://smith.langchain.com/public/acdfa49d-aa11-48fb-9d9c-13a687ff311f/r](https://smith.langchain.com/public/acdfa49d-aa11-48fb-9d9c-13a687ff311f/r)
