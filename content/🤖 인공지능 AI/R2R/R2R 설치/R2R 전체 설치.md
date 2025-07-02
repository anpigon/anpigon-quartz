---
title:
aliases:
  - R2R
related:
created: 2025-07-02 05:07:15
modified: 2025-07-02 05:18:33
---

> 원문: https://r2r-docs.sciphi.ai/self-hosting/installation/full

> [!tip] 이 설치 가이드는 R2R 전체 버전을 위한 것입니다. 단독 개발자나 프로토타입을 제작하는 팀의 경우, [[R2R 라이트 설치|R2R 라이트]]부터 시작하는 것을 권장합니다.

이 가이드는 Docker를 사용하여 R2R을 설치하고 실행하는 방법을 안내합니다. 이는 가장 빠르고 쉬운 시작 방법입니다.

## 사전 준비 사항

- 시스템에 Docker가 설치되어 있어야 합니다. 아직 Docker를 설치하지 않았다면, [공식 Docker 설치 가이드](https://docs.docker.com/engine/install/)를 참조하십시오.

## 설치

### 1. R2R 저장소 클론

Docker compose 파일에 접근하기 위해 R2R 저장소를 클론합니다.

```shell
git clone https://github.com/SciPhi-AI/R2R.git
cd R2R/docker
```

### 2. 환경 변수 설정

> [!tip] 전체 R2R 설치는 기본 `r2r.toml` 대신 사전 빌드된 사용자 정의 구성 파일인 [`full.toml`](https://github.com/SciPhi-AI/R2R/blob/main/py/core/configs/full.toml)을 사용합니다.

`env` 디렉토리로 이동하여 환경 변수를 설정합니다.

```shell
cd env
# 선호하는 텍스트 편집기로 r2r-full.env 파일을 편집합니다.
sudo nano r2r-full.env
```

### 필수 환경 변수

### 구성 선택 (하나 선택)

| 변수             | 설명                               | 기본값         |
| :--------------- | :--------------------------------- | :------------- |
| `R2R_CONFIG_NAME` | 사전 정의된 구성 사용              | `full` (OpenAI) |
| `R2R_CONFIG_PATH` | 사용자 정의 TOML 구성 파일 경로    | 없음           |

> 로컬 모델을 사용하려면 `R2R_CONFIG_NAME=full_ollama`로 설정하십시오.

### LLM API 키 (최소 하나 필수)

| 제공자   | 환경 변수          | 사용 조건                     |
| :------- | :----------------- | :---------------------------- |
| OpenAI   | `OPENAI_API_KEY`   | `R2R_CONFIG_NAME=full`        |
| Anthropic | `ANTHROPIC_API_KEY` | 사용자 정의 구성 또는 런타임 오버라이드 |
| Ollama   | `OLLAMA_API_BASE`  | `R2R_CONFIG_NAME=full_ollama` |

> Ollama의 기본값은 `http://host.docker.internal:11434`입니다.

### 외부 에이전트 도구 (선택 사항)

| 도구         | 환경 변수          | 목적             | 제공자 링크                       |
| :----------- | :----------------- | :--------------- | :-------------------------------- |
| `web_search` | `SERPER_API_KEY`   | 웹 검색 도구 활성화 | [Serper](https://serper.dev/)     |
| `web_scrape` | `FIRECRAWL_API_KEY` | 웹 스크랩 도구 활성화 | [Firecrawl](https://www.firecrawl.dev/) |

> [!tip] 이 환경 변수들은 Agentic RAG 기능과 함께 `web_search` 또는 `web_scrape` 도구를 사용할 계획이 있을 때만 필요합니다. R2R은 로컬 문서 작업 시 이 변수들 없이도 작동합니다.

에이전트 도구와 함께 R2R을 시작할 때, 다음 변수들을 실행 명령에 포함하십시오.

추가 옵션은 [전체 구성 가이드](https://r2r-docs.sciphi.ai/self-hosting/configuration/overview)를 참조하십시오.

```shell
# 클라우드 LLM 및 에이전트 도구 사용 예시
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-...
export SERPER_API_KEY=your_serper_api_key_here
export FIRECRAWL_API_KEY=your_firecrawl_api_key_here

COMPOSE_PROFILES=postgres docker compose -f compose.full.yaml up -d
```

### 3. 사용자 정의 구성 (선택 사항)

내장된 옵션 대신 사용자 정의 구성 파일을 사용하는 경우, 다음 단계를 따르십시오.

1. `user_configs` 디렉토리에 TOML 구성 파일을 생성합니다.
    ```shell
    # user_configs 디렉토리로 이동
    cd user_configs
    
    # 새 구성 파일 생성 (예: my_config.toml)
    touch my_config.toml
    
    # 구성 설정으로 파일 편집
    nano my_config.toml
    ```

2. `r2r-full.env` 파일을 업데이트하여 이 구성을 가리키도록 합니다.
    ```sh
    R2R_CONFIG_PATH=/app/user_configs/my_config.toml
    ```

> [!warning] `R2R_CONFIG_PATH`의 경로는 로컬 시스템 경로가 아닌 컨테이너 경로(`/app/user_configs/`)를 사용해야 합니다. 지정된 구성 파일이 `user_configs` 디렉토리에 실제로 존재하는지 확인하십시오. 지정된 경로에서 파일을 찾을 수 없으면 애플리케이션이 시작되지 않습니다.

예시 및 구성 템플릿은 [구성 가이드](https://r2r-docs.sciphi.ai/self-hosting/configuration/overview)를 참조하십시오.

### 4. R2R 서비스 시작

`docker` 디렉토리로 돌아가 서비스를 시작합니다.

```shell
cd ..
docker compose -f compose.full.yaml --profile postgres up -d
# `--profile postgres`는 외부 Postgres를 사용할 경우 생략할 수 있습니다.
```

### 5. R2R과 상호작용

Python 또는 JS SDK를 설치하거나 [http://localhost:7273](http://localhost:7273/)으로 이동하여 대시보드를 통해 R2R과 상호작용합니다.

Python SDK를 설치하려면:

```shell
pip install r2r
```

## 다음 단계

R2R을 성공적으로 설치한 후:

1. **설치 확인**: [http://localhost:7272/v3/health](http://localhost:7272/v3/health)에서 R2R API에 접속하여 모든 구성 요소가 올바르게 실행되는지 확인하십시오.
2. **빠른 시작**: [R2R 빠른 시작 가이드](https://r2r-docs.sciphi.ai/self-hosting/quickstart)를 따라 첫 번째 RAG 애플리케이션을 설정하십시오.
3. **심층 튜토리얼**: 더 포괄적인 이해를 위해 [R2R 워크스루](https://r2r-docs.sciphi.ai/documentation/walkthrough)를 진행하십시오.
4. **설정 사용자 정의**: R2R 시스템을 [구성](https://r2r-docs.sciphi.ai/self-hosting/configuration/overview)하십시오.

설치 또는 설정 중 문제가 발생하면, [Discord 커뮤니티](https://discord.gg/p6KqD2kjtB) 또는 [GitHub 저장소](https://github.com/SciPhi-AI/R2R)를 통해 도움을 요청하십시오.
