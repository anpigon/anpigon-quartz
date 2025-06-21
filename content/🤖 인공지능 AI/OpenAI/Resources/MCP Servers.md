---
dg-publish: true
cteated: 2025-06-21T15:57:00
tags:
  - open-ai
  - mcp
  - mcp-server
related: "[[OpenAI]]"
---

> 원문: https://platform.openai.com/docs/mcp

> [!TLDR]
> 사용자는 회사 지식을 활용하여 ChatGPT를 맞춤화하기 위해 OpenAI의 개방형 프로토콜인 모델 컨텍스트 프로토콜(MCP) 기반의 원격 서버를 직접 구축하고 ChatGPT에 연결할 수 있으며, ==이 과정에서 안전 및 보경 고려사항을 반드시 이해해야 합니다==.
>
> - MCP 서버는 `검색` 및 `가져오기` 도구를 통해 ChatGPT의 심층 연구 기능을 확장합니다.
> - 서버 구축 시 ==OAuth 및 동적 클라이언트 등록을 통한 인증, 그리고 인트라넷 환경에서는 터널링(예: ngrok)==이 필수적입니다.
> - ==신뢰할 수 없는 MCP 서버 연결==은 숨겨진 지침(프롬프트 인젝션)이나 악의적인 동작을 유발할 수 있으므로, ==연결 전 철저한 실사==와 ==데이터 공유 위험 검토가 중요==합니다.

# MCP 서버

회사 지식을 활용하여 ChatGPT를 맞춤화하려면 사용자 지정 원격 MCP 서버를 구축하고 연결하세요.

[모델 컨텍스트 프로토콜](https://modelcontextprotocol.io/introduction)(MCP)은 AI 모델을 추가 도구 및 지식으로 확장하는 데 업계 표준이 되고 있는 개방형 프로토콜입니다. 모든 데이터 소스 위에 MCP 서버를 구축할 수 있습니다. 이 가이드에서는 ChatGPT에서 사용할 기본적인 원격 MCP 서버를 설정하는 방법을 다룹니다.
## 작동 방식

독점 시스템을 포함한 모든 도구를 ChatGPT의 심층 연구 기능에 연결하여, 직원들이 ChatGPT에서 회사 지식에 접근할 수 있도록 합니다. 일반적인 절차는 다음과 같습니다.

1.  `검색(search)` 도구와 `가져오기(fetch)` 도구를 노출하여 심층 연구에 최적화된 MCP 서버를 구축합니다.
2.  ChatGPT에서 사용자 지정 심층 연구 커넥터를 생성합니다.
3.  ChatGPT가 서비스와 효과적으로 상호 작용할 수 있도록 커넥터 설정에 자세한 사용 지침을 포함합니다.
4.  ChatGPT에서 직접 커넥터를 테스트하고 개선합니다.
5.  선택적으로 (ChatGPT Enterprise, Edu 또는 Team 관리자의 경우), 커넥터를 전체 작업 공간에 게시합니다. 그러면 심층 연구(deep research)에서 추가적인 지식 소스로 나타납니다.

> [!info] **애플리케이션에서 MCP 원격 서버에 접근하고 싶으신가요?**
> 지금 읽고 계신 가이드는 ChatGPT에 연결할 원격 MCP 서버를 생성하는 방법에 대한 것입니다. LLM 애플리케이션에서 도구 네트워크에 API 접근을 하려면, [모델이 MCP 원격 서버를 사용하도록 설정하는 방법](https://platform.openai.com/docs/guides/tools-remote-mcp)을 알아보세요.
## MCP 생태계

MCP 생태계는 아직 초기 단계입니다. 현재 인기 있는 원격 MCP 서버로는 [Cloudflare](https://developers.cloudflare.com/agents/guides/remote-mcp-server/), [HubSpot](https://developers.hubspot.com/mcp), [Intercom](https://developers.intercom.com/docs/guides/mcp), [PayPal](https://developer.paypal.com/tools/mcp-server/), [Pipedream](https://pipedream.com/docs/connect/mcp/openai/), [Plaid](https://plaid.com/docs/mcp/), [Shopify](https://shopify.dev/docs/apps/build/storefront-mcp), [Stripe](https://docs.stripe.com/mcp), [Square](https://developer.squareup.com/docs/mcp), [Twilio](https://github.com/twilio-labs/function-templates/tree/main/mcp-server), [Zapier](https://zapier.com/mcp) 등이 있습니다. 앞으로 더 많은 서버와 이러한 서버를 쉽게 찾을 수 있도록 돕는 레지스트리들이 출시될 것으로 예상됩니다. MCP 프로토콜 자체도 초기 단계이며, 프로토콜이 발전함에 따라 MCP 도구에 더 많은 업데이트를 추가할 예정입니다.

사용자 지정 원격 MCP 서버를 사용하기로 결정하기 전에 [위험 및 안전 정보](https://platform.openai.com/docs/mcp#risks-and-safety)를 이해하는 것이 좋습니다.
## MCP 서버 구축

아직 MCP에 익숙하지 않다면, [MCP 소개](https://modelcontextprotocol.io/introduction)를 읽어보세요. 선호하는 도구 및 라이브러리에서 간단한 서버 지침을 찾을 수 있습니다. 다음은 몇 가지 자료입니다:
-   [Cloudflare](https://developers.cloudflare.com/agents/guides/remote-mcp-server/)
-   [Azure Functions](https://devblogs.microsoft.com/dotnet/build-mcp-remote-servers-with-azure-functions/)
-   [Stainless](https://www.stainless.com/blog/generate-mcp-servers-from-openapi-specs)
### 기본적인 원격 서버 설정

시작점으로, GitHub의 심층 연구 MCP 서버 [샘플 앱](https://github.com/kwhinnery-openai/sample-deep-research-mcp)을 사용하세요. 이 최소한의 예제는 컵케이크 주문을 검색하고 가져오는 원격 MCP 서버를 생성하고 실행하는 방법을 보여줍니다.
1.  [레포지토리](https://github.com/kwhinnery-openai/sample-deep-research-mcp)를 클론하거나 파일을 기존 레포지토리에 복사합니다.
2.  서버를 설정합니다. Python에서는 다음 명령어를 실행할 수 있습니다:
    ```shell
    python -m venv env
    source env/bin/activate
    pip install -r requirements.txt
    ```
3.  서버를 실행합니다. SSE 전송을 사용하여 `http://127.0.0.1:8000`에서 시작됩니다.
    ```shell
    python sample_mcp.py
    ```
4.  필요한 사용자 지정 사항으로 샘플 파일을 업데이트합니다:
    -   `sample_mcp.py`는 메인 서버 코드입니다.
    -   `records.json`은 컵케이크 주문 데이터입니다 (동일한 디렉토리에 있어야 합니다).

MCP 서버는 여러 도구를 가질 수 있습니다. 현재 ChatGPT에서 MCP 서버에 연결하는 것은 사용자가 심층 연구를 수행할 수 있도록 하는 것으로 제한됩니다. 이는 귀하의 MCP 원격 서버가 검색 및 문서 검색 도구를 갖춘 검색 엔진과 유사해야 함을 의미합니다.
### 검색 설정

검색 도구를 정의합니다. 간단한 컵케이크 주문 예제에서는 코드가 다음과 같습니다:

```python
def create_server():
    mcp = FastMCP(name="Cupcake MCP", instructions="Search cupcake orders")

    @mcp.tool()
    async def search(query: str):
        """
        Search for cupcake orders – keyword match.
        """
        toks = query.lower().split()
        ids = []
        for r in RECORDS:
            hay = " ".join(
                [
                    r.get("title", ""),
                    r.get("text", ""),
                    " ".join(r.get("metadata", {}).values()),
                ]
            ).lower()
            if any(t in hay for t in toks):
                ids.append(r["id"])
        return {"ids": ids}
```

#### 검색 의미론
이 도구를 정의하고 MCP 서버를 통해 노출하기 위한 검색 의미론은 익숙한 방식과 약간 다릅니다. 다음은 전체 사양입니다:

```json
{
  "tools": [
    {
      "name": "search",
      "description": "Searches for resources using the provided query string and returns matching results.",
      "input_schema": {
        "type": "object",
        "properties": {
          "query": {"type": "string", "description": "Search query."}
        },
        "required": ["query"]
      },
      "output_schema": {
        "type": "object",
        "properties": {
          "results": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "id": {"type": "string", "description": "ID of the resource."},
                "title": {"type": "string", "description": "Title or headline of the resource."},
                "text": {"type": "string", "description": "Text snippet or summary from the resource."},
                "url": {"type": ["string", "null"], "description": "URL of the resource. Optional but needed for citations to work."},
              },
              "required": ["id", "title", "text"]
            }
          }
        },
        "required": ["results"]
      }
    }
  ]
}
```

#### 모델에 유효한 쿼리 구성 방법 교육
위 사양에서 `description` 필드는 중요합니다. 이 필드에서 심층 연구 모델에게 유효한 검색 쿼리를 구성하기 위해 이 도구를 _어떻게_ 사용해야 하는지 설명합니다.
예를 들어, 원격 MCP 서버가 다음과 같이 하나의 쿼리 문자열로 표현되는 사용자로부터의 복잡한 쿼리 구문을 지원하도록 할 수 있습니다:

```text
type:deals amount:gt:1000
```

이러한 종류의 쿼리 구문을 활성화하려면 `description》필드가 모델에게 이 사용자 입력에서 유효한 쿼리를 형성하는 방법을 알려야 합니다. OpenAI는 심층 연구 모델이 이 검색 도구를 호출할 쿼리 문자열을 형성하도록 프롬프트하는 데 `description》필드에 의존합니다.
이를 사용하여 표현력 있는 쿼리를 구축하세요. 실제 예시로, 다음은 HubSpot의 검색 도구 설명입니다.

```json
Purpose:\\n  1. Search for resources in the HubSpot CRM of a specific object type (only contacts, deals, companies, tickets are supported).\\n  2. Note that only a subset of the properties will be returned.\\n  3. For the complete set of properties, use the Fetch tool.\\n\\nUsage:\\n  1. List a few objects to understand the data model of a specific object type.\\n  2. Search for objects of a specific object type using filters.\\n  3. Make sure to use the offset from the prior search result to call the tool again to paginate through the entire list.\\n  4. Search for associated objects.\\n\\nSearch Tool Response:\\n  1. The Search Tools response contains an idfor each attribute. Note that thisidis not the same as thehs_object_idof the object.\\n  2. Theidcan ONLY be used to fetch the object metadata using the Fetch tool. Use thehs_object_id when creating search queries involving specific objects.\\n', is_consequential=True, params={'type': 'object', 'properties': {'query': {'type': 'string', 'description': 'HubSpot Search Query DSL specification.\\n\\n • Tokens separated by spaces\\n • Each token: key[:op]:value\\n   – key ∈ { object_type, q|query, any HubSpot CRM property name, associated_{object_type} , limit, offset, sort }\\n   – op (optional; default “eq”) ∈ { eq, neq, gt, gte, lt, lte, in, not_in, contains_token, not_contains_token, has_property, not_has_property }\\n   – value:\\n     • unquoted (alphanumeric, no spaces)\\n     • or quoted in single/double quotes (to include spaces)\\n     • for in/not_in: comma-separated list (no spaces)\\n • Semantics:\\n   – object_type (mandatory) → which object to search (only contacts, deals, companies & tickets supported)\\n   – q/query → free-text search on a few key fields unique for each object type.\\n   – propertyName[:op]:value → filters, all AND’d in one group\\n   – associated_{object_type}:id → search for objects associated with the specified object type (e.g. associated_contacts:<hs_object_id of contact>).\\n      You can also use the associated_{object_type} key with in, not_in, eq and neq operators. For example, associated_contacts:in:123,456. No other format is supported.\\n   – limit, offset → integers. offset can help paginate through results, and will be returned in the search API response if more items exist.\\n   – sort:property[:asc|:desc] (default asc)\\n\\n • Searching for associated objects to a given object requires knowing its hs_object_id. If the given object\\'s hs_object_id is not known,\\\\n   then search must be done in two steps:\\n     1. Search for the given object to get its hs_object_id (e.g. for contact, use a query like: object_type:contacts email:someone@example.com )\\n     2. Search for the associated objects using that hs_object_id ( e.g. for the contact with hs_object_id 123, use the filter: associated_contacts:123 )\\n\\n • The q/query attribute searches across a few key fields unique for each object type:\\n  – Contacts: firstname, lastname, email, phone, company, hs_additional_emails, fax\\n  – Companies: name, website, domain, phone\\n  – Deals: dealname, pipeline, dealstage, description, dealtype\\n  – Tickets: subject, content, hs_pipeline_stage, hs_ticket_category, hs_ticket_id\\n\\n EXAMPLES:\\n\\n user: show me my deals\\n query: object_type:deals\\n\\n user: show me all contacts containing \\'John Doe\\'\\n query: object_type:contacts q:\\'John Doe\\'\\n\\n user: Show me the 5 most recently modified contacts.\\n query: object_type:contacts limit:5 offset:0 sort:lastmodifieddate:desc\\n\\n user: Find contacts whose email contains “@example.com”, sorted by the last contacted date.\\n query: object_type:contacts email:contains_token:\"@example.com\" limit:20 offset:0 sort:lastcontacted:desc\\n\\n user: List marketing qualified and sales qualified leads in the U.S.\\n query: object_type:contacts lifecyclestage:in:marketingqualifiedlead,salesqualifiedlead country:US\\n\\n user: Retrieve deals for owner 12345 in Q1 2025 but exclude “Closed Lost”.\\n query: object_type:deals hubspot_owner_id:12345 dealstage:not_in:closedlost closedate:gte:2025-01-01 closedate:lte:2025-03-31 limit:100 offset:0 sort:closedate:asc\\n\\n user: Find tech companies with annual revenue between $1,000,000 and $10,000,000.\\n query: object_type:companies industry:Technology annualrevenue:gte:1000000 annualrevenue:lte:10000000 limit:100 offset:0 sort:annualrevenue:desc\\n\\n user: List contacts that have a phone number defined but no website.\\n query: object_type:contacts phone:has_property website:not_has_property limit:100 offset:0 sort:createdate:asc\\n\\n user: Find companies whose description does not contain “startup”\\n query: object_type:companies description:not_contains_token:\"startup\"\\n\\n user: Find tickets mentioning \\'refund\\'\\n query: object_type:tickets q:\\'refund\\'\\n\\n user: Find all contacts associated with company hs_object_id 12345.\\n query: object_type:contacts associated_companies:12345 limit:100 offset:0 sort:createdate:desc\\n\\n user: Show me deals not associated with contact hs_object_id 54321.\\n query: object_type:deals associated_contacts:neq:54321 limit:100 offset:0 sort:amount:desc\\n\\n user: Find contacts associated with deals with hs_object_ids in 24680,24681 who are in California.\\n query: object_type:contacts associated_deals:in:24680,24681 state:CA limit:100 offset:0\\n\\n Unsupported features:\\n   • Boolean logic: NO OR, NOT, AND keywords or parenthesis grouping\\n   • Relative dates: NO “now–7d”, “today”, “last week” syntax\\n   • Aggregations / metrics: NO count, sum, facet, group‐by\\n   • Fuzzy or proximity: NO “~2” fuzzy match operators\\n   • Nested expressions or sub-queries\\n   • Custom functions or scripts\\n   • Escaping beyond simple single/double quotes\\n   • No support for searching for objects without associations directly. You must list all objects and see if they have associations from contacts, companies, or deals.'}}, 'required': ['query'], 'additionalProperties': False}, return_type={'$defs': {'Annotations': {'additionalProperties': True, 'properties': {'audience': {'anyOf': [{'items': {'enum': ['user', 'assistant'], 'type': 'string'}, 'type': 'array'}, {'type': 'null'}], 'default': None, 'title': 'Audience'}, 'priority': {'anyOf': [{'maximum': 1.0, 'minimum': 0.0, 'type': 'number'}, {'type': 'null'}], 'default': None, 'title': 'Priority'}}, 'title': 'Annotations', 'type': 'object'}, 'BlobResourceContents': {'additionalProperties': True, 'description': 'Binary contents of a resource.', 'properties': {'uri': {'format': 'uri', 'minLength': 1, 'title': 'Uri', 'type': 'string'}, 'mimeType': {'anyOf': [{'type': 'string'}, {'type': 'null'}], 'default': None, 'title': 'Mimetype'}, 'blob': {'title': 'Blob', 'type': 'string'}}, 'required': ['uri', 'blob'], 'title': 'BlobResourceContents', 'type': 'object'}, 'EmbeddedResource': {'additionalProperties': True, 'description': 'The contents of a resource, embedded into a prompt or tool call result.\\n\\nIt is up to the client how best to render embedded resources for the benefit\\nof the LLM and/or the user.', 'properties': {'type': {'const': 'resource', 'title': 'Type', 'type': 'string'}, 'resource': {'anyOf': [{'$ref': '#/$defs/TextResourceContents'}, {'$ref': '#/$defs/BlobResourceContents'}], 'title': 'Resource'}, 'annotations': {'anyOf': [{'$ref': '#/$defs/Annotations'}, {'type': 'null'}], 'default': None}}, 'required': ['type', 'resource'], 'title': 'EmbeddedResource', 'type': 'object'}, 'ImageContent': {'additionalProperties': True, 'description': 'Image content for a message.', 'properties': {'type': {'const': 'image', 'title': 'Type', 'type': 'string'}, 'data': {'title': 'Data', 'type': 'string'}, 'mimeType': {'title': 'Mimetype', 'type': 'string'}, 'annotations': {'anyOf': [{'$ref': '#/$defs/Annotations'}, {'type': 'null'}], 'default': None}}, 'required': ['type', 'data', 'mimeType'], 'title': 'ImageContent', 'type': 'object'}, 'TextContent': {'additionalProperties': True, 'description': 'Text content for a message.', 'properties': {'type': {'const': 'text', 'title': 'Type', 'type': 'string'}, 'text': {'title': 'Text', 'type': 'string', 'title': 'Text', 'type': 'string'}, 'annotations': {'anyOf': [{'$ref': '#/$defs/Annotations'}, {'type': 'null'}], 'default': None}}, 'required': ['type', 'text'], 'title': 'TextContent', 'type': 'object'}, 'TextResourceContents': {'additionalProperties': True, 'description': 'Text contents of a resource.', 'properties': {'uri': {'format': 'uri', 'minLength': 1, 'title': 'Uri', 'type': 'string'}, 'mimeType': {'anyOf': [{'type': 'string'}, {'type': 'null'}], 'default': None, 'title': 'Mimetype'}, 'text': {'title': 'Text', 'type': 'string'}}, 'required': ['uri', 'text'], 'title': 'TextResourceContents', 'type': 'object'}}, 'additionalProperties': True, 'description': "The server's response to a tool call.", 'properties': {'_meta': {'anyOf': [{'additionalProperties': True, 'type': 'object', 'default': None, 'title': 'Meta'}, 'content': {'items': {'anyOf': [{'$ref': '#/$defs/TextContent'}, {'$ref': '#/$defs/ImageContent'}, {'$ref': '#/$defs/EmbeddedResource'}]}, 'title': 'Content', 'type': 'array'}, 'isError': {'default': False, 'title': 'Iserror', 'type': 'boolean'}}, 'required': ['content'], 'title': 'CallToolResult', 'type': 'object'}


### 문서 검색 설정
문서 검색 도구는 인용을 활성화하는 데 도움을 줍니다. 컵케이크 주문 예제에서 문서 검색 코드는 다음과 같습니다:

```python
@mcp.tool()
async def fetch(id: str):
    """
    Fetch a cupcake order by ID.
    """
    if id not in LOOKUP:
        raise ValueError("unknown id")
    return LOOKUP[id]

return mcp
```

다음은 전체 사양입니다:

```json
{
  "tools": [
    {
      "name": "fetch",
      "description": "Retrieves detailed content for a specific resource identified by the given ID.",
      "input_schema": {
        "type": "object",
        "properties": {
          "id": {"type": "string", "description": "ID of the resource to fetch."}
        },
        "required": ["id"]
      },
      "output_schema": {
        "type": "object",
        "properties": {
          "id": {"type": "string", "description": "ID of the resource."},
          "title": {"type": "string", "description": "Title or headline of the fetched resource."},
          "text": {"type": "string", "description": "Complete textual content of the resource."},
          "url": {"type": ["string", "null"], "description": "URL of the resource. Optional but needed for citations to work."},
          "metadata": {
            "type": ["object", "null"],
            "additionalProperties": {"type": "string"},
            "description": "Optional metadata providing additional context."
          }
        },
        "required": ["id", "title", "text"]
      }
    }
  ]
}
```

### 인증 처리

사용자 지정 원격 MCP 서버를 구축하는 사람으로서, 권한 부여와 인증은 데이터를 보호하는 데 도움이 됩니다. OAuth와 [동적 클라이언트 등록](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization#2-4-dynamic-client-registration)을 사용하는 것을 권장합니다. 프로토콜의 인증에 대해 더 자세히 알아보려면 [MCP 사용자 가이드](https://modelcontextprotocol.io/docs/concepts/transports#authentication-and-authorization)를 읽거나 [권한 부여 사양](https://modelcontextprotocol.io/specification/2025-03-26/basic/authorization)을 참조하십시오.
ChatGPT에서 사용자 지정 원격 MCP 서버를 연결하면, 작업 공간의 사용자는 귀하의 애플리케이션으로 OAuth 흐름을 받게 됩니다.
### 전송 및 터널링

원격 MCP 서버는 인터넷 주소 지정이 가능해야 합니다. 따라서 서버가 인트라넷에 호스팅되어 있다면 어떤 형태의 터널링이 필요합니다. [ngrok](https://ngrok.com/)은 이를 위한 편리한 도구 중 하나이지만, Cloudflare와 같은 다른 터널링 솔루션도 있습니다.
### 테스트 및 디버깅

MCP 서버를 테스트하려면, API [플레이그라운드](https://platform.openai.com/playground)을 사용하여 서버에 접근 가능한지, 도구 목록이 예상대로 해결되는지 확인하세요. 플레이그라운드를 사용하여 심층 연구를 실행하기 전에 서버가 결과를 반환하는 능력을 부분적으로 확인할 수 있습니다. 심층 연구는 반복적인 개선 (예: 검색 도구 설명을 개선하는 동안)에 더 시간이 오래 걸립니다.

최상의 결과를 위해, 플레이그라운드에서 OpenAI o3 또는 o3 mini로 테스트하는 것을 권장합니다.
## 원격 MCP 서버 연결
1.  [ChatGPT 설정](https://chatgpt.com/#settings)에서 원격 MCP 서버를 직접 가져옵니다.
2.  **커넥터** 탭에서 서버를 연결합니다. 이제 컴포저 > 심층 연구 도구에서 볼 수 있습니다. 서버를 소스로 추가해야 할 수도 있습니다.
3.  몇 가지 프롬프트를 실행하여 서버를 테스트합니다.
## 위험 및 안전

사용자 지정 MCP 서버를 사용하면 ==ChatGPT 작업 공간을 외부 애플리케이션에 연결==할 수 있으며, 이를 통해 ChatGPT는 이러한 애플리케이션에서 데이터에 접근하고, 데이터를 송수신하며, 조치를 취할 수 있습니다. 사용자 지정 MCP 서버는 ==OpenAI에서 개발하거나 검증한 것이 아니==며, 자체 이용 약관이 적용되는 타사 서비스입니다.

악의적인 MCP 서버를 발견하면 [security@openai.com](mailto:security@openai.com)으로 신고해 주십시오.
### 신뢰할 수 있는 서버 연결

ChatGPT 작업 공간에 추가하는 사용자 지정 MCP 서버에 주의하십시오... ==기반 애플리케이션을 알고 신뢰하지 않는 한 사용자 지정 MCP 서버에 연결하지 않는 것을 권장합니다==... ==악의적인 MCP 서버에는 ChatGPT가 예기치 않게 동작하도록 설계된 숨겨진 지침(프롬프트 인젝션)==이 포함될 수 있습니다... ==어떤 MCP 서버에든 연결하기 전에 공유될 데이터 유형을 신중하고 철저하게 검토하십시오==.

### 서버 구축

==접근을 허용하는 데이터에 주의하십시오==... ==도구의 JSON에 민감한 정보를 넣지 마십시오==. 또한 ==원격 MCP 서버에 접근하는 ChatGPT 사용자로부터 민감한 정보를 저장하지 마십시오==.

MCP 서버를 구축하는 사람으로서, 도구 정의에 악의적인 내용을 넣지 마십시오. 현재로서는 검색 및 문서 검색만 지원합니다.
## 다른 사용자를 위한 MCP 서버 배포

대기업은 ChatGPT의 심층 연구 기능을 통해 다른 사람들이 회사 지식을 사용할 수 있도록 MCP 서버를 배포하기를 원할 수 있습니다. MCP 서버를 배포하려면 관리자와 협력하십시오.