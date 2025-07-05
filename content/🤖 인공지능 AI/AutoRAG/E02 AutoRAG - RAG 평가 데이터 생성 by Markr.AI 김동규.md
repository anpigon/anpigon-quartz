---
created: 2024-10-06 10:44:31
modified: 2024-10-06 11:43:32
dg-publish: true
---

> [!summary]
> AutoRAG 평가 데이터셋을 제작하는 방법에 대해 소개하는 영상입니다.
> AutoRAG을 사용할 때 핵심은 평가 데이터셋을 만드는 것입니다.
> 이를 통해 정확한 성능을 평가하고 RAG을 최적화할 수 있습니다.
> 동규님은 굉장히 많은 시간을 데이터셋 구성에 투자해야 한다고 강조합니다.
> AutoRAG를 통해 QA 데이터를 생성하거나, LM 모델을 이용하여 코퍼스 데이터셋을 만들 수 있습니다.
> 프롬프트를 잘 작성하고 고성능 LLM을 사용하여 품질 높은 데이터셋을 생성하는 것이 가장 중요합니다.

![](https://www.youtube.com/watch?v=l1j4QUELLNY)
> AutoRAG 한국어 튜토리얼: https://github.com/Marker-Inc-Korea/AutoRAG-tutorial-ko

## 설치하기

```sh
pip install -Uq AUtoRAG
```

## 문서 파싱
### `run_parse.py` 파일 작성

```python
import os

from autorag.parser import Parser
import click
from dotenv import load_dotenv

root_dir = os.path.dirname(os.path.realpath(__file__))


@click.command()
@click.option('--data_path_glob', type=click.Path(exists=False, dir_okay=True),
			  default=os.path.join(root_dir, "raw_docs", "*.pdf"))
@click.option('--config', type=click.Path(exists=True, dir_okay=False), default=os.path.join(root_dir, "config", "parse.yaml"))
@click.option('--project_dir', type=click.Path(dir_okay=True), default=os.path.join(root_dir, "parsed_raw"))
def main(data_path_glob, config, project_dir):
	load_dotenv()

	if not os.path.exists(project_dir):
		os.makedirs(project_dir)

	parser = Parser(data_path_glob=data_path_glob, project_dir=project_dir)
	parser.start_parsing(config)


if __name__ == '__main__':
	main()
```

- `data_path_glob`: 파싱할 문서 파일의 경로 (기본값: `raw_docs/*.pdf`)
- `config`: 파싱 설정 파일의 경로 (기본값: `config/parse.yaml`)
- `project_dir`: 파싱된 결과를 저장할 디렉토리 (기본값: `parsed_raw`)

### `config/parse.yaml` 파일 작성

```yaml
modules:
  - module_type: langchain_parse
    parse_method: [ pdfminer, pdfplumber, pypdfium2, pypdf, pymupdf ]
  - module_type: langchain_parse
    parse_method: upstagedocumentparse
  - module_type: llamaparse
    result_type: markdown
    language: ko
  - module_type: table_hybrid_parse
    text_parse_module: langchain_parse
    text_params:
      parse_method: pdfplumber
    table_parse_module: langchain_parse
    table_params:
      parse_method: upstagedocumentparse
```
> [Parse](https://docs.auto-rag.com/data_creation/parse/parse.html), [Langchain Parse](https://docs.auto-rag.com/data_creation/parse/langchain_parse.html), [Table Hybrid Parse](https://docs.auto-rag.com/data_creation/parse/table_hybrid_parse.html)

#### [LamaParse](https://docs.llamaindex.ai/en/stable/llama_cloud/llama_parse)

LamaParse는 하루에 1,000페이지가 무료로 제공됩니다. 유료 요금제에 가입하면 일주일에 7천 개의 무료 페이지가 제공되고 각 페이지당 \$0.003이 부과됩니다. [Pricing 참고](https://docs.llamaindex.ai/en/stable/llama_cloud/llama_parse/#pricing)

[여기](https://cloud.llamaindex.ai/api-key)에서 LamaParse API Key를 발급합니다. 그리고 `.env`에 `LLAMA_CLOUD_API_KEY` 를 작성합니다.

#### `.env`
```properties
LLAMA_CLOUD_API_KEY=llx-⋯
```

#### [Upstage Document Parse](https://developers.upstage.ai/docs/apis/document-parse)

Upstage Document Parse는 유료입니다. [Pricing 참고](https://www.upstage.ai/pricing?utm_term=Pricing)

[여기](https://console.upstage.ai/api-keys)에서 Upstage API Key를 발급합니다. 그리고 `.env`에 `UPSTAGE_API_KEY` 를 작성합니다.

#### `.env`
```properties
UPSTAGE_API_KEY=up_⋯
```

### 문서 파싱 실행

```sh
python run_parse.py
```

## 문서 분할하기

### `run_chunk.py` 파일 작성

```python
import os

import click
from dotenv import load_dotenv

from autorag.chunker import Chunker

root_dir = os.path.dirname(os.path.realpath(__file__))

@click.command()
@click.option('--raw_path', type=click.Path(exists=True, dir_okay=False, file_okay=True),
			  default=os.path.join(root_dir, "parsed_raw", "0", "5.parquet"))
@click.option('--config', type=click.Path(exists=True, dir_okay=False), default=os.path.join(root_dir, "config", "chunk.yaml"))
@click.option('--project_dir', type=click.Path(dir_okay=True), default=os.path.join(root_dir, "chunked_corpus"))
def main(raw_path, config, project_dir):
	load_dotenv()

	if not os.path.exists(project_dir):
		os.makedirs(project_dir)

	parser = Chunker.from_parquet(raw_path, project_dir=project_dir)
	parser.start_chunking(config)


if __name__ == '__main__':
    main()
```

- `raw_path`: 분할할 parquet 파일의 경로 (기본값: `parsed_raw/0/5.parquet)
- `config`: 분할 설정 파일의 경로 (기본값: `config/chunk.yaml`)
- `project_dir`: 파싱된 결과를 저장할 디렉토리 (기본값: `chunked_corpus`)

### `config/chunk.yaml` 파일 작성

```yaml
modules:
  - module_type: llama_index_chunk
    chunk_method: [ Token, Sentence ]
    chunk_size: [ 1024, 512 ]
    chunk_overlap: [ 24 ]
    add_file_name: ko
  - module_type: llama_index_chunk
    chunk_method: [ SentenceWindow ]
    sentence_splitter: kiwi
    window_size: 3
    add_file_name: ko
  - module_type: llama_index_chunk
    chunk_method: [ Semantic_llama_index ]
    embed_model: openai
    buffer_size: 1
    breakpoint_percentile_threshold: 95
    add_file_name: ko
  - module_type: langchain_chunk
    chunk_method: recursivecharacter
    separators: [ " ", "\n" ]
```
> [Chunk](https://docs.auto-rag.com/data_creation/chunk/chunk.html),  [Langchain Chunk](https://docs.auto-rag.com/data_creation/chunk/langchain_chunk.html), [Llama Index Chunk](https://docs.auto-rag.com/data_creation/chunk/llama_index_chunk.html)

### 문서 분할 실행

```sh
python run_chunk.py
```

