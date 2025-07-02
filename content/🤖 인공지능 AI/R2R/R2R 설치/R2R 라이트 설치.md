---
title:
aliases:
  - R2R 라이트
related:
created: 2025-07-02 05:13:56
modified: 2025-07-02 05:19:00
---

이 가이드는 Docker 없이 로컬 시스템에 R2R을 설치하고 실행하는 방법을 안내합니다. 이 방식은 R2R 소스 코드에 대한 더 많은 사용자 정의 및 제어를 가능하게 합니다.

## 전제 조건

시작하기 전에 다음 사항이 설치되어 있거나 클라우드에서 사용 가능한지 확인하십시오:

- Python 3.12 이상
- pip (Python 패키지 관리자)
- Git
- Postgres + pgvector

## 추가 종속성 설치

먼저, `core` 추가 종속성과 함께 설치하십시오:

```shell
pip install 'r2r[core]'
```

`core` 종속성은 Postgres 데이터베이스와 결합하여 사용자 대면 R2R 애플리케이션을 프로덕션에 배포하는 데 필요한 구성 요소를 제공합니다.

오케스트레이션 또는 `Unstructured.io`를 사용한 파싱과 같은 고급 기능이 필요한 경우 [전체 설치](https://r2r-docs.sciphi.ai/self-hosting/installation/full)를 참조하십시오.

## 환경 설정

R2R은 다양한 서비스에 대한 연결을 필요로 합니다. 필요에 따라 다음 환경 변수를 설정하십시오:

###### 클라우드 LLM 공급자

R2R 내 LLM 구성에 대한 자세한 정보는 [여기 문서](https://r2r-docs.sciphi.ai/self-hosting/configuration/llm)를 참조하십시오.

```shell
 # 클라우드 LLM 설정
 export OPENAI_API_KEY=sk-...
 # export ANTHROPIC_API_KEY=...
 # ...
```

참고로, R2R은 완전히 로컬에서 실행될 수 있으므로 클라우드 공급자는 선택 사항입니다. 로컬 설치에 대한 자세한 내용은 [여기](https://r2r-docs.sciphi.ai/self-hosting/local-rag)를 참조하십시오.

###### Postgres+pgvector

R2R을 사용하면 자체 Postgres+pgvector 인스턴스 또는 원격 클라우드 인스턴스에 연결할 수 있습니다. R2R 내 Postgres 구성에 대한 자세한 문서는 [여기](https://r2r-docs.sciphi.ai/self-hosting/configuration/database)를 참조하십시오.

```shell
 # Postgres+pgvector 설정
 R2R_POSTGRES_USER=$YOUR_POSTGRES_USER
 R2R_POSTGRES_PASSWORD=$YOUR_POSTGRES_PASSWORD
 R2R_POSTGRES_HOST=$YOUR_POSTGRES_HOST
 R2R_POSTGRES_PORT=$YOUR_POSTGRES_PORT
 R2R_POSTGRES_DBNAME=$YOUR_POSTGRES_DBNAME
 R2R_PROJECT_NAME=$YOUR_PROJECT_NAME # 아래 참고
```

> [!tip] `R2R_PROJECT_NAME` 환경 변수는 선택된 R2R 프로젝트가 상주하는 Postgres 데이터베이스 내의 테이블을 정의합니다. R2R에 필요한 테이블이 존재하지 않으면 R2R 초기화 시 생성됩니다.

Postgres에 익숙하지 않다면 [Supabase의 무료 클라우드 서비스](https://supabase.com/docs)가 좋은 시작점입니다.

###### 웹 도구 공급자

R2R의 Agentic RAG 기능을 사용하여 웹 기반 도구를 사용할 계획이라면 다음을 설정해야 합니다:

```shell
 # web_search 도구용 (Serper API 사용)
 export SERPER_API_KEY=your_serper_api_key_here
 
 # web_scrape 도구용 (Firecrawl API 사용)
 export FIRECRAWL_API_KEY=your_firecrawl_api_key_here
```

이러한 API 키는 다음에서 얻을 수 있습니다:

- Serper: [https://serper.dev/](https://serper.dev/)
- Firecrawl: [https://www.firecrawl.dev/](https://www.firecrawl.dev/)

> [!tip] 이 환경 변수는 Agentic RAG 기능과 함께 `web_search` 또는 `web_scrape` 도구를 사용할 계획인 경우에만 필요합니다. R2R은 로컬 문서 작업 시 이 변수 없이도 작동합니다.

## R2R 실행

r2r 라이브러리 설치 후 다음 명령어를 사용하여 R2R을 시작할 수 있습니다:

```shell
python -m r2r.serve
```

로컬 LLM 사용 시:

```shell
export R2R_CONFIG_NAME=ollama
python -m r2r.serve
```

## Python 개발 모드

R2R을 로컬에서 개발하려는 경우:

1. 종속성 복제 및 설치:
    ```shell
    git clone https://github.com/SciPhi-AI/R2R.git
    cd R2R/py
    pip install -e .[core]
    ```

2. 환경 설정: 위 환경 설정 섹션에 나열된 단계를 따르십시오. 또한, 개발 편의를 위해 로컬 .env 파일을 도입하고, 특정 요구 사항에 맞게 로컬 `r2r.toml`을 사용자 정의할 수 있습니다.

3. 서버 시작:
    ```shell
    python -m r2r.serve
    ```

## 다음 단계

R2R을 성공적으로 설치한 후:

1. **설치 확인**: [http://localhost:7272/v3/health](http://localhost:7272/v3/health)에서 R2R API에 접속하여 모든 구성 요소가 올바르게 실행되는지 확인하십시오.

2. **빠른 시작**: [R2R 빠른 시작 가이드](https://r2r-docs.sciphi.ai/self-hosting/quickstart)를 따라 첫 번째 RAG 애플리케이션을 설정하십시오.

3. **심층 튜토리얼**: 더 포괄적인 이해를 위해 [R2R 워크스루](https://r2r-docs.sciphi.ai/documentation/walkthrough)를 진행하십시오.

4. **설정 사용자 정의**: [구성 가이드](https://r2r-docs.sciphi.ai/self-hosting/configuration/overview)를 통해 R2R 구성 요소를 설정하십시오.

설치 또는 설정 중 문제가 발생하면 [Discord 커뮤니티](https://discord.gg/p6KqD2kjtB) 또는 [GitHub 저장소](https://github.com/SciPhi-AI/R2R)를 통해 도움을 요청하십시오.
