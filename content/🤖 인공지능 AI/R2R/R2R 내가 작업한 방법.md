---
title:
aliases: []
related:
created: 2025-07-02 11:03:29
modified: 2025-07-02 11:24:26
---

###### 1. R2R 저장소 클론

```shell
git clone https://github.com/SciPhi-AI/R2R.git
cd R2R/docker
```

###### 2. `R2R/docker/env/r2r-full.env` 파일 편집

```ENV
R2R_CONFIG_PATH=/app/user_configs/full_ollama.toml
# R2R_CONFIG_NAME 주석 처리
```

###### 3. `R2R/docker/user_configs/full_ollama.toml` 파일 작성

> 참고 소스: https://github.com/SciPhi-AI/R2R/blob/main/py/core/configs/full_ollama.toml

```toml
[app]
# LLM used for internal operations, like deriving conversation names
fast_llm = "ollama/gemma3n"

# LLM used for user-facing output, like RAG replies
quality_llm = "ollama/deepseek-r1"

# LLM used for ingesting visual inputs
vlm = "ollama/gemma3" # TODO - Replace with viable candidate

# LLM used for transcription
audio_lm = "ollama/gemma3" # TODO - Replace with viable candidate


# Reasoning model, used for `research` agent
reasoning_llm = "ollama/deepseek-r1"
# Planning model, used for `research` agent
planning_llm = "ollama/deepseek-r1"

[embedding]
provider = "ollama"
base_model = "bge-m3"
base_dimension = 1_024
batch_size = 128
concurrent_request_limit = 2

[completion_embedding]
provider = "ollama"
base_model = "bge-m3"
base_dimension = 1_024
batch_size = 128
concurrent_request_limit = 2

[agent]
tools = ["search_file_knowledge"]

[completion]
provider = "litellm"
concurrent_request_limit = 1

  [completion.generation_config]
  temperature = 0.1
  top_p = 1
  max_tokens_to_sample = 1_024
  stream = false
  api_base = "http://host.docker.internal:11434"

[ingestion]
provider = "unstructured_local"
strategy = "auto"
chunking_strategy = "by_title"
new_after_n_chars = 512
max_characters = 1_024
combine_under_n_chars = 128
overlap = 20
chunks_for_document_summary = 16
document_summary_model = "ollama/gemma3n"
automatic_extraction = false

[orchestration]
provider = "hatchet"
```

###### 4. 도커 실행

```shell
docker compose -f 'compose.full.yaml' up -d
```