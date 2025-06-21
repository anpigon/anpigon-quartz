---
dg-publish: true
title: LangChain v0.3 릴리즈 소식
tags:
  - langchain
created: 2025-05-03 04:42:41
updated: 2025-05-06 01:23:29
---
> *출처: [https://python.langchain.com/docs/versions/v0_3](https://python.langchain.com/docs/versions/v0_3/)*

_최종 업데이트: 2024년 9월 16일_

## 변경 사항

- 모든 패키지가 내부적으로 Pydantic 1에서 Pydantic 2로 업그레이드되었습니다. 사용자 코드에서 Pydantic 2를 사용하는 것이 완벽하게 지원되며, `langchain_core.pydantic_v1` 또는 `pydantic.v1`과 같은 브리지를 사용할 필요가 없습니다.
- Pydantic 1은 2024년 6월에 수명이 다하여 더 이상 지원되지 않습니다.
- Python 3.8은 2024년 10월에 수명이 다하여 더 이상 지원되지 않습니다.

**위 변경 사항이 유일한 호환성 변경 사항입니다.**

## 새로운 기능

0.2.x 개발 중에 다음 기능이 추가되었습니다:

- 더 많은 통합 기능(integrations)을 `langchain-community`에서 자체 `langchain-x` 패키지로 옮겼습니다.  레거시 구현은 `langchain-community`에 남아 있으며 사용 중단(deprecated)된 것으로 표시되므로 이는 호환성에 영향을 미치지 않는 변경입니다. 이를 통해 이러한 통합 기능의 종속성, 테스트 및 버전을 더 효율적으로 관리할 수 있습니다. API 참조에서 모든 최신 통합 패키지를 확인할 수 있습니다.
- 간소화된 도구 정의 및 사용법. 자세한 내용은 [여기](https://blog.langchain.dev/improving-core-tool-interfaces-and-docs-in-langchain/)에서 확인하세요.
- 채팅 모델과 상호 작용하기 위한 유틸리티가 추가되었습니다: [universal model constructor](https://python.langchain.com/v0.2/docs/how_to/chat_models_universal_init/), [rate limiter](https://python.langchain.com/v0.2/docs/how_to/chat_model_rate_limiting/), [message utilities](https://python.langchain.com/v0.2/docs/how_to/#messages)
- 사용자 지정 이벤트를 디스패치하는 기능 [dispatch custom events](https://python.langchain.com/v0.2/docs/how_to/callbacks_custom_events/)이 추가되었습니다.
- 통합 문서 및 API 참조가 개편되었습니다. 자세한 내용은 [여기](https://blog.langchain.dev/langchain-integration-docs-revamped/)에서 확인하세요.
- 여러 레거시 체인이 더 이상 사용되지 않는 것으로 표시되었으며, 모든 체인에 대한 마이그레이션 가이드를 추가했습니다. 이러한 체인은 `langchain` 1.0.0에서 제거될 예정입니다. 더 이상 사용되지 않는 체인 및 관련 마이그레이션 가이드는 [여기](https://python.langchain.com/v0.2/docs/versions/migrating_chains/)에서 확인하세요.

## 코드 업데이트 방법

`langchain`/`langchain-community`/`langchain-core` 0.0 또는 0.1을 사용 중이라면 먼저 [0.2로 업그레이드](https://python.langchain.com/v0.2/docs/versions/v0_2/)하는 것이 좋습니다.

`langgraph`를 사용하는 경우 `langgraph>=0.2.20,<0.3`으로 업그레이드하세요. 이렇게 하면 모든 기본 패키지의 0.2 또는 0.3 버전에서 작동합니다.

다음은 릴리스된 모든 패키지의 전체 목록과 버전 제약 조건 업그레이드를 권장하는 패키지입니다. 이제 `langchain-core` 0.3이 필요한 모든 패키지의 마이너 버전이 증가했습니다. 이제 `langchain-core` 0.2 및 0.3과 호환되는 모든 패키지의 패치 버전이 증가했습니다.

더 이상 사용되지 않는 임포트를 자동으로 업데이트하려면 `langchain-cli`를 사용하면 됩니다. CLI는 LangChain 0.0.x 및 LangChain 0.1에서 도입된 더 이상 사용되지 않는 임포트 업데이트와 `langchain_core.pydantic_v1` 및 `langchain.pydantic_v1` 임포트 업데이트를 처리합니다.

### 기본(Base) 패키지

| Package                  | Latest | Recommended constraint |
| ------------------------ | ------ | ---------------------- |
| langchain                | 0.3.0  | \>=0.3,<0.4            |
| langchain-community      | 0.3.0  | \>=0.3,<0.4            |
| langchain-text-splitters | 0.3.0  | \>=0.3,<0.4            |
| langchain-core           | 0.3.0  | \>=0.3,<0.4            |
| langchain-experimental   | 0.3.0  | \>=0.3,<0.4            |

### 다운스트림(Downstream) 패키지

| Package | Latest | Recommended constraint |
| --- | --- | --- |
| langgraph | 0.2.20 | \>=0.2.20,<0.3 |
| langserve | 0.3.0 | \>=0.3,<0.4 |

### 통합(Integration) 패키지
최신 버전의 패키지로 업데이트한 후에는 Pydantic v1에서 Pydantic v2로의 내부 전환으로 인해 발생하는 다음과 같은 문제를 해결해야 할 수 있습니다.

| Package | Latest | Recommended constraint |
| --- | --- | --- |
| langchain-ai21 | 0.2.0 | \>=0.2,<0.3 |
| langchain-aws | 0.2.0 | \>=0.2,<0.3 |
| langchain-anthropic | 0.2.0 | \>=0.2,<0.3 |
| langchain-astradb | 0.4.1 | \>=0.4.1,<0.5 |
| langchain-azure-dynamic-sessions | 0.2.0 | \>=0.2,<0.3 |
| langchain-box | 0.2.0 | \>=0.2,<0.3 |
| langchain-chroma | 0.1.4 | \>=0.1.4,<0.2 |
| langchain-cohere | 0.3.0 | \>=0.3,<0.4 |
| langchain-elasticsearch | 0.3.0 | \>=0.3,<0.4 |
| langchain-exa | 0.2.0 | \>=0.2,<0.3 |
| langchain-fireworks | 0.2.0 | \>=0.2,<0.3 |
| langchain-groq | 0.2.0 | \>=0.2,<0.3 |
| langchain-google-community | 2.0.0 | \>=2,<3 |
| langchain-google-genai | 2.0.0 | \>=2,<3 |
| langchain-google-vertexai | 2.0.0 | \>=2,<3 |
| langchain-huggingface | 0.1.0 | \>=0.1,<0.2 |
| langchain-ibm | 0.2.0 | \>=0.2,<0.3 |
| langchain-milvus | 0.1.6 | \>=0.1.6,<0.2 |
| langchain-mistralai | 0.2.0 | \>=0.2,<0.3 |
| langchain-mongodb | 0.2.0 | \>=0.2,<0.3 |
| langchain-nomic | 0.1.3 | \>=0.1.3,<0.2 |
| langchain-ollama | 0.2.0 | \>=0.2,<0.3 |
| langchain-openai | 0.2.0 | \>=0.2,<0.3 |
| langchain-pinecone | 0.2.0 | \>=0.2,<0.3 |
| langchain-postgres | 0.0.13 | \>=0.0.13,<0.1 |
| langchain-prompty | 0.1.0 | \>=0.1,<0.2 |
| langchain-qdrant | 0.1.4 | \>=0.1.4,<0.2 |
| langchain-redis | 0.1.0 | \>=0.1,<0.2 |
| langchain-sema4 | 0.2.0 | \>=0.2,<0.3 |
| langchain-together | 0.2.0 | \>=0.2,<0.3 |
| langchain-unstructured | 0.1.4 | \>=0.1.4,<0.2 |
| langchain-upstage | 0.3.0 | \>=0.3,<0.4 |
| langchain-voyageai | 0.2.0 | \>=0.2,<0.3 |
| langchain-weaviate | 0.0.3 | \>=0.0.3,<0.1 |
최신 버전의 패키지로 업데이트한 후에는 Pydantic v1에서 Pydantic v2로 내부 전환으로 인해 발생하는 다음 문제를 해결해야 할 수 있습니다:

- 코드가 LangChain 외에 Pydantic에 의존하는 경우, `pydantic>=2,<3`이 되도록 pydantic 버전 제약 조건을 업그레이드해야 합니다. pydantic v1을 사용하는 경우, LangChain이 아닌 코드를 Pydantic v2로 마이그레이션하는 데 도움이 필요하면 [Pydantic 마이그레이션 가이드](https://docs.pydantic.dev/latest/migration/)를 참조하세요.
- Pydantic v1에서 v2로의 내부 전환으로 인해 LangChain 구성 요소에 여러 가지 부작용이 발생할 수 있습니다. 아래에 몇 가지 일반적인 사례와 권장 솔루션을 나열했습니다.

## Pydantic 2로 전환할 때 발생하는 일반적인 문제

### 1. `langchain_core.pydantic_v1` 네임스페이스를 사용하지 마세요

`langchain_core.pydantic_v1` 또는 `langchain.pydantic_v1`를 모두 `pydantic` 임포트로 대체합니다.

예를 들어,

```python
from langchain_core.pydantic_v1 import BaseModel
```

다음으로 변경:

```python
from pydantic import BaseModel
```

Pydantic 2에는 여러 가지 호환성 변경 사항이 있으므로 Pydantic 코드를 추가로 업데이트해야 할 수도 있습니다. Pydantic 1에서 2로 코드를 업그레이드하는 방법은 [Pydantic 마이그레이션](https://docs.pydantic.dev/latest/migration/)을 참조하세요.

### 2. Pydantic 객체를 LangChain API에 전달하기

다음 API를 사용하는 사용자:

- `BaseChatModel.bind_tools`
- `BaseChatModel.with_structured_output`
- `Tool.from_function`
- `StructuredTool.from_function`

위 API를 호출하는 경우 Pydantic 1 객체(pydantic 2의 `pydantic.v1` 네임스페이스를 통해 생성됨)가 아닌 Pydantic 2 객체를 API에 전달하도록 해야 합니다.

> **주의**: 이러한 API 중 일부에서는 `v1` 객체를 허용할 수 있지만, 향후 문제를 방지하기 위해 사용자는 Pydantic 2 객체를 사용하는 것이 좋습니다.

### 3. LangChain 모델 서브클래싱하기

기존 LangChain 모델(예: `BaseTool`, `BaseChatModel`, `LLM`)의 모든 서브클래싱(sub-classing)은 Pydantic 2 기능을 사용하도록 업그레이드해야 합니다.

예를 들어, Pydantic 1 기능(예: `validator`)에 의존하는 모든 사용자 코드는 Pydantic 2에 해당하는 기능(예: `field_validator`)으로 업데이트해야 하며, `pydantic.v1`, `langchain_core.pydantic_v1`, `langchain.pydantic_v1` 참조는 `pydantic`에서 가져오는 것으로 대체되어야 합니다.

```python
# pydantic 2가 설치된 경우
from pydantic.v1 import validator, Field 

# pydantic 1가 설치된 경우
# from pydantic import validator, Field
# from langchain_core.pydantic_v1 import validator, Field
# from langchain.pydantic_v1 import validator, Field

class CustomTool(BaseTool): # BaseTool은 v1 코드
    x: int = Field(default=1)

    def _run(*args, **kwargs):
        return "hello"

    @validator('x') # v1 코드
    @classmethod
    def validate_x(cls, x: int) -> int:
        return 1
```

다음으로 변경:

```python
from pydantic import Field, field_validator # pydantic v2
from langchain_core.pydantic_v1 import BaseTool

class CustomTool(BaseTool): # BaseTool은 v1 코드
    x: int = Field(default=1)

    def _run(*args, **kwargs):
        return "hello"

    @field_validator('x') # v2 코드
    @classmethod
    def validate_x(cls, x: int) -> int:
        return 1


CustomTool(
    name='custom_tool',
    description="hello",
    x=1,
)
```

### 4. model_rebuild()

LangChain 모델에서 서브클래싱(sub-classing)할 때, 사용자는 파일에 관련 임포트를 추가하고 모델을 다시 빌드해야 할 수 있습니다.

`model_rebuild`에 대한 자세한 내용은 [여기](https://docs.pydantic.dev/latest/concepts/models/#rebuilding-model-schema)에서 확인할 수 있습니다.

```python
from langchain_core.output_parsers import BaseOutputParser


class FooParser(BaseOutputParser):
    ...
```

새로운 코드:

```python
from typing import Optional as Optional

from langchain_core.output_parsers import BaseOutputParser

class FooParser(BaseOutputParser):
    ...

FooParser.model_rebuild()
```

## langchain-cli를 사용하여 마이그레이션

`langchain-cli`는 코드에서 더 이상 사용되지 않는 LangChain 임포트를 자동으로 업데이트하는 데 도움이 될 수 있습니다.

`langchain-cli`는 더 이상 사용되지 않는 LangChain 임포트만 처리하며, 코드를 pydantic 1에서 pydantic 2로 업그레이드하는 데는 도움이 되지 않는다는 점에 유의하세요.

Pydantic 1에서 2로의 마이그레이션 자체에 대한 도움말은 [Pydantic 마이그레이션 가이드라인](https://docs.pydantic.dev/latest/migration/)을 참고하시기 바랍니다.

0.0.31 버전부터 `langchain-cli`는 코드 모드 적용을 위해 [gritql](https://about.grit.io/)에 의존합니다.

### 설치하기

```sh
pip install -U langchain-cli
langchain-cli --version # <-- 버전이 0.0.31 이상인지 확인하세요.
```

### 사용법

마이그레이션 스크립트가 완벽하지 않으므로 먼저 코드의 백업이 있는지 확인해야 합니다(예: `git`과 같은 버전 관리 도구를 사용).

`langchain-cli`는 LangChain 0.3에서 도입된 `langchain_core.pydantic_v1` 지원 중단과 이전 지원 중단을 처리합니다(예: `from langchain.chat_models import ChatOpenAI`는 `from langchain_openai import ChatOpenAI`가 되어야 함),

마이그레이션 스크립트는 실행당 한 번만 임포트 교체를 적용하므로 **두 번** 실행해야 합니다.

예를 들어, 코드가 여전히 이전 임포트 `from langchain.chat_models import ChatOpenAI`를 사용하고 있다고 가정해 보겠습니다:

첫 번째 실행 후에는 다음과 같이 됩니다: `from langchain_community.chat_models import ChatOpenAI` 두 번째 실행 후에는 다음과 같이 표시됩니다: `from langchain_openai import ChatOpenAI`가 표시됩니다.

```python
# 처음 실행할 때
# from langchain.chat_models import ChatOpenAI를 대체합니다.
langchain-cli migrate --help [path to code] # 도움말
langchain-cli migrate [path to code] # 적용

# 임포트 교체를 더 적용하려면 두 번째로 실행하세요.
langchain-cli migrate --diff [path to code] # 미리 보기
langchain-cli migrate [path to code] # 적용
```

### 기타 옵션

```sh
# 도움말 메뉴 보기
langchain-cli migrate --help
# 적용하지 않고 변경 사항 미리 보기
langchain-cli migrate --diff [path to code]
# 대화형으로 변경 사항 승인
langchain-cli migrate --interactive [path to code]
```