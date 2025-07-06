---
aliases: []
related:
title: "[번역] Claude Code Hooks: 사용자 정의 셸 명령어로 동작 제어하기"
created: 2025-07-06 06:28:52
modified: 2025-07-06 11:24:17
tags: [자동화, ClaudeCode]
source_url: https://docs.anthropic.com/en/docs/claude-code/hooks
clippings: "[[2025-07-06]]"
description: Claude Code의 동작을 사용자 정의하고 확장하려면 쉘 명령어를 등록하세요.
---

> 원문: https://docs.anthropic.com/en/docs/claude-code/hooks

> [!summary]
> Claude Code hook은 Claude Code의 생명 주기 내 특정 시점에 실행되는 사용자 정의 셸 명령어입니다. 이를 통해 알림 사용자 정의, 자동 포맷, 로깅, 피드백 제공, 사용자 정의 권한 설정 등 Claude Code의 동작을 결정적으로 제어할 수 있습니다. hook은 설정 파일에서 구성되며, 다양한 이벤트(PreToolUse, PostToolUse, Notification, Stop, SubagentStop)에 따라 실행됩니다. hook은 JSON 데이터를 입력으로 받아, 종료 코드 또는 JSON 출력을 통해 Claude Code에 피드백을 제공할 수 있습니다. 보안상의 이유로 hook 사용 시 주의가 필요하며, 입력 유효성 검사, 셸 변수 인용, 경로 탐색 방지 등 보안 모범 사례를 준수해야 합니다.

## 소개

Claude Code hook은 Claude Code의 생명 주기 내 다양한 시점에 실행되는 사용자 정의 셸 명령어입니다. hook은 Claude Code의 동작에 대한 결정적인 제어를 제공하여, LLM이 실행 여부를 선택하는 대신 특정 동작이 항상 발생하도록 보장합니다.

예시 사용 사례는 다음과 같습니다:

* **알림**: Claude Code가 사용자 입력이나 실행 권한을 기다릴 때 알림 방식을 사용자 정의합니다.
* **자동 포맷**: 모든 파일 편집 후 .ts 파일에 `prettier`, .go 파일에 `gofmt` 등을 실행합니다.
* **로깅**: 규정 준수 또는 디버깅을 위해 실행된 모든 명령어를 추적하고 집계합니다.
* **피드백**: Claude Code가 코드베이스 규칙을 따르지 않는 코드를 생성할 때 자동화된 피드백을 제공합니다.
* **사용자 정의 권한**: 운영 파일 또는 민감한 디렉터리에 대한 수정을 차단합니다.

이러한 규칙을 프롬프트 지시 대신 hook으로 인코딩함으로써, 제안을 예상될 때마다 실행되는 앱 수준 코드로 바꿀 수 있습니다.

hook은 사용자 전체 권한으로 확인 없이 셸 명령어를 실행합니다. hook이 안전하고 보안에 철저한지 확인할 책임은 사용자에게 있습니다. Anthropic은 hook 사용으로 인한 데이터 손실이나 시스템 손상에 대해 어떠한 책임도 지지 않습니다. [[Security Considerations]]를 검토하십시오.

## 빠른 시작

이 빠른 시작에서는 Claude Code가 실행하는 셸 명령어를 로깅하는 hook을 추가할 것입니다.

빠른 시작 전제 조건: 명령줄에서 JSON 처리를 위해 `jq`를 설치하십시오.

### 1단계: Hook 구성 열기

`/hooks` [[슬래시 명령어]]를 실행하고 `PreToolUse` hook 이벤트를 선택하십시오.

`PreToolUse` hook은 도구 호출 전에 실행되며, Claude에게 다르게 수행할 작업에 대한 피드백을 제공하면서 도구 호출을 차단할 수 있습니다.

### 2단계: 매처 추가

Bash 도구 호출에서만 hook을 실행하려면 `+ Add new matcher…`를 선택하십시오.

매처로 `Bash`를 입력하십시오.

### 3단계: Hook 추가

`+ Add new hook…`를 선택하고 다음 명령어를 입력하십시오:

```shell
jq -r '"\(.tool_input.command) - \(.tool_input.description // "No description")"' >> ~/.claude/bash-command-log.txt
```

### 4단계: 구성 저장

저장 위치는 홈 디렉터리에 로깅하므로 `User settings`를 선택하십시오. 이 hook은 현재 프로젝트뿐만 아니라 모든 프로젝트에 적용될 것입니다.

그런 다음 REPL로 돌아올 때까지 Esc를 누르십시오. 이제 hook이 등록되었습니다!

### 5단계: Hook 확인

구성을 확인하려면 `/hooks`를 다시 실행하거나 `~/.claude/settings.json`을 확인하십시오:

```json
"hooks": {
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "jq -r '\"\\(.tool_input.command) - \\(.tool_input.description // \"No description\")\"' >> ~/.claude/bash-command-log.txt"
        }
      ]
    }
  ]
}
```

## 구성

Claude Code hook은 다음 [[설정 파일]]에서 구성됩니다:

* `~/.claude/settings.json` - 사용자 설정
* `.claude/settings.json` - 프로젝트 설정
* `.claude/settings.local.json` - 로컬 프로젝트 설정 (커밋되지 않음)
* 엔터프라이즈 관리 정책 설정

### 구조

Hook은 매처별로 구성되며, 각 매처는 여러 hook을 가질 수 있습니다:

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

* **matcher**: 도구 이름을 일치시키기 위한 패턴 (`PreToolUse` 및 `PostToolUse`에만 적용 가능)
    * 단순 문자열은 정확히 일치합니다: `Write`는 Write 도구만 일치시킵니다.
    * 정규 표현식을 지원합니다: `Edit|Write` 또는 `Notebook.*`
    * 생략되거나 빈 문자열인 경우, hook은 모든 일치하는 이벤트에 대해 실행됩니다.
* **hooks**: 패턴이 일치할 때 실행할 명령어 배열
    * `type`: 현재 `"command"`만 지원됩니다.
    * `command`: 실행할 셸 명령어
    * `timeout`: (선택 사항) 진행 중인 모든 hook을 취소하기 전에 명령어가 실행되어야 하는 시간(초 단위).

## Hook 이벤트

### PreToolUse

Claude가 도구 매개변수를 생성한 후, 도구 호출을 처리하기 전에 실행됩니다.

**일반적인 매처:**

* `Task` - 에이전트 작업
* `Bash` - 셸 명령어
* `Glob` - 파일 패턴 일치
* `Grep` - 내용 검색
* `Read` - 파일 읽기
* `Edit`, `MultiEdit` - 파일 편집
* `Write` - 파일 쓰기
* `WebFetch`, `WebSearch` - 웹 작업

### PostToolUse

도구가 성공적으로 완료된 직후 실행됩니다.

PreToolUse와 동일한 매처 값을 인식합니다.

### Notification

Claude Code가 알림을 보낼 때 실행됩니다.

### Stop

주요 Claude Code 에이전트가 응답을 마쳤을 때 실행됩니다.

### SubagentStop

Claude Code 서브 에이전트(Task 도구 호출)가 응답을 마쳤을 때 실행됩니다.

## Hook 입력

Hook은 세션 정보 및 이벤트별 데이터를 포함하는 JSON 데이터를 stdin을 통해 받습니다:

```json
{
  // Common fields
  session_id: string
  transcript_path: string  // Path to conversation JSON

  // Event-specific fields
  ...
}
```

### PreToolUse 입력

`tool_input`의 정확한 스키마는 도구에 따라 달라집니다.

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.txt",
    "content": "file content"
  }
}
```

### PostToolUse 입력

`tool_input` 및 `tool_response`의 정확한 스키마는 도구에 따라 달라집니다.

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.txt",
    "content": "file content"
  },
  "tool_response": {
    "filePath": "/path/to/file.txt",
    "success": true
  }
}
```

### Notification 입력

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "message": "Task completed successfully",
  "title": "Claude Code"
}
```

### Stop 및 SubagentStop 입력

`stop_hook_active`는 Claude Code가 이미 정지 hook의 결과로 계속 진행 중일 때 참입니다. 이 값을 확인하거나 트랜스크립트를 처리하여 Claude Code가 무한정 실행되는 것을 방지하십시오.

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../00893aaf-19fa-41d2-8238-13269b9b3ca0.jsonl",
  "stop_hook_active": true
}
```

## Hook 출력

Hook이 Claude Code로 출력을 반환하는 방법은 두 가지입니다. 출력은 차단 여부와 Claude 및 사용자에게 표시되어야 할 피드백을 전달합니다.

### 단순: 종료 코드

Hook은 종료 코드, stdout, stderr를 통해 상태를 전달합니다:

* **종료 코드 0**: 성공. `stdout`은 트랜스크립트 모드(CTRL-R)에서 사용자에게 표시됩니다.
* **종료 코드 2**: 차단 오류. `stderr`은 Claude에게 자동으로 처리되도록 피드백됩니다. 아래 hook 이벤트별 동작을 참조하십시오.
* **다른 종료 코드**: 비차단 오류. `stderr`은 사용자에게 표시되고 실행은 계속됩니다.

> [!warning]
> 참고: 종료 코드가 0인 경우 Claude Code는 stdout을 보지 못합니다.

#### 종료 코드 2 동작

| Hook 이벤트       | 동작 |
| :-------- | :--- |
| `PreToolUse`   | 도구 호출을 차단하고 Claude에게 오류를 표시합니다. |
| `PostToolUse`  | Claude에게 오류를 표시합니다 (도구는 이미 실행됨). |
| `Notification` | 해당 없음, 사용자에게만 stderr를 표시합니다. |
| `Stop`         | 중지를 차단하고 Claude에게 오류를 표시합니다. |
| `SubagentStop` | 중지를 차단하고 Claude 서브 에이전트에게 오류를 표시합니다. |

### 고급: JSON 출력

Hook은 더 정교한 제어를 위해 `stdout`에 구조화된 JSON을 반환할 수 있습니다:

#### 일반 JSON 필드

모든 hook 유형은 다음 선택적 필드를 포함할 수 있습니다:

```json
{
  "continue": true, // hook 실행 후 Claude가 계속 진행해야 하는지 여부 (기본값: true)
  "stopReason": "string" // continue가 false일 때 표시되는 메시지
  "suppressOutput": true, // 트랜스크립트 모드에서 stdout 숨기기 (기본값: false)
}
```

`continue`가 false인 경우, hook 실행 후 Claude는 처리를 중지합니다.

* `PreToolUse`의 경우, 이는 특정 도구 호출만 차단하고 Claude에게 자동 피드백을 제공하는 `"decision": "block"`과는 다릅니다.
* `PostToolUse`의 경우, 이는 Claude에게 자동화된 피드백을 제공하는 `"decision": "block"`과는 다릅니다.
* `Stop` 및 `SubagentStop`의 경우, 이는 어떤 `"decision": "block"` 출력보다 우선합니다.
* 모든 경우에, `"continue" = false`는 어떤 `"decision": "block"` 출력보다 우선합니다.

`stopReason`은 사용자에게 표시되는 이유와 함께 `continue`를 동반하며, Claude에게는 표시되지 않습니다.

#### `PreToolUse` 결정 제어

`PreToolUse` hook은 도구 호출이 진행될지 여부를 제어할 수 있습니다.

* “approve”는 권한 시스템을 우회합니다. `reason`은 사용자에게 표시되지만 Claude에게는 표시되지 않습니다.
* “block”은 도구 호출이 실행되는 것을 방지합니다. `reason`은 Claude에게 표시됩니다.
* `undefined`는 기존 권한 흐름으로 이어집니다. `reason`은 무시됩니다.

```json
{
  "decision": "approve" | "block" | undefined,
  "reason": "Explanation for decision"
}
```

#### `PostToolUse` 결정 제어

`PostToolUse` hook은 도구 호출이 진행될지 여부를 제어할 수 있습니다.

* “block”은 `reason`으로 Claude에게 자동으로 프롬프트합니다.
* `undefined`는 아무것도 하지 않습니다. `reason`은 무시됩니다.

```json
{
  "decision": "block" | undefined,
  "reason": "Explanation for decision"
}
```

#### `Stop`/`SubagentStop` 결정 제어

`Stop` 및 `SubagentStop` hook은 Claude가 계속 진행해야 하는지 여부를 제어할 수 있습니다.

* “block”은 Claude가 중지하는 것을 방지합니다. Claude가 어떻게 진행해야 할지 알도록 `reason`을 채워야 합니다.
* `undefined`는 Claude가 중지하는 것을 허용합니다. `reason`은 무시됩니다.

```json
{
  "decision": "block" | undefined,
  "reason": "Must be provided when Claude is blocked from stopping"
}
```

#### JSON 출력 예시: Bash 명령어 편집

```python
#!/usr/bin/env python3
import json
import re
import sys

# Define validation rules as a list of (regex pattern, message) tuples
VALIDATION_RULES = [
    (
        r"\bgrep\b(?!.*\|)",
        "Use 'rg' (ripgrep) instead of 'grep' for better performance and features",
    ),
    (
        r"\bfind\s+\S+\s+-name\b",
        "Use 'rg --files | rg pattern' or 'rg --files -g pattern' instead of 'find -name' for better performance",
    ),
]


def validate_command(command: str) -> list[str]:
    issues = []
    for pattern, message in VALIDATION_RULES:
        if re.search(pattern, command):
            issues.append(message)
    return issues


try:
    input_data = json.load(sys.stdin)
except json.JSONDecodeError as e:
    print(f"Error: Invalid JSON input: {e}", file=sys.stderr)
    sys.exit(1)

tool_name = input_data.get("tool_name", "")
tool_input = input_data.get("tool_input", {})
command = tool_input.get("command", "")

if tool_name != "Bash" or not command:
    sys.exit(1)

# Validate the command
issues = validate_command(command)

if issues:
    for message in issues:
        print(f"• {message}", file=sys.stderr)
    # Exit code 2 blocks tool call and shows stderr to Claude
    sys.exit(2)
```

## MCP 도구 작업

Claude Code hook은 [[Model Context Protocol (MCP) 도구]]와 원활하게 작동합니다. MCP 서버가 도구를 제공할 때, hook에서 일치시킬 수 있는 특별한 명명 패턴으로 나타납니다.

### MCP 도구 명명

MCP 도구는 `mcp__<server>__<tool>` 패턴을 따릅니다. 예시:

* `mcp__memory__create_entities` - Memory 서버의 엔티티 생성 도구
* `mcp__filesystem__read_file` - Filesystem 서버의 파일 읽기 도구
* `mcp__github__search_repositories` - GitHub 서버의 검색 도구

### MCP 도구용 hook 구성

특정 MCP 도구 또는 전체 MCP 서버를 대상으로 지정할 수 있습니다:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Memory operation initiated' >> ~/mcp-operations.log"
          }
        ]
      },
      {
        "matcher": "mcp__.*__write.*",
        "hooks": [
          {
            "type": "command",
            "command": "/home/user/scripts/validate-mcp-write.py"
          }
        ]
      }
    ]
  }
}
```

## 예시

### 코드 포맷팅

파일 수정 후 코드 자동 포맷:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "/home/user/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

### 알림

Claude Code가 권한을 요청하거나 프롬프트 입력이 유휴 상태가 되었을 때 전송되는 알림을 사용자 정의합니다.

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "python3 ~/my_custom_notifier.py"
          }
        ]
      }
    ]
  }
}
```

## 보안 고려 사항

R

**사용자 책임 하에 사용**: Claude Code hook은 시스템에서 임의의 셸 명령어를 자동으로 실행합니다. hook을 사용함으로써 다음 사항을 인정합니다:

* 구성하는 명령어에 대한 책임은 전적으로 사용자에게 있습니다.
* Hook은 사용자 계정이 접근할 수 있는 모든 파일을 수정, 삭제 또는 접근할 수 있습니다.
* 악의적이거나 잘못 작성된 hook은 데이터 손실 또는 시스템 손상을 야기할 수 있습니다.
* Anthropic은 어떠한 보증도 제공하지 않으며 hook 사용으로 인한 어떠한 손해에 대해서도 책임을 지지 않습니다.
* 운영 환경에서 사용하기 전에 안전한 환경에서 hook을 철저히 테스트해야 합니다.

구성 파일에 추가하기 전에 모든 hook 명령어를 항상 검토하고 이해하십시오.

### 보안 모범 사례

다음은 더 안전한 hook을 작성하기 위한 몇 가지 핵심 모범 사례입니다:

1. **입력 유효성 검사 및 정제** - 입력 데이터를 맹목적으로 신뢰하지 마십시오.
2. **항상 셸 변수 인용** - `$VAR` 대신 `"$VAR"`를 사용하십시오.
3. **경로 탐색 차단** - 파일 경로에서 `..`를 확인하십시오.
4. **절대 경로 사용** - 스크립트에 대해 전체 경로를 지정하십시오.
5. **민감한 파일 건너뛰기** - `.env`, `.git/`, 키 등을 피하십시오.

### 구성 안전성

설정 파일에서 hook을 직접 편집해도 즉시 적용되지 않습니다. Claude Code는 다음을 수행합니다:

1. 시작 시 hook의 스냅샷을 캡처합니다.
2. 세션 내내 이 스냅샷을 사용합니다.
3. hook이 외부에서 수정된 경우 경고합니다.
4. 변경 사항이 적용되려면 `/hooks` 메뉴에서 검토가 필요합니다.

이는 악의적인 hook 수정이 현재 세션에 영향을 미치는 것을 방지합니다.

## Hook 실행 세부 정보

* **타임아웃**: 기본적으로 60초 실행 제한, 명령어별 구성 가능.
    * 개별 명령어가 타임아웃되면 모든 진행 중인 hook이 취소됩니다.
* **병렬 처리**: 모든 일치하는 hook은 병렬로 실행됩니다.
* **환경**: Claude Code의 환경과 함께 현재 디렉터리에서 실행됩니다.
* **입력**: stdin을 통한 JSON
* **출력**:
    * PreToolUse/PostToolUse/Stop: 트랜스크립트(Ctrl-R)에 진행 상황 표시
    * Notification: 디버그 전용으로 로깅됨 (`--debug`)

## 디버깅

hook 문제 해결:

1. `/hooks` 메뉴에 구성이 표시되는지 확인하십시오.
2. [[설정 파일]]이 유효한 JSON인지 확인하십시오.
3. 명령어를 수동으로 테스트하십시오.
4. 종료 코드를 확인하십시오.
5. stdout 및 stderr 형식 기대치를 검토하십시오.
6. 올바른 인용 문자 이스케이프를 확인하십시오.
7. hook을 디버깅하려면 `claude --debug`를 사용하십시오. 성공적인 hook의 출력은 아래와 같이 나타납니다.

```
[DEBUG] Executing hooks for PostToolUse:Write
[DEBUG] Getting matching hook commands for PostToolUse with query: Write
[DEBUG] Found 1 hook matchers in settings
[DEBUG] Matched 1 hooks for query "Write"
[DEBUG] Found 1 hook commands to execute
[DEBUG] Executing hook command: <Your command> with timeout 60000ms
[DEBUG] Hook command completed with status 0: <Your stdout>
```

진행 메시지는 트랜스크립트 모드(Ctrl-R)에 다음을 보여주면서 나타납니다:

* 어떤 hook이 실행 중인지
* 실행 중인 명령어
* 성공/실패 상태
* 출력 또는 오류 메시지

이 페이지가 도움이 되셨나요?