---
title: "[번역] Claude Desktop 확장 기능: Claude Desktop용 원클릭 MCP 서버 설치"
related:
aliases: 번역_Claude Desktop용 원클릭 MCP 서버 설치
created: 2025-06-30 03:18:27
modified: 2025-06-30 03:25:41
---

> 원문: https://www.anthropic.com/engineering/desktop-extensions

저희가 작년에 모델 컨텍스트 프로토콜(MCP)을 출시했을 때, 개발자들이 Claude가 파일 시스템부터 데이터베이스까지 모든 것에 접근할 수 있도록 해주는 놀라운 로컬 서버를 구축하는 것을 보았습니다. 하지만 저희는 설치가 너무 복잡하다는 동일한 피드백을 계속 들었습니다. 사용자들은 개발자 도구를 필요로 했고, 구성 파일을 수동으로 편집해야 했으며, 종종 종속성 문제에 막혔습니다.

오늘, 저희는 MCP 서버 설치를 버튼 클릭만큼 간단하게 만들어주는 새로운 패키징 형식인 데스크톱 확장 기능을 소개합니다.

### MCP 설치 문제 해결

로컬 MCP 서버는 Claude Desktop 사용자에게 강력한 기능을 제공합니다. 사용자들은 로컬 애플리케이션과 상호 작용하고, 비공개 데이터에 접근하며, 개발 도구와 통합할 수 있습니다. 이 모든 과정에서 데이터는 사용자 기기에 유지됩니다. 그러나 현재의 설치 과정은 상당한 장벽을 만듭니다.

- **개발자 도구 필요**: 사용자들은 Node.js, Python 또는 기타 런타임을 설치해야 합니다.
- **수동 구성**: 각 서버는 JSON 구성 파일을 편집해야 합니다.
- **종속성 관리**: 사용자들은 패키지 충돌 및 버전 불일치를 해결해야 합니다.
- **검색 메커니즘 부재**: 유용한 MCP 서버를 찾으려면 GitHub를 검색해야 합니다.
- **업데이트 복잡성**: 서버를 최신 상태로 유지하려면 수동 재설치가 필요합니다.

이러한 마찰 지점들로 인해 MCP 서버는 강력함에도 불구하고 비기술 사용자에게는 대부분 접근하기 어려웠습니다.

### 데스크톱 확장 기능 소개

데스크톱 확장 기능(.dxt 파일)은 전체 MCP 서버(모든 종속성 포함)를 단일 설치 가능 패키지로 묶어 이러한 문제를 해결합니다. 사용자에게 달라지는 점은 다음과 같습니다.

**이전:**

```sh
# Node.js를 먼저 설치합니다.
npm install -g @example/mcp-server
# ~/.claude/claude_desktop_config.json을 수동으로 편집합니다.
# Claude Desktop을 다시 시작합니다.
# 작동하기를 바랍니다.
```

**이후:**

1. .dxt 파일을 다운로드합니다.
2. Claude Desktop으로 열기 위해 더블 클릭합니다.
3. "설치"를 클릭합니다.

이것이 전부입니다. 터미널도, 구성 파일도, 종속성 충돌도 없습니다.

## 아키텍처 개요

데스크톱 확장 기능은 로컬 MCP 서버와 Claude Desktop 및 데스크톱 확장 기능을 지원하는 다른 앱이 알아야 할 모든 것을 설명하는 `manifest.json`을 포함하는 ZIP 아카이브입니다.

```
extension.dxt (ZIP 아카이브)
├── manifest.json         # 확장 기능 메타데이터 및 구성
├── server/               # MCP 서버 구현
│   └── [서버 파일]
├── dependencies/         # 모든 필수 패키지/라이브러리
└── icon.png             # 선택 사항: 확장 기능 아이콘

# 예시: Node.js 확장 기능
extension.dxt
├── manifest.json         # 필수: 확장 기능 메타데이터 및 구성
├── server/               # 서버 파일
│   └── index.js          # 메인 진입점
├── node_modules/         # 번들된 종속성
├── package.json          # 선택 사항: NPM 패키지 정의
└── icon.png              # 선택 사항: 확장 기능 아이콘

# 예시: Python 확장 기능
extension.dxt (ZIP 파일)
├── manifest.json         # 필수: 확장 기능 메타데이터 및 구성
├── server/               # 서버 파일
│   ├── main.py           # 메인 진입점
│   └── utils.py          # 추가 모듈
├── lib/                  # 번들된 Python 패키지
├── requirements.txt      # 선택 사항: Python 종속성 목록
└── icon.png              # 선택 사항: 확장 기능 아이콘
```

데스크톱 확장 기능에서 유일하게 필수적인 파일은 manifest.json입니다. Claude Desktop은 모든 복잡성을 처리합니다.

- **내장 런타임**: Claude Desktop에 Node.js를 함께 제공하여 외부 종속성을 제거합니다.
- **자동 업데이트**: 새 버전이 사용 가능할 때 확장 기능이 자동으로 업데이트됩니다.
- **보안 비밀**: API 키와 같은 민감한 구성은 OS 키체인에 저장됩니다.

매니페스트에는 사람이 읽을 수 있는 정보(이름, 설명, 작성자 등), 기능 선언(도구, 프롬프트), 사용자 구성 및 런타임 요구 사항이 포함됩니다. 대부분의 필드는 선택 사항이므로 최소 버전은 매우 짧지만, 실제로는 지원되는 세 가지 확장 기능 유형(Node.js, Python, 기존 바이너리/실행 파일) 모두 파일들을 포함할 것으로 예상됩니다.

```json
{
  "dxt_version": "0.1",                     // 이 매니페스트가 따르는 DXT 사양 버전
  "name": "my-extension",                   // 기계가 읽을 수 있는 이름 (CLI, API에 사용)
  "version": "1.0.0",                       // 확장 기능의 시맨틱 버전
  "description": "A simple MCP extension",  // 확장 기능이 수행하는 작업에 대한 간략한 설명
  "author": {                               // 작성자 정보 (필수)
    "name": "Extension Author"              // 작성자 이름 (필수 필드)
  },
  "server": {                               // 서버 구성 (필수)
    "type": "node",                         // 서버 유형: "node", "python", 또는 "binary"
    "entry_point": "server/index.js",       // 메인 서버 파일 경로
    "mcp_config": {                         // MCP 서버 구성
      "command": "node",                    // 서버를 실행할 명령
      "args": [                             // 명령에 전달되는 인수
        "${__dirname}/server/index.js"      // ${__dirname}은 확장 기능의 디렉터리로 대체됩니다.
      ]
    }
  }
}
```

매니페스트 사양에는 로컬 MCP 서버의 설치 및 구성을 더 쉽게 만드는 것을 목표로 하는 여러 편리한 옵션이 있습니다. 서버 구성 객체는 사용자 정의 구성(템플릿 리터럴 형태)과 플랫폼별 재정의를 모두 수용할 수 있도록 정의될 수 있습니다. 확장 기능 개발자는 사용자로부터 어떤 종류의 구성을 수집할지 상세하게 정의할 수 있습니다.

매니페스트가 구성에 어떻게 도움이 되는지 구체적인 예를 살펴보겠습니다. 아래 매니페스트에서 개발자는 사용자가 `api_key`를 제공해야 한다고 선언합니다. Claude는 사용자가 해당 값을 제공할 때까지 확장 기능을 활성화하지 않으며, 이를 운영체제의 비밀 저장소에 자동으로 안전하게 보관하고, 서버를 시작할 때 `${user_config.api_key}`를 사용자가 제공한 값으로 투명하게 대체합니다. 마찬가지로, `${__dirname}`은 확장 기능의 압축 해제된 디렉터리 전체 경로로 대체됩니다.

```json
{
  "dxt_version": "0.1",
  "name": "my-extension",
  "version": "1.0.0",
  "description": "A simple MCP extension",
  "author": {
    "name": "Extension Author"
  },
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"],
      "env": {
        "API_KEY": "${user_config.api_key}"
      }
    }
  },
  "user_config": {
    "api_key": {
      "type": "string",
      "title": "API Key",
      "description": "Your API key for authentication",
      "sensitive": true,
      "required": true
    }
  }
}
```

대부분의 선택적 필드를 포함한 전체 `manifest.json`은 다음과 같을 수 있습니다.

```json
{
  "dxt_version": "0.1",
  "name": "My MCP Extension",
  "display_name": "My Awesome MCP Extension",
  "version": "1.0.0",
  "description": "A brief description of what this extension does",
  "long_description": "A detailed description that can include multiple paragraphs explaining the extension's functionality, use cases, and features. It supports basic markdown.",
  "author": {
    "name": "Your Name",
    "email": "yourname@example.com",
    "url": "https://your-website.com"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/your-username/my-mcp-extension"
  },
  "homepage": "https://example.com/my-extension",
  "documentation": "https://docs.example.com/my-extension",
  "support": "https://github.com/your-username/my-extension/issues",
  "icon": "icon.png",
  "screenshots": [
    "assets/screenshots/screenshot1.png",
    "assets/screenshots/screenshot2.png"
  ],
  "server": {
    "type": "node",
    "entry_point": "server/index.js",
    "mcp_config": {
      "command": "node",
      "args": ["${__dirname}/server/index.js"],
      "env": {
        "ALLOWED_DIRECTORIES": "${user_config.allowed_directories}"
      }
    }
  },
  "tools": [
    {
      "name": "search_files",
      "description": "Search for files in a directory"
    }
  ],
  "prompts": [
    {
      "name": "poetry",
      "description": "Have the LLM write poetry",
      "arguments": ["topic"],
      "text": "Write a creative poem about the following topic: ${arguments.topic}"
    }
  ],
  "tools_generated": true,
  "keywords": ["api", "automation", "productivity"],
  "license": "MIT",
  "compatibility": {
    "claude_desktop": ">=1.0.0",
    "platforms": ["darwin", "win32", "linux"],
    "runtimes": {
      "node": ">=16.0.0"
    }
  },
  "user_config": {
    "allowed_directories": {
      "type": "directory",
      "title": "Allowed Directories",
      "description": "Directories the server can access",
      "multiple": true,
      "required": true,
      "default": ["${HOME}/Desktop"]
    },
    "api_key": {
      "type": "string",
      "title": "API Key",
      "description": "Your API key for authentication",
      "sensitive": true,
      "required": false
    },
    "max_file_size": {
      "type": "number",
      "title": "Maximum File Size (MB)",
      "description": "Maximum file size to process",
      "default": 10,
      "min": 1,
      "max": 100
    }
  }
}
```

확장 기능과 매니페스트를 보려면 [dxt 저장소의 예시](https://github.com/anthropics/dxt/tree/main/examples)를 참조하십시오.

`manifest.json`의 모든 필수 및 선택적 필드에 대한 전체 사양은 [오픈 소스 툴체인](https://github.com/anthropics/dxt/blob/main/MANIFEST.md)의 일부로 찾을 수 있습니다.

### 첫 확장 기능 구축하기

기존 MCP 서버를 데스크톱 확장 기능으로 패키징하는 과정을 살펴보겠습니다. 간단한 파일 시스템 서버를 예시로 사용하겠습니다.

#### 1단계: 매니페스트 생성

먼저, 서버용 매니페스트를 초기화합니다.

```sh
npx @anthropic-ai/dxt init
```

이 대화형 도구는 서버에 대해 질문하고 완전한 manifest.json을 생성합니다. 가장 기본적인 manifest.json을 빠르게 생성하려면 `--yes` 매개변수와 함께 명령을 실행할 수 있습니다.

#### 2단계: 사용자 구성 처리

서버에 사용자 입력(예: API 키 또는 허용된 디렉터리)이 필요한 경우, 매니페스트에 선언합니다.

```json
"user_config": {
  "allowed_directories": {
    "type": "directory",
    "title": "Allowed Directories",
    "description": "Directories the server can access",
    "multiple": true,
    "required": true,
    "default": ["${HOME}/Documents"]
  }
}
```

Claude Desktop은 다음을 수행합니다.

- 사용자 친화적인 구성 UI를 표시합니다.
- 확장 기능을 활성화하기 전에 입력을 검증합니다.
- 민감한 값을 안전하게 저장합니다.
- 개발자 구성에 따라 인수로 또는 환경 변수로 구성을 서버에 전달합니다.

아래 예시에서는 사용자 구성을 환경 변수로 전달하지만, 인수로도 전달할 수 있습니다.

```json
"server": {
   "type": "node",
   "entry_point": "server/index.js",
   "mcp_config": {
   "command": "node",
   "args": ["${__dirname}/server/index.js"],
   "env": {
      "ALLOWED_DIRECTORIES": "${user_config.allowed_directories}"
   }
   }
}
```

#### 3단계: 확장 기능 패키징

모든 것을 `.dxt` 파일로 묶습니다.

```sh
npx @anthropic-ai/dxt pack
```

이 명령은 다음을 수행합니다.

1. 매니페스트를 검증합니다.
2. `.dxt` 아카이브를 생성합니다.

#### 4단계: 로컬 테스트

`.dxt` 파일을 Claude Desktop의 설정 창으로 드래그합니다. 다음을 볼 수 있습니다.

- 확장 기능에 대한 사람이 읽을 수 있는 정보
- 필수 권한 및 구성
- 간단한 "설치" 버튼

### 고급 기능

#### 교차 플랫폼 지원

확장 기능은 다양한 운영 체제에 적응할 수 있습니다.

```json
"server": {
  "type": "node",
  "entry_point": "server/index.js",
  "mcp_config": {
    "command": "node",
    "args": ["${__dirname}/server/index.js"],
    "platforms": {
      "win32": {
        "command": "node.exe",
        "env": {
          "TEMP_DIR": "${TEMP}"
        }
      },
      "darwin": {
        "env": {
          "TEMP_DIR": "${TMPDIR}"
        }
      }
    }
  }
}
```

#### 동적 구성

런타임 값에 템플릿 리터럴을 사용합니다.

- `${__dirname}`: 확장 기능의 설치 디렉터리
- `${user_config.key}`: 사용자가 제공한 구성
- `${HOME}, ${TEMP}`: 시스템 환경 변수

#### 기능 선언

사용자가 기능을 미리 이해하도록 돕습니다.

```json
"tools": [
  {
    "name": "read_file",
    "description": "Read contents of a file"
  }
],
"prompts": [
  {
    "name": "code_review",
    "description": "Review code for best practices",
    "arguments": ["file_path"]
  }
]
```

### 확장 기능 디렉터리

저희는 Claude Desktop에 내장된 엄선된 확장 기능 디렉터리와 함께 출시합니다. 사용자들은 GitHub를 검색하거나 코드를 검증할 필요 없이 한 번의 클릭으로 찾아보고, 검색하고, 설치할 수 있습니다.

데스크톱 확장 기능 사양과 macOS 및 Windows용 Claude의 구현이 시간이 지남에 따라 발전할 것으로 예상하지만, 확장 기능이 Claude의 기능을 창의적인 방식으로 확장하는 다양한 방법을 기대하고 있습니다.

확장 기능을 제출하려면:

1. 제출 양식에 있는 지침을 따르는지 확인합니다.
2. Windows 및 macOS에서 테스트합니다.
3. [확장 기능을 제출합니다](https://docs.google.com/forms/d/14_Dmcig4z8NeRMB_e7TOyrKzuZ88-BLYdLvS6LPhiZU/edit).
4. 저희 팀이 품질 및 보안을 검토합니다.

### 개방형 생태계 구축

저희는 MCP 서버를 둘러싼 개방형 생태계에 전념하고 있으며, 여러 애플리케이션과 서비스에 보편적으로 채택될 수 있는 능력이 커뮤니티에 도움이 되었다고 믿습니다. 이러한 약속에 따라, 저희는 데스크톱 확장 기능 사양, 툴체인, 그리고 macOS 및 Windows용 Claude가 데스크톱 확장 기능에 대한 자체 지원을 구현하는 데 사용되는 스키마 및 핵심 기능을 오픈 소스로 공개합니다. 저희는 dxt 형식이 Claude뿐만 아니라 다른 AI 데스크톱 애플리케이션에서도 로컬 MCP 서버를 더 이식성 있게 만들기를 바랍니다.

저희는 다음을 오픈 소스로 공개합니다.

- 완전한 DXT 사양
- 패키징 및 검증 도구
- 참조 구현 코드
- TypeScript 유형 및 스키마

이는 다음을 의미합니다.

- **MCP 서버 개발자에게**: 한 번 패키징하면 DXT를 지원하는 모든 곳에서 실행할 수 있습니다.
- **앱 개발자에게**: 처음부터 구축할 필요 없이 확장 기능 지원을 추가할 수 있습니다.
- **사용자에게**: 모든 MCP 지원 애플리케이션에서 일관된 경험을 제공합니다.

사양과 툴체인은 의도적으로 0.1 버전으로 지정되었으며, 저희는 더 큰 커뮤니티와 협력하여 형식을 발전시키고 변경하기를 기대합니다. 여러분의 의견을 기다리겠습니다.

### 보안 및 기업 고려 사항

저희는 확장 기능이 특히 기업에게 새로운 보안 고려 사항을 도입한다는 것을 이해합니다. 데스크톱 확장 기능의 미리 보기 릴리스에는 여러 안전 장치를 내장했습니다.

#### 사용자용

- 민감한 데이터는 OS 키체인에 유지됩니다.
- 자동 업데이트
- 설치된 확장 기능을 감사할 수 있는 기능

#### 기업용

- 그룹 정책(Windows) 및 MDM(macOS) 지원
- 승인된 확장 기능을 사전 설치할 수 있는 기능
- 특정 확장 기능 또는 게시자를 차단 목록에 추가
- 확장 기능 디렉터리 전체를 비활성화
- 프라이빗 확장 기능 디렉터리 배포

조직 내에서 확장 기능을 관리하는 방법에 대한 자세한 내용은 [문서](https://support.anthropic.com/en/articles/10949351-getting-started-with-model-context-protocol-mcp-on-claude-for-desktop)를 참조하십시오.

### 시작하기

자신만의 확장 기능을 구축할 준비가 되셨습니까? 시작하는 방법은 다음과 같습니다.

**MCP 서버 개발자용**: [개발자 문서](https://github.com/anthropics/dxt)를 검토하거나, 로컬 MCP 서버 디렉터리에서 다음 명령을 실행하여 바로 시작하십시오.

```sh
npm install -g @anthropic-ai/dxt
dxt init
dxt pack
```

**Claude Desktop 사용자용**: 최신 버전으로 업데이트하고 설정에서 확장 기능 섹션을 찾으십시오.

**기업용**: 배포 옵션에 대한 기업 문서를 검토하십시오.

### Claude Code로 구축하기

Anthropic 내부에서는 Claude가 최소한의 개입으로 확장 기능을 구축하는 데 탁월하다는 것을 발견했습니다. Claude Code를 사용하고 싶다면, 확장 기능이 무엇을 하기를 원하는지 간략하게 설명한 다음 프롬프트에 다음 컨텍스트를 추가하는 것을 권장합니다.

```
저는 이것을 "DXT"로 약칭되는 데스크톱 확장 기능으로 구축하고 싶습니다. 다음 단계를 따르십시오.

1. **사양을 철저히 읽으십시오:**
   - https://github.com/anthropics/dxt/blob/main/README.md - DXT 아키텍처 개요, 기능 및 통합 패턴
   - https://github.com/anthropics/dxt/blob/main/MANIFEST.md - 완전한 확장 기능 매니페스트 구조 및 필드 정의
   - https://github.com/anthropics/dxt/tree/main/examples - "Hello World" 예시를 포함한 참조 구현

2. **적절한 확장 기능 구조를 만드십시오:**
   - MANIFEST.md 사양에 따라 유효한 manifest.json을 생성하십시오.
   - 적절한 도구 정의와 함께 @modelcontextprotocol/sdk를 사용하여 MCP 서버를 구현하십시오.
   - 적절한 오류 처리 및 타임아웃 관리를 포함하십시오.

3. **최고의 개발 관행을 따르십시오:**
   - stdio 전송을 통해 적절한 MCP 프로토콜 통신을 구현하십시오.
   - 명확한 스키마, 유효성 검사 및 일관된 JSON 응답으로 도구를 구성하십시오.
   - 이 확장 기능이 로컬에서 실행될 것이라는 사실을 활용하십시오.
   - 적절한 로깅 및 디버깅 기능을 추가하십시오.
   - 적절한 문서 및 설정 지침을 포함하십시오.

4. **테스트 고려 사항:**
   - 모든 도구 호출이 적절하게 구조화된 응답을 반환하는지 확인하십시오.
   - 매니페스트가 올바르게 로드되고 호스트 통합이 작동하는지 확인하십시오.

즉시 테스트할 수 있는 완전하고 프로덕션 준비가 된 코드를 생성하십시오. 방어적 프로그래밍, 명확한 오류 메시지, 그리고 생태계와의 호환성을 보장하기 위한 정확한 DXT 사양 준수에 중점을 두십시오.
```

### 결론

데스크톱 확장 기능은 사용자가 로컬 AI 도구와 상호 작용하는 방식에 근본적인 변화를 나타냅니다. 설치 마찰을 제거함으로써, 저희는 강력한 MCP 서버를 개발자뿐만 아니라 모든 사람이 접근할 수 있도록 만들고 있습니다.

내부적으로 Anthropic에서는 데스크톱 확장 기능을 사용하여 매우 실험적인 MCP 서버(재미있는 것도 있고 유용한 것도 있습니다)를 공유하고 있습니다. 한 팀은 저희 모델이 GameBoy에 직접 연결되었을 때 얼마나 멀리 갈 수 있는지 실험했습니다. 이는 저희의 ["Claude plays Pokémon" 연구](https://www.anthropic.com/news/visible-extended-thinking)와 유사합니다. 저희는 데스크톱 확장 기능을 사용하여 인기 있는 [PyBoy](https://github.com/Baekalfen/PyBoy) GameBoy 에뮬레이터를 열고 Claude가 제어할 수 있도록 하는 단일 확장 기능을 패키징했습니다. 모델의 기능을 사용자가 이미 로컬 기기에 가지고 있는 도구, 데이터 및 애플리케이션에 연결할 수 있는 수많은 기회가 있다고 믿습니다.

![PyBoy MCP와 슈퍼 마리오 랜드 시작 화면을 보여주는 데스크톱](0%20-%20Inbox/ReadItLater/assets/Claude%20Desktop%20Extensions%20One-click%20MCP%20server%20installation%20for%20Claude%20Desktop-thG30d3vCm.webp)

여러분께서 무엇을 구축할지 기대됩니다. 수천 개의 MCP 서버를 탄생시킨 동일한 창의성이 이제 단 한 번의 클릭으로 수백만 명의 사용자에게 도달할 수 있습니다. MCP 서버를 공유할 준비가 되셨습니까? [검토를 위해 확장 기능을 제출하십시오](https://forms.gle/tyiAZvch1kDADKoP9).
