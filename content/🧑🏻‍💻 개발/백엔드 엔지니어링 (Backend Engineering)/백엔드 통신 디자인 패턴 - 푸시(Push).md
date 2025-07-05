---
created: 2024-09-25 10:57:16
modified: 2024-09-25 11:02:57
dg-publish: true
---

## 푸시 모델 개요

- 푸시는 서버가 클라이언트에게 즉시 데이터를 전송하는 방식입니다.
- 실시간 알림, 채팅 앱 등에 적합한 모델입니다.
- 클라이언트는 요청을 보내지 않고도 서버로부터 데이터를 받을 수 있습니다.

## 푸시 모델의 장단점

### 장점
- 실시간 데이터 전송 가능
- 클라이언트의 요청 없이 서버에서 데이터 전송 가능

### 단점
- 클라이언트가 항상 온라인 상태여야 함
- 클라이언트의 처리 능력을 고려하지 않고 데이터를 전송할 수 있음
- 양방향 프로토콜 필요

## 웹소켓을 이용한 푸시 구현 예제

- Node.js와 웹소켓을 사용하여 간단한 채팅 애플리케이션 구현
- 서버는 연결된 모든 클라이언트에게 메시지를 브로드캐스트
- 클라이언트 연결/연결 해제 처리 필요

### 서버 측  코드

```js
const http = require("http");
const WebSocketServer = require("websocket").server;

let connections = [];

// HTTP 서버를 생성합니다.
const httpserver = http.createServer()

 // 모든 작업을 수행하기 위해 WebSocketServer 라이브러리에 httpserver 객체를 전달하면, 이 클래스가 req/res를 재정의합니다. 
const websocket = new WebSocketServer({
	"httpServer": httpserver 
});

//listen on the TCP socket
httpserver.listen(8080, () => {
	console.log("My server is listening on port 8080")
})


// 정상적인 웹소켓 요청이 오면, 연결을 받습니다. 
websocket.on("request", request => {
    const connection = request.accept(null, request.origin);
    connection.on("message", message => {
        // 메시지가 도착하면 모든 사람에게 메시지를 보냅니다.
        connections.forEach (c => c.send(`User${connection.socket.remotePort} says: ${message.utf8Data}`));
    });
    
    connections.push(connection);
    
    // 누군가 방금 연결되면 모든 사람에게 알려줍니다.
    connections.forEach (c => {
	    c.send(`User${connection.socket.remotePort} just connected.`)
    });
})
```

### 클라이언트 측 코드

```js
let ws = new WebSocket("ws://localhost:8080");

ws.onmessage = message => {
	console.log(`Received: ${message.data}`);
}

ws.send("Hello! I'm client")
```

## 주의사항

- 연결이 끊긴 클라이언트 처리 로직 필요
- 대규모 사용자를 대상으로 할 때는 확장성 고려 필요

