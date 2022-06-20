# -FOSS-Final-project

## Socket.io

## Socket.io란?
![image](https://user-images.githubusercontent.com/48556414/174516525-cd584061-3899-402f-954a-442cfb618da8.png)
socket.io란 실시간으로 상호작용하는 웹 서비스를 만드는 기술인 웹소켓을 쉽게 사용할 수 있게 해주는 framework입니다.
### 그래서 WebSocket이랑 뭐가 다른가요?

WebSocket
- HTML5 웹 표준 기술
- 매우 빠르게 작동하며 통신할 때 아주 적은 데이터를 이용함
- 이벤트를 단순히 듣고, 보내는 것만 가능함

Socket.io
- 표준 기술이 아니며, 라이브러리임
- 소켓 연결 실패 시 fallback을 통해 다른 방식으로 알아서 해당 클라이언트와 연결을 시도함
- 방 개념을 이용해 일부 클라이언트에게만 데이터를 전송하는 브로드캐스팅이 가능함

### 왜 Socket.io 인가?
기본적인 websocket의 기능을 제공함과 동시에, room이라는 기능을 이용해 여러 개의 방을 만들 수 있고, 
소켓에 연결된 전체 클라이언트에게 broadcast를 보낼 수 있다거나,
room별로 broadcast를 보낼 수 있다.

## Socket.io 이벤트 통신
클라이언트에서 발생하는 이벤트를 개발자 임의로 설정할 수 있다.
이벤트는 문자열로 지정하며 직접 이벤트를 발생시킬 수 있다.
```javascript
// 해당 이벤트를 받고 콜백함수를 실행
socket.on('받을 이벤트 명', (msg) => {
})
 
// 이벤트 명을 지정하고 메세지를 보낸다.
socket.emit('전송할 이벤트 명', msg)
```
이런식으로 메세지 마다 고유한 이벤트를 등록해 구별해서 송수신하면,
다양한 통신 기능을 구현할 수 있다.

다음은 구현해본 ox 퀴즈 중 하나의 function이다.
```javascript
socket.on("ready check", ({ roomName }) => {
    if ((readyStorage.get(roomName)).length === wsServer.sockets.adapter.rooms.get(roomName)?.size) {
      //console.log(true);
      wsServer.sockets.emit("ready", true);
    }
});
```
## Socket.io 송수신
### Socket.io 수신

```javascript
// 접속된 모든 클라이언트에게 메시지를 전송한다
io.emit('event_name', msg);
 
// 메시지를 전송한 클라이언트에게만 메시지를 전송한다
socket.emit('event_name', msg);
 
// 메시지를 전송한 클라이언트를 제외한 모든 클라이언트에게 메시지를 전송한다
socket.broadcast.emit('event_name', msg);
 
// 특정 클라이언트에게만 메시지를 전송한다
io.to(id).emit('event_name', data);
```

### Socket.io 송신

```javascript
// 클라이언트와 소켓IO 연결됬는지 안됬는지 이벤트 실행. (채팅방에 누가 입장하였습니다/퇴장하였습니다 )
io.on('connection/disconnection', (socket) => {
});
 
// 클라이언트에서 지정한 이벤트가 emit되면 수신 발생
socket.on('event_name', (data) => {
});
```

## Socket.io 시작하기
### Socket.io 설치하기
```node
npm i socket.io
```


