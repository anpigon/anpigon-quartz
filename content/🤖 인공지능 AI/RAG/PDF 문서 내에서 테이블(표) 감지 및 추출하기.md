---
created: 2024-10-04 09:24:29
modified: 2024-10-04 10:04:31
title: PDF 문서 내에서 테이블(표) 감지 및 추출하기
permlink: 20241004t130348062z
tags:
  - rag
  - pdf
  - parser
  - autorag
  - pdfplumber
dg-publish: true
---

[AutoRAG팀의 최근 유튜브 라이브 방송(2024-10-03)](https://www.youtube.com/watch?v=zjUPWtsjdWk)에서 문서 파싱에 대한 흥미로운 접근 방식을 소개했습니다.

PDF 문서에서 표를 처리할 때:

- 표가 포함된 경우: 유료 OCR 모델을 통한 파싱(Upstage 또는 Clova 모델 사용)
- 일반 문서의 경우: 기본 문서 파서 사용

이에 대한 자세한 내용은 [AutoRAG Table Hybrid Parse 문서](https://docs.auto-rag.com/data_creation/parse/table_hybrid_parse.html)에서 확인할 수 있으며, 구현 코드는 [AutoRAG의 GitHub 저장소](https://github.com/Marker-Inc-Korea/AutoRAG/blob/784203b5070747070371f6fd68fb6c5ad694fcc1/autorag/data/parse/table_hybrid_parse.py#L93-L100)에서 확인 가능합니다.

---

## 접근 방법 1: pdfplumber 사용

먼저 `pdfplumber` 라이브러리를 사용해 간단한 테스트를 진행했습니다:

제가 테스트한 코드는 다음과 같습니다.
```python
import pdfplumber

with pdfplumber.open("2303.00716v2.pdf") as pdf:
    page = pdf.pages[0] 
    table = page.extract_table()
    
print(table)
```

하지만 이 방법은 PDF 내 표 감지 성능이 만족스럽지 않았습니다.

특히 복잡한 레이아웃이나 서식이 있는 경우, 표 감지가 잘 안되었고 정확한 표 추출도 어려웠습니다.

## 접근 방법 2: Table Transformer 사용

더 정확한 표 감지를 위해 Microsoft에서 개발한 [table-transformer-detection](https://huggingface.co/microsoft/table-transformer-detection) 모델을 사용해보았습니다. 이 모델은 컴퓨터 비전 기반으로 이미지 내의 표를 감지합니다.

```python
from transformers import AutoImageProcessor, TableTransformerForObjectDetection
from PIL import Image
import torch
import matplotlib.pyplot as plt
import matplotlib.patches as patches

# 이미지 프로세서와 모델 로드
image_processor = AutoImageProcessor.from_pretrained("microsoft/table-transformer-detection")
model = TableTransformerForObjectDetection.from_pretrained("microsoft/table-transformer-detection")

# 이미지 로드 및 전처리
image = Image.open("./2303.00716v2.png").convert("RGB")
inputs = image_processor(images=image, return_tensors="pt")

# 모델 추론
with torch.no_grad():
    outputs = model(**inputs)

# 결과 후처리
target_sizes = torch.tensor([image.size[::-1]])
results = image_processor.post_process_object_detection(outputs, threshold=0.7, target_sizes=target_sizes)[0]

# 탐지된 테이블 시각화
fig, ax = plt.subplots()
ax.imshow(image)

# 결과 출력
for score, label, box in zip(results["scores"], results["labels"], results["boxes"]):
    box = [round(i, 2) for i in box.tolist()]
    print(
        f"탐지된 테이블 | 점수: {round(score.item(), 3)} | 라벨: {model.config.id2label[label.item()]} | 박스 좌표: {box}"
    )
    # Create a Rectangle patch
    rect = patches.Rectangle((box[0], box[1]), box[2]-box[0], box[3]-box[1], linewidth=1, edgecolor='r', facecolor='none')
    # Add the patch to the Axes
    ax.add_patch(rect)

plt.show()
```

### 실행 결과
	탐지된 테이블 | 점수: 1.0 | 라벨: table | 박스 좌표: [77.27, 277.53, 797.86, 410.45]

![](https://i.imgur.com/sHLOxBc.png)

Table Transformer는 PDF를 이미지로 변환하는 과정이 다소 번거로웠지만, 문서 내 표를 높은 정확도로 감지했고, 정확한 위치 좌표도 제공했습니다. 감지된 표 영역은 빨간색 박스로 표시되었습니다.

## 결론

PDF 문서에서 표를 추출할 때:

1. pdfplumber는 속도는 빠르지만 정확도가 떨어질 수 있습니다
2. Table Transformer와 같은 전문화된 모델을 사용하면 더 정확한 표 감지가 가능합니다
3. 감지된 표 영역은 추후 OCR이나 다른 처리 방법과 결합하여 활용할 수 있습니다

---

- 관련 논문:  https://arxiv.org/pdf/2303.00716

- 참고 글: https://velog.io/@jbc21c/Table-Transformer-structure-recognition-%ED%85%8C%EC%9D%B4%EB%B8%94-%EA%B0%90%EC%A7%80-%EB%B0%8F-%EA%B5%AC%EC%A1%B0%EB%B6%84%EC%84%9D
