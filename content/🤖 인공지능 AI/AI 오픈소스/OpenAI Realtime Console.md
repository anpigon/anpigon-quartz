---
created: 2024-10-06 08:23:21
modified: 2024-10-06 08:54:29
title: OpenAI Realtime Console
tags:
  - open-ai
  - Realtime
  - 오픈소스
permlink: 20241005t235345078z
category: kr
dg-publish: true
---

> [!summary]
> 이 **OpenAI Realtime Console** 프로젝트는 OpenAI Realtime API를 검사하고 디버깅하는 데 도움이 되는 도구입니다. 또한 Realtime API를 사용하는 클라이언트 애플리케이션을 개발하는 데 도움이 되는 참조 클라이언트와 오디오 관리 라이브러리를 제공합니다.

> 원문: https://github.com/openai/openai-realtime-console/

[OpenAI Realtime Console](https://github.com/openai/openai-realtime-console)은 OpenAI Realtime API를 위한 검사기 및 인터랙티브 API 참고 자료로 설계되었습니다. 이 프로젝트는 두 개의 유틸리티 라이브러리와 함께 제공되며, 하나는 브라우저와 Node.js용 **참조 클라이언트 역할**을 하는 [openai/openai-realtime-api-beta](https://github.com/openai/openai-realtime-api-beta)이고, 다른 하나는 브라우저에서 간단한 오디오 관리를 가능하게 하는 [`/src/lib/wavtools`](https://github.com/openai/openai-realtime-console/blob/main/src/lib/wavtools)입니다.

![](https://github.com/openai/openai-realtime-console/raw/main/readme/realtime-console-demo.png)

## 시작하기

리포지토리를 클론하고 패키지를 설치합니다.

```sh
$ git clone org-14957082@github.com:openai/openai-realtime-console.git
$ cd openai-realtime-console
$ npm i
```

서버 시작하기

```sh
$ npm start
```

브라우저를 열고 `localhost:3000`에 접속합니다.

## 사용하기

Realtime API에 액세스할 수 있는 OpenAI API 키가 필요합니다. 시작 시 입력하라는 메시지가 표시됩니다. 이 키는 `localStorage`를 통해 저장되며, UI에서 언제든지 변경할 수 있습니다.

세션을 시작하려면 **연결(connect)** 해야 합니다. 이를 위해서는 마이크 액세스가 필요합니다. 그런 다음 **manual**(Push-to-talk) 및 **vad**(음성 활동 감지) 대화 모드 중 하나를 선택하고 언제든지 전환할 수 있습니다.

두 가지 기능이 활성화되어 있습니다;

- `get_weather`: 어디서든 날씨를 물어보면 모델이 최선을 다해 위치를 파악하고 지도에 표시하며 해당 위치의 날씨를 가져옵니다. 위치 액세스 권한이 없으며 모델의 학습 데이터에서 좌표를 '추측'하므로 정확도가 완벽하지 않을 수 있다는 점에 유의하세요.
- `set_memory`: 모델에 정보를 기억하도록 요청하면 모델이 왼쪽의 JSON 블롭에 정보를 저장합니다.

푸시 투 토크(Push-to-talk) 또는 VAD 모드에서 언제든지 모델을 자유롭게 중단할 수 있습니다.

## 릴레이 서버 사용하기

보다 강력한 구현을 구축하고 자체 서버를 사용하여 참조 클라이언트를 사용해보고 싶다면 Node.js [릴레이 서버](https://github.com/openai/openai-realtime-console/blob/main/relay-server/index.js)를 포함했습니다.

```shell
$ npm run relay
```

`localhost:8081`에서 자동으로 시작됩니다.

**다음 구성으로 `.env` 파일**을 만들어야 합니다:

```
OPENAI_API_KEY=YOUR_API_KEY
REACT_APP_LOCAL_RELAY_SERVER_URL=http://localhost:8081
```

`.env.` 변경 사항을 적용하려면 React 앱과 릴레이 서버를 모두 재시작해야 합니다. 로컬 서버 URL은 [`ConsolePage.tsx`](https://github.com/openai/openai-realtime-console/blob/main/src/pages/ConsolePage.tsx)를 통해 로드됩니다. 언제든지 릴레이 서버 사용을 중지하려면 환경 변수를 삭제하거나 빈 문자열로 설정하면 됩니다.

```js
/**
 * 로컬 릴레이 서버를 실행하면 
 * API 키를 숨기고 서버에서 사용자 지정 로직을 실행할 수 있습니다.
 *
 * 로컬 릴레이 서버 주소를 다음과 같이 설정합니다:
 * REACT_APP_LOCAL_RELAY_SERVER_URL=http://localhost:8081
 *
 * `.env` 파일에 OPENAI_API_KEY=를 설정해야 합니다.
 * `npm run relay`를 `npm start`과 병렬로 실행할 수 있습니다.
 */
const LOCAL_RELAY_SERVER_URL: string =
  process.env.REACT_APP_LOCAL_RELAY_SERVER_URL || '';
```

이 서버는 **단순 메시지 중계** 용도로만 사용되지만 확장할 수 있습니다:

- 온라인으로 플레이할 앱을 출시하려는 경우 API 자격 증명 숨기기
- 서버에서 비밀로 유지하려는 특정 호출(예: `instructions`)을 직접 처리합니다.
- 클라이언트가 수신 및 전송할 수 있는 이벤트 유형 제한하기

이러한 기능은 직접 구현해야 합니다.

## 실시간 API 레퍼런스 클라이언트

최신 레퍼런스 클라이언트 및 문서는 GitHub에서 [openai/openai-realtime-api-beta](https://github.com/openai/openai-realtime-api-beta)에서 확인할 수 있습니다.

이 클라이언트는 모든 React(프론트엔드) 또는 Node.js 프로젝트에서 직접 사용할 수 있습니다. 전체 문서는 GitHub 리포지토리를 참조하세요. 하지만 여기 가이드를 시작하기 위한 입문서로 사용할 수 있습니다.

```js
import { RealtimeClient } from '/src/lib/realtime-api-beta/index.js';

const client = new RealtimeClient({ apiKey: process.env.OPENAI_API_KEY });

// 연결에 앞서 매개변수 설정 가능
client.updateSession({ instructions: 'You are a great, upbeat friend.' });
client.updateSession({ voice: 'alloy' });
client.updateSession({ turn_detection: 'server_vad' });
client.updateSession({ input_audio_transcription: { model: 'whisper-1' } });

// 이벤트 처리 설정
client.on('conversation.updated', ({ item, delta }) => {
  const items = client.conversation.getItems(); // 이것을 사용하여 모든 항목을 렌더링할 수 있습니다.
  /* 대화에 대한 모든 변경 사항을 포함하며, 델타(delta)가 채워질 수 있습니다. */
});

// Realtime API에 연결
await client.connect();

// 아이템 전송 및 생성 트리거
client.sendUserMessageContent([{ type: 'text', text: `How are you?` }]);
```

### 스트리밍 오디오 보내기

스트리밍 오디오를 보내려면 `.appendInputAudio()` 메서드를 사용합니다. `turn_detection: 'disabled'` 모드인 경우 `.generate()`를 사용하여 모델에 응답하도록 지시해야 합니다.

```js
// 사용자 오디오 전송, Int16Array 또는 ArrayBuffer여야 합니다.
// 기본 오디오 포맷은 24,000Hz 샘플 속도의 pcm16입니다.
// 0.1초 단위로 1초 분량의 노이즈를 채웁니다.
for (let i = 0; i < 10; i++) {
  const data = new Int16Array(2400);
  for (let n = 0; n < 2400; n++) {
    const value = Math.floor((Math.random() * 2 - 1) * 0x8000);
    data[n] = value;
  }
  client.appendInputAudio(data);
}

// 보류 중인 오디오가 커밋되고 모델에 다음을 생성하도록 요청합니다.
client.createResponse();
```

### 도구 추가 및 사용

도구 작업은 간단합니다. `.addTool()`을 호출하고 콜백을 두 번째 파라미터로 설정하기만 하면 됩니다. 콜백은 도구의 매개변수와 함께 실행되고 결과는 자동으로 모델로 다시 전송됩니다.

```js
// 콜백을 지정하여 도구를 추가할 수도 있습니다.
client.addTool(
  {
    name: 'get_weather',
    description:
      'Retrieves the weather for a given lat, lng coordinate pair. Specify a label for the location.',
    parameters: {
      type: 'object',
      properties: {
        lat: {
          type: 'number',
          description: 'Latitude',
        },
        lng: {
          type: 'number',
          description: 'Longitude',
        },
        location: {
          type: 'string',
          description: 'Name of the location',
        },
      },
      required: ['lat', 'lng', 'location'],
    },
  },
  async ({ lat, lng, location }) => {
    const result = await fetch(
      `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lng}&current=temperature_2m,wind_speed_10m`
    );
    const json = await result.json();
    return json;
  }
);
```

### 모델 중단하기

특히 `turn_detection: 'disabled'` 모드에서 모델을 수동으로 중단하고 싶을 수 있습니다. 이를 위해 다음을 사용할 수 있습니다:

```js
// id는 현재 생성 중인 항목의 ID입니다. 
// sampleCount는 청취자가 들은 오디오 샘플의 수입니다.
client.cancelResponse(id, sampleCount);
```

이 메서드를 사용하면 모델이 즉시 생성을 중단할 뿐만 아니라`sampleCount` 이후의 모든 오디오를 제거하고 텍스트 응답을 지움으로써 재생 중인 항목도 잘립니다. 이 메서드를 사용하면 모델을 중단하고 사용자 상태보다 앞서 생성된 모든 것을 '기억'하지 못하게 할 수 있습니다.

### 참조 클라이언트 이벤트

`RealtimeClient`에는 애플리케이션 제어 흐름을 위한 다섯 가지 주요 클라이언트 이벤트가 있습니다. 이것은 클라이언트 사용에 대한 개요일 뿐이며, 전체 실시간 API 이벤트 사양은 상당히 방대하므로 더 많은 제어가 필요한 경우 GitHub 리포지토리를 확인하세요: [openai/openai-realtime-api-beta](https://github.com/openai/openai-realtime-api-beta).

```js
// 연결 실패와 같은 오류
client.on('error', (event) => {
  /* do something */
});

// VAD 모드에서 사용자가 말하기 시작합니다.
// 필요한 경우 이를 사용하여 이전 응답의 오디오 재생을 중지할 수 있습니다.
client.on('conversation.interrupted', () => {
  /* do something */
});

// 대화에 대한 모든 변경 사항을 포함합니다.
// 델타(delta)가 채워질 수 있습니다.
client.on('conversation.updated', ({ item, delta }) => {
  // get all items, e.g. if you need to update a chat window
  const items = client.conversation.getItems();
  switch (item.type) {
    case 'message':
      // system, user, or assistant message (item.role)
      break;
    case 'function_call':
      // always a function call from the model
      break;
    case 'function_call_output':
      // always a response from the user / application
      break;
  }
  if (delta) {
    // 특정 이벤트에 대해 다음 중 하나만 채워집니다.
    // delta.audio = Int16Array, audio added
    // delta.transcript = string, transcript added
    // delta.arguments = string, function arguments added
  }
});

// 대화에 항목이 추가된 후에만 트리거됩니다.
client.on('conversation.item.appended', ({ item }) => {
  /* item 상태는 'in_progress' 또는 'completed' 일 수 있습니다. */
});

// 대화에서 항목이 완료된 후에만 트리거됨
// 항상 conversation.item.appended 이후에 트리거됩니다.
client.on('conversation.item.completed', ({ item }) => {
  /* item 상태는 항상 'completed'입니다. */
});
```

## Wavtools

Wavtools에는 브라우저에서 PCM16 오디오 스트림을 쉽게 관리하고 녹음 및 재생할 수 있는 기능이 포함되어 있습니다.

### WavRecorder 퀵스타트

```js
import { WavRecorder } from '/src/lib/wavtools/index.js';

const wavRecorder = new WavRecorder({ sampleRate: 24000 });
wavRecorder.getStatus(); // "ended"

// 권한 요청, 마이크 연결
await wavRecorder.begin();
wavRecorder.getStatus(); // "paused"

// 녹음 시작
// 이 콜백은 기본적으로 8192개의 샘플 청크로 트리거됩니다.
// { mono, raw }는 Int16Array(PCM16) 모노 및 전체 채널 데이터입니다.
await wavRecorder.record((data) => {
  const { mono, raw } = data;
});
wavRecorder.getStatus(); // "recording"

// 녹음 중지
await wavRecorder.pause();
wavRecorder.getStatus(); // "paused"

// "audio/wav" 오디오 파일 출력
const audio = await wavRecorder.save();

// 현재 오디오 버퍼를 지우고 녹음을 시작합니다.
await wavRecorder.clear();
await wavRecorder.record();

// 시각화를 위한 데이터 가져오기
const frequencyData = wavRecorder.getFrequencies();

// 녹음 중지, 마이크, 출력 파일 연결 해제
await wavRecorder.pause();
const finalAudio = await wavRecorder.end();

// 장치 변경을 수신합니다(예: 누군가 마이크를 분리하는 경우) 
// deviceList는 MediaDeviceInfo[] + `default` 속성의 배열입니다.
wavRecorder.listenForDeviceChange((deviceList) => {});
```

### WavStreamPlayer 퀵스타트

```js
import { WavStreamPlayer } from '/src/lib/wavtools/index.js';

const wavStreamPlayer = new WavStreamPlayer({ sampleRate: 24000 });

// 오디오 출력에 연결
await wavStreamPlayer.connect();

// 1초 분량의 빈 PCM16 오디오 생성
const audio = new Int16Array(24000);

// 3초 분량의 오디오를 대기열에 추가하면 즉시 재생이 시작됩니다.
wavStreamPlayer.add16BitPCM(audio, 'my-track');
wavStreamPlayer.add16BitPCM(audio, 'my-track');
wavStreamPlayer.add16BitPCM(audio, 'my-track');

// 시각화를 위한 데이터 가져오기
const frequencyData = wavStreamPlayer.getFrequencies();

// 언제든지 오디오를 중단(재생 중지)할 수 있습니다.
// 다시 시작하려면 .add16BitPCM()을 다시 호출해야 합니다.
const trackOffset = await wavStreamPlayer.interrupt();
trackOffset.trackId; // "my-track"
trackOffset.offset; // sample number
trackOffset.currentTime; // time in track
```