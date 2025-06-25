---
related:
title: "Anthropic Claude Code SDK: 개발자를 위한 AI 코딩 통합"
created: 2025-06-13 12:09:26
updated: 2025-06-13 12:09:26
tags: []
source_url: https://docs.anthropic.com/en/docs/claude-code/sdk
MOC: ["[[2025-06-13]]"]
aliases: Anthropic_Claude_Code_SDK_개발자를_위한_AI_코딩_통합
modified: 2025-06-25 11:22:14
---

> [!summary]
> Anthropic의 Claude Code SDK는 개발자가 Claude의 코딩 능력을 애플리케이션에 통합할 수 있도록 지원합니다. 이 SDK는 명령줄, TypeScript, Python을 통해 사용할 수 있으며, Claude Code를 서브 프로세스로 실행하여 AI 기반 코딩 도우미를 구축할 수 있게 합니다. API 키 인증, 기본 사용법, 다중 턴 대화, 사용자 정의 시스템 프롬프트, MCP 구성, CLI 옵션, 출력 형식 등 다양한 기능을 제공하며, 실제 애플리케이션 통합 및 관련 리소스에 대한 정보도 포함합니다.

Claude Code SDK를 통해 Claude Code를 애플리케이션에 프로그램 방식으로 통합하는 방법에 대해 설명합니다.

Claude Code SDK는 개발자가 Claude Code를 애플리케이션에 프로그램 방식으로 통합할 수 있도록 허용합니다. 이 SDK는 Claude Code를 서브 프로세스로 실행하여 Claude의 기능을 활용하는 AI 기반 코딩 도우미 및 도구를 구축하는 방법을 제공합니다.

이 SDK는 명령줄, TypeScript, Python에서 사용 가능합니다.

## 인증

Claude Code SDK를 사용하려면 전용 API 키를 생성하는 것이 좋습니다.
- Anthropic Console에서 Anthropic API 키를 생성하십시오.
- 그런 다음, `ANTHROPIC_API_KEY` 환경 변수를 설정하십시오. 이 키는 안전하게 저장하는 것을 권장합니다(예: GitHub 시크릿 사용).

## SDK 기본 사용법

Claude Code SDK를 통해 애플리케이션에서 Claude Code를 비대화형 모드로 사용할 수 있습니다.

#### 명령줄

다음은 명령줄 SDK의 몇 가지 기본 예시입니다.

```shell
# 단일 프롬프트를 실행하고 종료합니다 (인쇄 모드).
$ claude -p "피보나치 숫자를 계산하는 함수를 작성하십시오."
# 파이프를 사용하여 표준 입력을 제공합니다.
$ echo "이 코드를 설명하십시오." | claude -p
# 메타데이터와 함께 JSON 형식으로 출력합니다.
$ claude -p "Hello World 함수를 생성하십시오." --output-format json
# JSON 출력이 도착하는 대로 스트리밍합니다.
$ claude -p "React 컴포넌트를 빌드하십시오." --output-format stream-json
```

#### TypeScript

TypeScript SDK는 NPM의 주요 `@anthropic-ai/claude-code` 패키지에 포함되어 있습니다.

```typescript
import { query, type SDKMessage } from "@anthropic-ai/claude-code";

const messages: SDKMessage[] = [];

for await (const message of query({
  prompt: "foo.py에 대한 하이쿠를 작성하십시오.",
  abortController: new AbortController(),
  options: {
    maxTurns: 3,
  },
})) {
  messages.push(message);
}

console.log(messages);
```

TypeScript SDK는 명령줄 SDK에서 지원하는 모든 인수를 허용하며, 다음도 포함합니다.
- `abortController`: Abort 컨트롤러입니다. 기본값은 `new AbortController()`입니다.
- `cwd`: 현재 작업 디렉터리입니다. 기본값은 `process.cwd()`입니다.
- `executable`: 사용할 JavaScript 런타임입니다. Node.js에서 실행 시 `node`, Bun에서 실행 시 `bun`입니다.
- `executableArgs`: 실행 파일에 전달할 인수입니다. 기본값은 `[]`입니다.
- `pathToClaudeCodeExecutable`: Claude Code 실행 파일의 경로입니다. 기본값은 `@anthropic-ai/claude-code`와 함께 제공되는 실행 파일입니다.

#### Python

Python SDK는 PyPI에서 `claude-code-sdk`로 사용 가능합니다.

```shell
pip install claude-code-sdk
```

사전 요구 사항은 다음과 같습니다.
- Python 3.10 이상
- Node.js
- Claude Code CLI: `npm install -g @anthropic-ai/claude-code`

기본 사용법은 다음과 같습니다.

```python
import anyio
from claude_code_sdk import query, ClaudeCodeOptions, Message

async def main():
    messages: list[Message] = []

    async for message in query(
        prompt="foo.py에 대한 하이쿠를 작성하십시오.",
        options=ClaudeCodeOptions(max_turns=3)
    ):
        messages.append(message)

    print(messages)

anyio.run(main)
```

Python SDK는 `ClaudeCodeOptions` 클래스를 통해 명령줄 SDK에서 지원하는 모든 인수를 허용합니다.

```python
from claude_code_sdk import query, ClaudeCodeOptions
from pathlib import Path

options = ClaudeCodeOptions(
    max_turns=3,
    system_prompt="당신은 유용한 비서입니다.",
    cwd=Path("/path/to/project"),  # 문자열 또는 Path일 수 있습니다.
    allowed_tools=["Read", "Write", "Bash"],
    permission_mode="acceptEdits"
)

async for message in query(prompt="안녕하세요", options=options):
    print(message)
```

## 고급 사용법

아래 문서는 명령줄 SDK를 예시로 사용하지만, TypeScript 및 Python SDK에서도 사용할 수 있습니다.

#### 다중 턴 대화

다중 턴 대화의 경우, 대화를 재개하거나 가장 최근 세션에서 계속할 수 있습니다.

```shell
# 가장 최근 대화를 계속합니다.
$ claude --continue

# 계속하고 새로운 프롬프트를 제공합니다.
$ claude --continue "이제 더 나은 성능을 위해 이것을 리팩터링하십시오."
# 세션 ID로 특정 대화를 재개합니다.
$ claude --resume 550e8400-e29b-41d4-a716-446655440000

# 인쇄 모드(비대화형)로 재개합니다.
$ claude -p --resume 550e8400-e29b-41d4-a716-446655440000 "테스트를 업데이트하십시오."
# 인쇄 모드(비대화형)로 계속합니다.
$ claude -p --continue "오류 처리를 추가하십시오."
```

#### 사용자 정의 시스템 프롬프트

Claude의 동작을 안내하기 위해 사용자 정의 시스템 프롬프트를 제공할 수 있습니다.

```shell
# 시스템 프롬프트를 재정의합니다 (--print에서만 작동합니다).
$ claude -p "REST API를 빌드하십시오." --system-prompt "당신은 선임 백엔드 엔지니어입니다. 보안, 성능 및 유지보수성에 중점을 두십시오."
# 특정 요구사항이 있는 시스템 프롬프트입니다.
$ claude -p "데이터베이스 스키마를 생성하십시오." --system-prompt "당신은 데이터베이스 아키텍트입니다. PostgreSQL 모범 사례를 사용하고 적절한 인덱싱을 포함하십시오."
```

기본 시스템 프롬프트에 지침을 추가할 수도 있습니다.

```shell
# 시스템 프롬프트를 추가합니다 (--print에서만 작동합니다).
$ claude -p "REST API를 빌드하십시오." --append-system-prompt "코드를 작성한 후에는 반드시 직접 코드 검토를 하십시오."
```

#### MCP 구성

모델 컨텍스트 프로토콜(MCP)을 사용하면 외부 서버의 추가 도구 및 리소스로 Claude Code를 확장할 수 있습니다. `--mcp-config` 플래그를 사용하여 데이터베이스 액세스, API 통합 또는 사용자 정의 도구와 같은 특수 기능을 제공하는 MCP 서버를 로드할 수 있습니다.

MCP 서버가 포함된 JSON 구성 파일을 생성하십시오.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/files"
      ]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    }
  }
}
```

그런 다음 Claude Code에서 사용하십시오.

```shell
# 구성에서 MCP 서버를 로드합니다.
$ claude -p "프로젝트의 모든 파일을 나열하십시오." --mcp-config mcp-servers.json
# 중요: MCP 도구는 --allowedTools를 사용하여 명시적으로 허용되어야 합니다.
# MCP 도구는 mcp__$serverName__$toolName 형식을 따릅니다.
$ claude -p "TODO 주석을 검색하십시오." \
  --mcp-config mcp-servers.json \
  --allowedTools "mcp__filesystem__read_file,mcp__filesystem__list_directory"

# 비대화형 모드에서 권한 프롬프트를 처리하기 위해 MCP 도구를 사용합니다.
$ claude -p "애플리케이션을 배포하십시오." \
  --mcp-config mcp-servers.json \
  --allowedTools "mcp__permissions__approve" \
  --permission-prompt-tool mcp__permissions__approve
```

> [!info]
> MCP 도구를 사용할 때는 `--allowedTools` 플래그를 사용하여 명시적으로 허용해야 합니다. MCP 도구 이름은 `mcp__<serverName>__<toolName>` 패턴을 따르며, 여기서 `serverName`은 MCP 구성 파일의 키이고 `toolName`은 해당 서버에서 제공하는 특정 도구입니다.
>
> 이 보안 조치는 MCP 도구가 명시적으로 허용될 때만 사용되도록 보장합니다.
>
> 서버 이름만 지정하는 경우(예: `mcp__<serverName>`), 해당 서버의 모든 도구가 허용됩니다.
>
> 전역 패턴(예: `mcp__go*`)은 지원되지 않습니다.

#### 사용자 정의 권한 프롬프트 도구

선택적으로 `--permission-prompt-tool`을 사용하여 사용자가 특정 도구를 호출할 권한을 모델에 부여하는지 확인할 때 사용할 MCP 도구를 전달할 수 있습니다. 모델이 도구를 호출하면 다음이 발생합니다.

1. 먼저 권한 설정을 확인합니다. 모든 `settings.json` 파일뿐만 아니라 SDK에 전달된 `--allowedTools` 및 `--disallowedTools`를 확인합니다. 이 중 하나가 도구 호출을 허용하거나 거부하면 도구 호출을 진행합니다.
2. 그렇지 않으면 `--permission-prompt-tool`에 제공한 MCP 도구를 호출합니다.

`--permission-prompt-tool` MCP 도구는 도구 이름과 입력을 전달받으며, 결과와 함께 JSON 문자열화된 페이로드를 반환해야 합니다. 페이로드는 다음 중 하나여야 합니다.

```json
// 도구 호출이 허용됩니다.
{
  "behavior": "allow",
  "updatedInput": {...}, // 업데이트된 입력 또는 원본 입력 반환
}

// 도구 호출이 거부됩니다.
{
  "behavior": "deny",
  "message": "..." // 권한이 거부된 이유를 설명하는 사람이 읽을 수 있는 문자열
}
```

예를 들어, TypeScript MCP 권한 프롬프트 도구 구현은 다음과 같을 수 있습니다.

```typescript
const server = new McpServer({
  name: "Test permission prompt MCP Server",
  version: "0.0.1",
});

server.tool(
  "approval_prompt",
  '권한 확인을 시뮬레이션합니다 - 입력에 "allow"가 포함되어 있으면 승인하고, 그렇지 않으면 거부합니다.',
  {
    tool_name: z.string().describe("권한을 요청하는 도구"),
    input: z.object({}).passthrough().describe("도구에 대한 입력"),
  },
  async ({ tool_name, input }) => {
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(
            JSON.stringify(input).includes("allow")
              ? {
                  behavior: "allow",
                  updatedInput: input,
                }
              : {
                  behavior: "deny",
                  message: "테스트 승인 프롬프트 도구에 의해 권한이 거부되었습니다.",
                }
          ),
        },
      ],
    };
  }
);
```

이 도구를 사용하려면 MCP 서버를 추가한 다음(예: `--mcp-config` 사용) 다음과 같이 SDK를 호출하십시오.

```shell
claude -p "..." \
  --permission-prompt-tool mcp__test-server__approval_prompt \
  --mcp-config my-config.json
```

사용법 참고 사항은 다음과 같습니다.

- `updatedInput`을 사용하여 권한 프롬프트가 입력을 변경했음을 모델에 알리십시오. 그렇지 않으면 위의 예시처럼 `updatedInput`을 원본 입력으로 설정하십시오. 예를 들어, 도구가 사용자에게 파일 편집 차이를 보여주고 수동으로 차이를 편집하도록 허용하는 경우, 권한 프롬프트 도구는 해당 업데이트된 편집을 반환해야 합니다.
- 페이로드는 JSON으로 문자열화되어야 합니다.

## 사용 가능한 CLI 옵션

SDK는 Claude Code에서 사용할 수 있는 모든 CLI 옵션을 활용합니다. 다음은 SDK 사용을 위한 주요 옵션입니다.

- `--print`, `-p`: 비대화형 모드로 실행합니다. 예시: `claude -p "query"`
- `--output-format`: 출력 형식을 지정합니다 (`text`, `json`, `stream-json`). 예시: `claude -p --output-format json`
- `--resume`, `-r`: 세션 ID로 대화를 재개합니다. 예시: `claude --resume abc123`
- `--continue`, `-c`: 가장 최근 대화를 계속합니다. 예시: `claude --continue`
- `--verbose`: 상세 로깅을 활성화합니다. 예시: `claude --verbose`
- `--max-turns`: 비대화형 모드에서 에이전트 턴을 제한합니다. 예시: `claude --max-turns 3`
- `--system-prompt`: 시스템 프롬프트를 재정의합니다 (`--print`에서만 사용 가능). 예시: `claude --system-prompt "사용자 정의 지침"`
- `--append-system-prompt`: 시스템 프롬프트에 추가합니다 (`--print`에서만 사용 가능). 예시: `claude --append-system-prompt "사용자 정의 지침"`
- `--allowedTools`: 허용된 도구의 공백으로 구분된 목록 또는 쉼표로 구분된 문자열입니다. 예시: `claude --allowedTools mcp__slack mcp__filesystem`, `claude --allowedTools "Bash(npm install),mcp__filesystem"`
- `--disallowedTools`: 거부된 도구의 공백으로 구분된 목록 또는 쉼표로 구분된 문자열입니다. 예시: `claude --disallowedTools mcp__splunk mcp__github`, `claude --disallowedTools "Bash(git commit),mcp__github"`
- `--mcp-config`: JSON 파일에서 MCP 서버를 로드합니다. 예시: `claude --mcp-config servers.json`
- `--permission-prompt-tool`: 권한 프롬프트를 처리하기 위한 MCP 도구입니다 (`--print`에서만 사용 가능). 예시: `claude --permission-prompt-tool mcp__auth__prompt`

CLI 옵션 및 기능의 전체 목록은 CLI 사용 문서를 참조하십시오.

## 출력 형식

SDK는 여러 출력 형식을 지원합니다.

#### 텍스트 출력 (기본값)

응답 텍스트만 반환합니다.

```shell
$ claude -p "src/components/Header.tsx 파일을 설명하십시오."
# 출력: 이것은 React 컴포넌트이며...
```

#### JSON 출력

메타데이터를 포함한 구조화된 데이터를 반환합니다.

```shell
$ claude -p "데이터 계층은 어떻게 작동합니까?" --output-format json
```

응답 형식은 다음과 같습니다.

```json
{
  "type": "result",
  "subtype": "success",
  "total_cost_usd": 0.003,
  "is_error": false,
  "duration_ms": 1234,
  "duration_api_ms": 800,
  "num_turns": 6,
  "result": "여기에 응답 텍스트...",
  "session_id": "abc123"
}
```

## 스트리밍 JSON 출력

각 메시지를 수신하는 즉시 스트리밍합니다.

#### CLI 명령 예시

```bash
$ claude -p "Build an application" --output-format stream-json
```

각 대화는 초기 `init` 시스템 메시지로 시작하며, 이어서 사용자 및 어시스턴트 메시지 목록이 오고, 마지막으로 통계가 포함된 최종 `result` 시스템 메시지로 마무리됩니다. 각 메시지는 별도의 JSON 객체로 방출됩니다.

## 메시지 스키마

JSON API에서 반환되는 메시지는 다음 스키마에 따라 엄격하게 유형화됩니다.

#### SDKMessage 타입 정의

```ts
type SDKMessage =
  // An assistant message
  | {
      type: "assistant";
      message: Message; // from Anthropic SDK
      session_id: string;
    }
  // A user message
  | {
      type: "user";
      message: MessageParam; // from Anthropic SDK
      session_id: string;
    }
  // Emitted as the last message
  | {
      type: "result";
      subtype: "success";
      duration_ms: float;
      duration_api_ms: float;
      is_error: boolean;
      num_turns: int;
      result: string;
      session_id: string;
      total_cost_usd: float;
    }
  // Emitted as the last message, when we've reached the maximum number of turns
  | {
      type: "result";
      subtype: "error_max_turns" | "error_during_execution";
      duration_ms: float;
      duration_api_ms: float;
      is_error: boolean;
      num_turns: int;
      session_id: string;
      total_cost_usd: float;
    }
  // Emitted as the first message at the start of a conversation
  | {
      type: "system";
      subtype: "init";
      apiKeySource: string;
      cwd: string;
      session_id: string;
      tools: string[];
      mcp_servers: {
        name: string;
        status: string;
      }[];
      model: string;
      permissionMode: "default" | "acceptEdits" | "bypassPermissions" | "plan";
    };
```

이러한 유형은 곧 JSONSchema 호환 형식으로 게시될 예정입니다. Claude Code 기본 패키지에는 이 형식의 파괴적 변경 사항을 전달하기 위해 Semantic Versioning을 사용하고 있습니다.

#### Message 및 MessageParam 타입

`Message` 및 `MessageParam` 타입은 Anthropic SDK에서 사용할 수 있습니다. 예를 들어, Anthropic [TypeScript](https://github.com/anthropics/anthropic-sdk-typescript) 및 [Python](https://github.com/anthropics/anthropic-sdk-python/) SDK를 참조하십시오.

## 입력 형식

SDK는 여러 입력 형식을 지원합니다.

#### 텍스트 입력 (기본값)

입력 텍스트는 인수로 제공될 수 있습니다.

```bash
$ claude -p "Explain this code"
```

또는 입력 텍스트는 stdin을 통해 파이프될 수 있습니다.

```bash
$ echo "Explain this code" | claude -p
```

#### 스트리밍 JSON 입력

각 메시지가 사용자 턴을 나타내는 메시지 스트림이 `stdin`을 통해 제공됩니다. 이를 통해 `claude` 바이너리를 다시 시작하지 않고도 여러 턴의 대화를 할 수 있으며, 모델이 요청을 처리하는 동안 모델에 지침을 제공할 수 있습니다. 각 메시지는 출력 메시지 스키마와 동일한 형식의 JSON '사용자 메시지' 객체입니다. 메시지는 각 입력 라인이 완전한 JSON 객체인 [jsonl](https://jsonlines.org/) 형식으로 포맷됩니다. 스트리밍 JSON 입력에는 `-p` 및 `--output-format stream-json`이 필요합니다. 현재는 텍스트 전용 사용자 메시지로 제한됩니다.

```bash
$ echo '{"type":"user","message":{"role":"user","content":[{"type":"text","text":"Explain this code"}]}}' | claude -p --output-format=stream-json --input-format=stream-json --verbose
```

## 예시

#### 간단한 스크립트 통합

```bash
#!/bin/bash
# Simple function to run Claude and check exit code
run_claude() {
    local prompt="$1"
    local output_format="${2:-text}"
    if claude -p "$prompt" --output-format "$output_format"; then
        echo "Success!"
    else
        echo "Error: Claude failed with exit code $?" >&2
        return 1
    fi
}
# Usage examples
run_claude "Write a Python function to read CSV files"
run_claude "Optimize this database query" "json"
```

#### Claude로 파일 처리

```bash
# Process a file through Claude
$ cat mycode.py | claude -p "Review this code for bugs"
# Process multiple files
$ for file in *.js; do
    echo "Processing $file..."
    claude -p "Add JSDoc comments to this file:" < "$file" > "${file}.documented"
done
# Use Claude in a pipeline
$ grep -l "TODO" *.py | while read file; do
    claude -p "Fix all TODO items in this file" < "$file"
done
```

#### 세션 관리

```bash
# Start a session and capture the session ID
$ claude -p "Initialize a new project" --output-format json | jq -r '.session_id' > session.txt
# Continue with the same session
$ claude -p --resume "$(cat session.txt)" "Add unit tests"
```

## 모범 사례

1. **JSON 출력 형식 사용**은 응답을 프로그래밍 방식으로 파싱하는 데 유용합니다.
    ```bash
    # Parse JSON response with jq
    result=$(claude -p "Generate code" --output-format json)
    code=$(echo "$result" | jq -r '.result')
    cost=$(echo "$result" | jq -r '.cost_usd')
    ```
2. **오류를 유연하게 처리**하십시오. 종료 코드와 stderr를 확인하십시오.
    ```bash
    if ! claude -p "$prompt" 2>error.log; then
        echo "Error occurred:" >&2
        cat error.log >&2
        exit 1
    fi
    ```
3. 다중 턴 대화에서 컨텍스트 유지를 위해 **세션 관리를 사용**하십시오.
4. 장기 실행 작업에 대해 **타임아웃을 고려**하십시오.
    ```bash
    timeout 300 claude -p "$complex_prompt" || echo "Timed out after 5 minutes"
    ```
5. 여러 요청을 수행할 때 지연을 추가하여 **속도 제한을 준수**하십시오.

## 실제 응용 프로그램

Claude Code SDK는 개발 워크플로와 강력하게 통합될 수 있도록 합니다. 주목할 만한 예시 중 하나는 SDK를 사용하여 GitHub 워크플로 내에서 자동화된 코드 검토, PR 생성 및 이슈 분류 기능을 제공하는 [Claude Code GitHub Actions](https://docs.anthropic.com/en/docs/claude-code/github-actions)입니다.

## 관련 리소스

- [CLI 사용법 및 제어](https://docs.anthropic.com/en/docs/claude-code/cli-reference) - 전체 CLI 문서입니다.
- [GitHub Actions 통합](https://docs.anthropic.com/en/docs/claude-code/github-actions) - Claude로 GitHub 워크플로를 자동화하십시오.
- [일반적인 워크플로](https://docs.anthropic.com/en/docs/claude-code/common-workflows) - 일반적인 사용 사례에 대한 단계별 가이드입니다.