---
title: 
related: 
aliases: Gemini CLI
created: 2025-06-26 10:39:34
modified: 2025-06-26
---

> 원문: https://github.com/google-gemini/gemini-cli/blob/main/README.md

# Gemini CLI

![Gemini CLI 스크린샷](https://raw.githubusercontent.com/google-gemini/gemini-cli/refs/heads/main/docs/assets/gemini-screenshot.png)

이 저장소는 Gemini CLI를 포함합니다. 이는 명령줄 기반 AI 워크플로 도구로, 여러분의 도구와 연결되고 코드를 이해하며 워크플로를 가속화합니다.

Gemini CLI를 사용하면 다음과 같은 작업이 가능합니다:

- Gemini의 100만 토큰 컨텍스트 윈도우를 초월하여 대규모 코드베이스를 질의하고 수정할 수 있습니다.
- Gemini의 멀티모달 기능을 활용하여 PDF나 스케치에서 새로운 애플리케이션을 생성할 수 있습니다.
- 복잡한 리베이스 처리나 풀 리퀘스트 질의 같은 운영 작업을 자동화할 수 있습니다.
- 도구 및 MCP 서버를 사용하여 새로운 기능을 연결할 수 있습니다. 이에는 [Imagen, Veo 또는 Lyria를 통한 미디어 생성](https://github.com/GoogleCloudPlatform/vertex-ai-creative-studio/tree/main/experiments/mcp-genmedia)이 포함됩니다.
- Gemini에 내장된 [Google 검색](https://ai.google.dev/gemini-api/docs/grounding) 도구로 질문을 근거 있게 할 수 있습니다.

## 시작하기

1. **필수 조건:** [Node.js 버전 18](https://nodejs.org/en/download) 이상이 설치되어 있는지 확인하십시오.
2. **CLI 실행:** 터미널에서 다음 명령어를 실행하십시오:

   ```bash
   npx https://github.com/google-gemini/gemini-cli
   ```

   또는 다음으로 설치합니다:

   ```bash
   npm install -g @google/gemini-cli
   gemini
   ```

3. **색상 테마 선택**
4. **인증:** 프롬프트가 나타나면 개인 Google 계정으로 로그인하십시오. 이를 통해 Gemini를 사용하여 분당 최대 60회, 하루 1,000회의 모델 요청이 허용됩니다.

이제 Gemini CLI를 사용할 준비가 되었습니다!

### 고급 사용법 또는 상향된 한도

특정 모델을 사용하거나 더 높은 요청 용량이 필요한 경우 API 키를 이용할 수 있습니다:

1. [Google AI Studio](https://aistudio.google.com/apikey)에서 키를 생성하십시오.
2. 터미널에서 환경 변수로 설정합니다. `YOUR_API_KEY`를 생성한 키로 교체하십시오:

   ```bash
   export GEMINI_API_KEY="YOUR_API_KEY"
   ```

Google Workspace 계정을 포함한 기타 인증 방법은 [인증](./docs/cli/authentication.md) 가이드를 참조하십시오.

## 예시

CLI를 실행한 후에는 셸에서 Gemini와 상호작용을 시작할 수 있습니다.

새 디렉터리에서 프로젝트를 시작할 수 있습니다:

```sh
cd new-project/
gemini
> 제가 제공할 FAQ.md 파일을 사용하여 질문에 답변하는 Gemini 디스코드 봇을 작성해 주세요
```

또는 기존 프로젝트로 작업할 수 있습니다:

```sh
git clone https://github.com/google-gemini/gemini-cli
cd gemini-cli
gemini
> 어제 들어간 모든 변경 사항의 요약을 제공해 주세요
```

### 다음 단계

- [소스 기여 또는 빌드](https://github.com/google-gemini/gemini-cli/blob/main/CONTRIBUTING.md) 방법을 학습합니다.
- 사용 가능한 [CLI 명령어](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/commands.md)를 탐색합니다.
- 문제 발생 시 [문제 해결 가이드](https://github.com/google-gemini/gemini-cli/blob/main/docs/troubleshooting.md)를 참조합니다.
- 포괄적인 문서는 [전체 문서](https://github.com/google-gemini/gemini-cli/blob/main/docs/index.md)를 확인하십시오.
- 추가 영감을 위해 [인기 태스크](#인기-태스크)를 살펴보세요.

### 문제 해결

문제가 발생하면 [문제 해결](docs/troubleshooting.md) 가이드를 참조하십시오.

## 인기 태스크

### 새로운 코드베이스 탐색

기존 또는 새로 클론된 저장소에 `cd`로 이동한 후 `gemini`를 실행합니다.

```text
> 해당 시스템 아키텍처의 주요 부분을 설명해 주세요.
```

```text
> 어떤 보안 메커니즘이 적용되어 있나요?
```

### 기존 코드 작업

```text
> GitHub 이슈 #123에 대한 초안을 구현하세요.
```

```text
> 이 코드베이스를 최신 Java 버전으로 이전하려고 합니다. 계획부터 시작해 주세요.
```

### 워크플로 자동화

MCP 서버를 사용하여 로컬 시스템 도구와 엔터프라이즈 협업 제품군을 통합합니다.

```text
> 지난 7일간의 Git 기록을 기능 및 팀원별로 그룹화한 슬라이드 데크를 만들어 주세요.
```

```text
> 가장 많이 상호작용된 GitHub 이슈를 보여주는 벽걸이 디스플레이용 전체 화면 웹 앱을 만들어 주세요.
```

### 시스템과 상호작용

```text
> 이 디렉터리의 모든 이미지를 png로 변환하고, exif 데이터의 날짜를 사용해 이름을 변경해 주세요.
```

```text
> 제 PDF 청구서를 지출 월별로 정리해 주세요.
```

## 서비스 약관 및 개인정보 처리방침

Gemini CLI 사용에 적용되는 서비스 약관 및 개인정보 처리방침 세부사항은 [서비스 약관 및 개인정보 처리방침](./docs/tos-privacy.md)을 참조하십시오.