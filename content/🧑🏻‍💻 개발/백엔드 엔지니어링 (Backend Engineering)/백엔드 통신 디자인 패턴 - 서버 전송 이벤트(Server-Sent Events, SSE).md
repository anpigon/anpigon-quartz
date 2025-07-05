---
created: 2024-09-25 10:13:11
modified: 2024-09-25 10:40:13
dg-publish: true
---

## SSE의 개념과 작동 방식

1. SSE는 HTTP 프로토콜을 사용하여 서버에서 클라이언트로 실시간 데이터를 전송하는 기술입니다.
2. 클라이언트가 한 번 요청을 보내면, 서버는 지속적으로 데이터를 스트리밍 형태로 보냅니다.
3. 서버 응답은 끝나지 않고 계속되며, 클라이언트는 스트리밍 데이터를 파싱하여 개별 이벤트로 처리합니다.

## SSE의 장단점

### 장점

- 실시간성: 서버로부터 즉시 정보를 받을 수 있습니다.
- HTTP 호환성: 대부분의 HTTP 서버에서 지원됩니다.
- 간단한 구현: 웹소켓에 비해 구현이 쉽습니다.

### 단점

- 클라이언트 온라인 필요: 클라이언트가 항상 연결되어 있어야 합니다.
- 메시지 처리 문제: 클라이언트가 모든 메시지를 처리하지 못할 수 있습니다.
- HTTP/1.1 연결 제한: 브라우저에서 도메인당 연결 수 제한이 있을 수 있습니다. (브라우저당 6개)

## SSE 구현 예시

### 서버 측

- 특별한 `Content-Type` 헤더(text/event-stream)로 응답합니다.
- 데이터를 "data:" 접두사와 함께 전송하고, 두 개의 줄바꿈(`\n\n`)으로 이벤트를 구분합니다.

```js
const app = require("express")();

app.get("/stream", (req, res) => {
    res.setHeader("Content-Type", "text/event-stream");
    streamResponse(res);
})

let i = 0;
function streamResponse(res) {
    res.write("data: " + `message from server ---- [${i++}]\n\n`);
    setTimeout(() => streamResponse(res), 1000);
}

app.listen(8080)
```

### 클라이언트 측

- `EventSource` 객체를 사용하여 서버에 연결합니다.
- `onmessage` 이벤트 핸들러를 통해 서버로부터 오는 메시지를 처리합니다.

```js
let sse = new EventSource("http://localhost:8080/stream");

sse.onmessage = (msg) => console.log(msg)
```

## 주의사항

- HTTP/1.1의 연결 제한으로 인한 잠재적 문제가 있을 수 있습니다.
- HTTP/2는 이러한 제한을 해결할 수 있는 더 나은 접근 방식을 제공합니다.


