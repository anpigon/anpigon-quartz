---
title:
related:
aliases: Gemini CLI
created: 2025-06-26 11:52:41
modified: 2025-06-27 12:00:15
---

# Gemini CLI

Gemini CLI 내에서, `packages/cli`는 사용자가 Gemini AI 모델 및 관련 도구와 프롬프트를 송수신할 수 있는 프론트엔드입니다. Gemini CLI에 대한 전반적인 개요는 [주요 문서 페이지](../index.md)를 참조하시기 바랍니다.

## 이 섹션 탐색

- **[인증](./authentication.md):** Google AI 서비스 인증 설정에 대한 안내입니다.
- **[명령어](./commands.md):** Gemini CLI 명령어(예: `/help`, `/tools`, `/theme`)에 대한 참조입니다.
- **[구성](./configuration.md):** 구성 파일을 사용하여 Gemini CLI 동작을 맞춤 설정하는 방법에 대한 안내입니다.
- **[토큰 캐싱](./token-caching.md):** 토큰 캐싱을 통해 API 비용을 최적화합니다.
- **[테마](./themes.md):** 다양한 테마로 CLI 모양을 사용자 지정하는 방법에 대한 안내입니다.
- **[튜토리얼](tutorials.md):** Gemini CLI를 사용하여 개발 작업을 자동화하는 방법을 보여주는 튜토리얼입니다.

## 비대화형 모드

Gemini CLI는 스크립팅 및 자동화에 유용한 비대화형 모드로 실행될 수 있습니다. 이 모드에서는 CLI로 입력을 파이프하면, 명령을 실행한 다음 종료됩니다.

다음 예시는 터미널에서 Gemini CLI로 명령을 파이프하는 방법을 보여줍니다:

```bash
echo "What is fine tuning?" | gemini
```

Gemini CLI는 명령을 실행하고 그 출력을 터미널에 출력합니다. `--prompt` 또는 `-p` 플래그를 사용하여 동일한 동작을 구현할 수 있습니다. 예를 들어 다음과 같습니다:

```bash
gemini -p "What is fine tuning?"
```