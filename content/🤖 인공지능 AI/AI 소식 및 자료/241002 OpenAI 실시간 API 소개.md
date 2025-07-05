---
created: 2024-10-02 11:04:12
modified: 2024-10-03 09:34:40
title: OpenAI 실시간 API 소개
dg-publish: true
tags:
  - open-ai
---

> 출처: https://openai.com/index/introducing-the-realtime-api/

# 실시간 API 소개 | OpenAI

*October 1, 2024*

이제 개발자는 애플리케이션에 빠른 음성 대화 환경을 구축할 수 있습니다.

 오늘, 모든 유료 개발자가 앱에서 지연 시간이 짧은 멀티모달 경험을 구축할 수 있는 실시간 API의 공개 베타 버전을 소개합니다. ChatGPT의 고급 음성 모드와 마찬가지로, 실시간 API는 이미 API에서 지원되는 [6개의 사전 설정 음성](https://platform.openai.com/docs/guides/text-to-speech)을 사용하여 자연스러운 음성 대 음성 대화를 지원합니다.

또한, 실시간 API의 짧은 지연 시간이 필요하지 않은 사용 사례를 지원하기 위해 [Chat Completions API](https://platform.openai.com/docs/guides/chat-completions)에 오디오 입력 및 출력 기능을 도입합니다. 이 업데이트를 통해 개발자는 텍스트 또는 오디오 입력을 [GPT-4o](https://openai.com/index/hello-gpt-4o/)에 전달하고 모델이 텍스트, 오디오 또는 두 가지 중 원하는 것을 선택하여 응답하도록 할 수 있습니다.

언어 앱과 교육용 소프트웨어부터 고객 지원 경험에 이르기까지 개발자들은 이미 음성 경험을 활용하여 사용자와 소통하고 있습니다. 이제 실시간 API와 곧 채팅 완성 API의 오디오를 통해 개발자는 더 이상 이러한 경험을 구현하기 위해 여러 모델을 조합할 필요가 없습니다. 그 대신 단 한 번의 API 호출로 자연스러운 대화 환경을 구축할 수 있습니다.****

## 작동 방식

이전에는 개발자가 유사한 음성 비서 환경을 만들려면 [Whisper](https://openai.com/index/whisper/)와 같은 자동 음성 인식 모델을 사용하여 오디오를 텍스트로 변환하고, 추론이나 추론을 위해 텍스트 모델에 텍스트를 전달한 다음, [텍스트 음성 변환(text-to-speech)](https://platform.openai.com/docs/guides/text-to-speech) 모델을 사용하여 모델의 출력을 재생해야 했습니다. 이러한 접근 방식은 종종 감정, 강조 및 악센트가 손실되고 대기 시간이 눈에 띄게 길어지는 결과를 초래했습니다. 채팅 완성 API를 사용하면 개발자는 단 한 번의 API 호출로 전체 프로세스를 처리할 수 있지만, 사람의 대화보다는 여전히 느립니다. 실시간 API는 오디오 입력과 출력을 직접 스트리밍하여 이 문제를 개선하여 보다 자연스러운 대화 환경을 구현할 수 있습니다. 또한 ChatGPT의 고급 음성 모드처럼 자동으로 중단을 처리할 수도 있습니다.

내부적으로는 실시간 API를 통해 영구적인 웹소켓 연결을 생성하여 GPT-4o와 메시지를 교환할 수 있습니다. 이 API는 [함수 호출(function calling)](https://platform.openai.com/docs/guides/function-calling)을 지원하므로 음성 어시스턴트가 작업을 트리거하거나 새로운 컨텍스트를 가져와서 사용자 요청에 응답할 수 있습니다. 예를 들어 음성 어시스턴트는 사용자를 대신하여 주문을 하거나 관련 고객 정보를 검색하여 개인화된 응답을 제공할 수 있습니다.

### 고객 지원 에이전트, 언어 학습 어시스턴트 등 지원 강화

반복적인 배포 전략의 일환으로 소수의 파트너와 함께 실시간 API를 테스트하여 구축하는 동안 피드백을 수집하고 있습니다.

몇 가지 유망한 초기 사용 사례는 다음과 같습니다:

- [영양 및 피트니스 코칭 앱인 Healthify는 실시간 API를 사용하여 AI 코치 Ria와 자연스럽게 대화하면서 다음과 같은 경우 인간 영양사를 참여시킬 수 있습니다.](https://player.vimeo.com/video/1014799468?h=ae110f3d29&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479)
- [언어 학습 앱인 Speak는 실시간 API를 사용하여 역할극 기능을 강화하여 사용자가 새로운 언어로 대화를 연습할 수 있도록 지원합니다.](https://player.vimeo.com/video/1014803163?h=e73738c5cd&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479)

## 사용 가능 여부 및 가격

실시간 API는 오늘부터 모든 유료 개발자를 대상으로 공개 베타 버전으로 출시됩니다. 실시간 API의 오디오 기능은 새로운 GPT-4o 모델인 `gpt-4o-realtime-preview`를 통해 제공됩니다.

채팅 완료 API의 오디오는 앞으로 몇 주 내에 새로운 모델인 `gpt-4o-audio-preview`로 출시될 예정입니다. `gpt-4o-audio-preview`를 사용하면 개발자는 텍스트 또는 오디오를 GPT-4o에 입력하고 텍스트, 오디오 또는 두 가지 모두로 응답을 받을 수 있습니다.

실시간 API는 텍스트 토큰과 오디오 토큰을 모두 사용합니다. 텍스트 입력 토큰의 가격은 1M당 \$5, 출력 토큰의 가격은 1M당 \$20입니다. 오디오 입력은 1M 토큰당 100달러, 출력은 1M 토큰당 200달러로 책정됩니다. 이는 오디오 입력의 경우 분당 약 0.06달러, 오디오 출력의 경우 분당 0.24달러에 해당하는 금액입니다. Chat Completions API의 오디오도 동일한 가격이 적용됩니다.

## 안전 및 개인정보 보호

실시간 API는 플래그가 지정된 모델 입력 및 출력에 대한 자동화된 모니터링 및 인적 검토를 포함하여 여러 계층의 안전 보호 기능을 사용하여 API 남용의 위험을 완화합니다. 실시간 API는 [Preparedness 프레임워크](https://openai.com/preparedness)에 자세히 설명된 [GPT-4o 시스템 카드](https://openai.com/index/gpt-4o-system-card/#observed-safety-challenges-evaluations-and-mitigations)에 따른 평가를 포함하여 자동화된 평가와 인적 평가를 모두 사용하여 신중하게 평가한 ChatGPT의 고급 음성 모드를 구동하는 것과 동일한 버전의 GPT-4o를 기반으로 구축되었습니다. 또한 실시간 API는 고급 음성 모드를 위해 구축한 것과 동일한 오디오 안전 인프라를 활용하며, 테스트 결과 피해 가능성을 줄이는 데 도움이 되는 것으로 나타났습니다.

스팸, 오해의 소지가 있거나 다른 사람에게 해를 끼치기 위해 서비스 결과물을 용도 변경하거나 배포하는 것은 [사용 정책](https://openai.com/policies/usage-policies/)에 위배되며, 남용 가능성을 적극적으로 모니터링하고 있습니다. 또한, 개발자는 문맥상 명백한 경우가 아니라면 사용자가 AI와 상호작용하고 있다는 사실을 사용자에게 명확히 알려야 한다는 정책도 준수해야 합니다.

출시 전에 외부 레드팀 네트워크와 함께 실시간 API를 테스트한 결과, 실시간 API가 기존 완화 조치로 커버되지 않는 고위험 공백을 초래하지 않는다는 사실을 확인했습니다. 모든 API 서비스와 마찬가지로 실시간 API는 [기업 개인정보 보호](https://openai.com/enterprise-privacy/) 약정의 적용을 받습니다. 당사는 사용자의 명시적인 허가 없이 이 서비스에 사용되는 입력 또는 출력에 대해 모델을 훈련시키지 않습니다.

## 시작하기

개발자는 앞으로 [Playground](https://platform.openai.com/playground/realtime)에서 또는 [문서](http://platform.openai.com/docs/guides/realtime) 및 [참조 클라이언트](https://github.com/openai/openai-realtime-api-beta)를 사용하여 실시간 API로 빌드를 시작할 수 있습니다.

또한 [LiveKit](https://docs.livekit.io/agents/openai/) 및 [Agora](https://www.agora.io/en/products/agora-openai-conversational-ai-sdk/)와 협력하여 에코 제거, 재연결, 소음 차단과 같은 오디오 구성 요소의 클라이언트 라이브러리를 만들었으며, [Twilio](https://www.twilio.com/en-us/blog/twilio-openai-realtime-api-launch-integration)와 협력하여 개발자가 음성 통화를 통해 AI 가상 에이전트를 원활하게 구축, 배포 및 고객과 연결할 수 있는 [Twilio의 음성 API](https://www.twilio.com/en-us/voice)와 Realtime API를 통합하고 있습니다.

## 다음 단계

일반 공개를 위해 노력하면서 실시간 API를 개선하기 위한 피드백을 적극적으로 수집하고 있습니다. 도입할 예정인 기능 중 일부는 다음과 같습니다:

- **더 많은 모달리티**: 우선 실시간 API는 음성을 지원하며, 향후 시각 및 동영상과 같은 모달리티를 추가할 계획입니다.
- **속도 제한 증가**: 현재 API는 티어 5 개발자의 경우 약 100개의 동시 세션으로 속도가 제한되며, 티어 1~4의 경우 더 낮은 제한이 적용됩니다. 대규모 배포를 지원하기 위해 시간이 지남에 따라 이러한 제한을 늘릴 예정입니다.
- **공식 SDK 지원**: 실시간 API에 대한 지원을 OpenAI Python 및 Node.js SDK에 통합할 예정입니다.
- **프롬프트 캐싱**: 이전 대화 턴을 할인된 가격으로 재처리할 수 있도록 [프롬프트 캐싱](http://platform.openai.com/docs/guides/prompt-caching) 지원을 추가할 예정입니다.
- **확장된 모델 지원**: 실시간 API는 향후 해당 모델의 버전에서 GPT-4o mini도 지원할 예정입니다.

개발자들이 이러한 새로운 기능을 활용하여 교육부터 번역, 고객 서비스, 접근성 등 다양한 사용 사례에서 사용자를 위한 매력적인 새 오디오 경험을 어떻게 만들어낼지 기대가 됩니다.

---

## Links
- [🤖 OpenAI의 보이스모드가 API로 공개되었습니다.](https://www.linkedin.com/posts/kojunseo_openai-platform-activity-7247384461179322368-mrMT?utm_source=share&utm_medium=member_desktop)