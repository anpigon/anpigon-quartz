---
created: 2024-10-02 12:45:43
modified: 2024-10-02 02:48:13
dg-publish: true
related: "[[OpenAI]]"
---

> 원문: https://platform.openai.com/docs/guides/evals

# 모델 성능 평가하기

AI 모델을 사용하여 개발할 때는 모델의 출력이 정확하고 유용한지 확인하기 위해 지속적으로 테스트하는 것이 필수적입니다. 테스트 데이터를 사용하여 모델의 출력에 대한 평가(흔히 **evals**라고 함)를 정기적으로 실행하면 고품질의 안정적인 AI 애플리케이션을 구축하고 유지하는 데 도움이 됩니다.

OpenAI는 테스트 데이터 세트에서 평가를 생성하고 실행할 수 있도록 [OpenAI 대시보드](https://platform.openai.com/evaluations)에 내장된 도구를 제공합니다. 프로세스는 다음과 같습니다:

1. 테스트 데이터 세트 생성
2. 데이터 세트에 대한 평가를 정의하고 실행합니다.
3. 프롬프트 조정 및/또는 모델 파인튜닝으로 성능 향상
4. 만족할 때까지 반복합니다 🚀.

이 과정이 어떻게 이루어지는지 살펴봅시다!

## 테스트 데이터 세트 생성

소프트웨어 개발에서는 소프트웨어가 제대로 작동하는지 검증하기 위해 프로그램에 필요한 테스트 데이터(<b>픽스처<sub>fixtures</sub></b>라고도 함)를 만들어야 하는 경우가 많습니다. 단위 테스트는 픽스처 데이터로 코드를 실행하고 결과가 예상한 대로 나오는지 확인합니다.

마찬가지로, 평가에는 모델이 제대로 응답할 수 있어야 하는 일련의 테스트 입력이 필요합니다. 모델이 받게 될 요청 유형을 대표하지 않는 데이터로 모델을 테스트하면 알 수 없는 새로운 입력에 대해 모델이 어떻게 작동할지 확신할 수 없기 때문에 좋은 테스트 데이터를 확보하는 것은 LLM 정확도를 최적화하는 데 매우 중요합니다.

### 실제 트래픽에서 데이터 세트 생성하기

대표적인 테스트 데이터 세트를 생성하는 가장 좋은 방법 중 하나는 사용자의 실제 프로덕션 요청을 사용하는 것입니다. 이는 **Stored Completions**을 사용하여 가능합니다. LLM 응답을 생성하는 코드에서 아래 IT 지원 챗봇의 예에서와 같이 `store: true` 매개 변수를 사용하고 나중에 완료를 필터링하는 데 사용할 수 있는 `metadata` 태그를 포함하세요:

**API에서 메타데이터와 함께 Store completions 사용하기**
```javascript
import OpenAI from "openai"; 
const openai = new OpenAI(); 

const response = await openai.chat.completions.create({ 
	model: "gpt-4o", 
	messages: [ 
		{ role: "system", content: "You are a corporate IT support expert." }, 
		{ role: "user", content: "How can I hide the dock on my Mac?"}, 
	], 
	store: true, 
	metadata: { 
		role: "manager", 
		department: "accounting", 
		source: "homepage" 
	} 
}); 
console.log(response.choices[0]);
```

이렇게 하면 완료가 [여기 대시보드](https://platform.openai.com/chat-completions)에 표시됩니다.

![stored completion](https://i.imgur.com/pQrcZj6.png)

트래픽을 일부 기록했으면 중요한 기준에 따라 모델 출력을 테스트하는 평가를 만들 수 있습니다. 원하는 데이터 집합으로 완료를 필터링한 다음 'Evaluate' 버튼을 클릭합니다.

![create eval](https://i.imgur.com/KFMJc3D.png)

여기에서 모델의 출력을 판단하는 평가를 정의할 수 있습니다.

## 테스트 데이터에 대해 평가 정의 및 실행하기

수동으로 또는 [completions](https://platform.openai.com/chat-completions) UI의 플로우를 사용하여 테스트 데이터 세트를 생성한 후에는 평가 실행의 매개변수를 정의할 수 있습니다. 위의 단계를 따라 프로덕션 트래픽에서 테스트 데이터를 생성했다면 completions를 다시 실행할 필요가 없습니다. 바로 평가 기준을 정의할 수 있습니다.

![create criteria](https://i.imgur.com/Xb0BTle.png)

선택할 수 있는 여러 가지 평가 기준(**graders**라고도 함)이 있으며, 이러한 테스트는 모델 응답의 품질을 평가하는 데 도움이 됩니다. 한 가지 유연한 옵션은 모델 평가자로, 적절하다고 생각되는 모델 산출물을 채점하라는 메시지를 표시할 수 있습니다.

![add criteria](https://i.imgur.com/RCeEGnV.png)

모델에 대한 기준을 정의하고 나면 평가를 실행할 수 있습니다!

![create criteria](https://i.imgur.com/V9EGyGV.png)

## 반복 및 개선

평가가 실행되면 대시보드에서 결과 점수를 확인할 수 있습니다. 프롬프트와 기준을 반복하여 시간이 지남에 따라 모델 출력을 개선할 수 있습니다. 좋은 평가와 좋은 테스트 데이터가 있으면 프롬프트를 반복하고 생성 결과가 양호하다는 확신을 가지고 새로운 모델을 시도하는 데 도움이 될 수 있습니다.

- [파인튜닝: 파인튜닝: 사용 사례에 맞는 응답을 생성하는 모델의 기능을 개선하기](https://platform.openai.com/docs/guides/fine-tuning)
- [[Model Distillation|모델 디스틸레이션: 대규모 모델 결과를 더 작고 저렴하며 빠른 모델로 디스틸레이션하는 방법을 알아보세요.]]

