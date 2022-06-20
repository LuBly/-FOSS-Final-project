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
### Source Code 실행하기
#### Package 설치
```node
npm i
```
#### 실행
```node
npm start
```

### 실행 화면

#### 방 입장화면, 
localhost:3000/test 페이지에서 실행되는 화면입니다.

![image](https://user-images.githubusercontent.com/48556414/174543703-d8e6e977-68e7-4553-b4ad-c2a6cee1fb28.png)

#### 닉네임 및 room name을 입력하면 
source code src/public/js/main.js
```javascript
function handleRoomSubmit(event) {
     event.preventDefault();
     const input = welcome.querySelector("#room input");
     socket.emit("enter_room", input.value, showBeforeStartRoom);
     roomName = input.value;
     input.value = "";
}
```
input data와 실행하고자 하는 함수를 backend server로 송신합니다.

```javascript
socket.on("enter_room", (roomName, done) => {
 if ( countRoom(roomName) > 9 ){
     socket.emit("message specific user", socket.id, "정원초과로 입장하실 수 없습니다.😥");
 } else {
     socket.join(roomName);
     console.log(socket.rooms);
     done(roomName, countRoom(roomName));
     sockets.push(socket);
     users.set(socket.data.nickname, 0);	
     socket.to(roomName).emit("welcome", socket.data.nickname, roomName, countRoom(roomName));
 }
});
```
해당 event call이 이뤄질 때, backend 서버에서 socket event를 수신하여 해당 함수를 수행합니다.

#### 게임방 대기화면
본인 이외의 누군가 입장할 때 위의
```javascript
socket.to(roomName).emit("welcome", socket.data.nickname, roomName, countRoom(roomName));
```
함수를 통해 해당 방의 인원들에게 입장 메세지를 전송

![image](https://user-images.githubusercontent.com/48556414/174546179-1ead1e5d-8af5-4047-8390-9ee38261914b.png)

각 플레이어들이 준비 버튼을 클릭할 때 웹에서 event 송신
```javascript
function handleReadySubmit(event) {
     event.preventDefault();
     socket.emit("ready", roomName);
}
``` 
이후 서버에서 해당 event 수신 이후 방 내부의 모든 인원이 준비되었다 판단될 때 "ready" event를 송신
```javascript
socket.on("ready", (roomName) => {
     if (!readyStorage.includes(socket.id)) {
          readyStorage.push(socket.id);
     } else {
          readyStorage = readyStorage.filter((element) => {
          return element != socket.id
     });
     }

     if (readyStorage.length === wsServer.sockets.adapter.rooms.get(roomName)?.size) {
          wsServer.sockets.emit("ready");
     }
}); 
```
 모두 준비 완료인 상태일 때 해당 방의 상태를 게임 시작 가능한 상태(flag=1)로 변환하고 console에 "completely ready!!!" log 작성
 
 ![image](https://user-images.githubusercontent.com/48556414/174547088-f3c61956-00b1-4e97-8279-b365103ea5b5.png)
 
 > 모두 준비가 완료된 상태
 
 ```javascript
socket.on("ready", () => {
	    readyToStart();
});

function readyToStart() {
// 모든 ready가 끝났을 때 호출된다.
     console.log("completely ready!!!!!");
	    flag = 1;
}
```
### 게임 시작
- 게임 시작시 좌 상단의 타이머가 동시에 시작되고 타이머 시간 이내에 ox를 선택할 수 있다.

![image](https://user-images.githubusercontent.com/48556414/174547768-44186f1d-2c17-43ad-bbd7-1263cb536984.png)
> 게임이 시작되면 client는 server에게 "gameStart"와 "question" event를 송신
```javascript
function handleGameStart(event) {
     event.preventDefault();
	    if (flag) {
          socket.emit("gameStart", roomName);
          socket.emit("question", roomName, showQuestion);
     }
}
```
> server는 "gameStart"와 "question" event를 수신 후 해당 내용 처리
```javascript
socket.on("gameStart", (roomName) => {
     if (readyStorage.length === countRoom(roomName)){
        usersList = JSON.stringify(Array.from(users));
        wsServer.sockets.in(roomName).emit("scoreboard display", usersList);
        socket.emit("showGameRoom");
        socket.to(roomName).emit("showGameRoom");
     } else {
         socket.emit("message ready", socket.id, "참여자 모두 준비를 눌러주세요🙊");
         setTimeout(function() {  
              socket.emit("remove message");
         }, 1000);
     }
});
```

```javascript
socket.on("question", (roomName, done) => {
     let cnt = 0;

     for (let i = 0; i < question.length; i++) // Question 문제를 다 전송했는지 확인
     {
          if (arr[i] === 1) {
               cnt++;
          }
          if (cnt >= question.length) {
               return; 
          }
     }

     let index = Math.floor(Math.random() * 10);

     while(arr[index]) {
          index = Math.floor(Math.random() * 10);
     }		

     arr[index] = 1;

     answer = question[index].oxAnswer;
     explanation = question[index].explanation;

     socket.emit("round", question[index].oxQuestion, index);
     socket.to(roomName).emit("round", question[index].oxQuestion, index);
     socket.emit("timer"); //문제가 출제됨과 동시에 client의 timer 송신
     socket.to(roomName).emit("timer"); //문제가 출제됨과 동시에 같은 방의 room user client의 timer 송신
});
```


- 타이머가 종료될 때 선택한 ox에 따라 해당 user의 점수가 변동된다.
- 변동된 점수에 따라 우상단의 점수판이 바뀐다.
> 게임 시작 시 server에서 client에게 "scoreboard" event 송신
```javascript
socket.on("gameStart", (roomName) => {
     if (readyStorage.length === countRoom(roomName)){
        usersList = JSON.stringify(Array.from(users));
        wsServer.sockets.in(roomName).emit("scoreboard display", usersList);
        socket.emit("showGameRoom");
        socket.to(roomName).emit("showGameRoom");
     } else {
         socket.emit("message ready", socket.id, "참여자 모두 준비를 눌러주세요🙊");
         setTimeout(function() {  
              socket.emit("remove message");
         }, 1000);
     }
});
```

> client 수신내용
```javascript
socket.on("scoreboard display", (users) => {
     const scoreList = gameStart.querySelector("ul");
     newMap = new Map(JSON.parse(users));
     console.log(newMap);
     newMap.forEach((value, key) => { //이 부분을 통해 html의 점수판 부분 변경
          const li = document.createElement("li");
          li.innerText = `${key}: ${value}`;
          scoreList.append(li);
     });
});
```


 
## 결론
이 처럼 우리는 실시간 플레이에 필요한 다양한 event들과 이에 해당하는 함수들을 우리의 필요에 맞게 제작하고 사용할 수 있다.
client와 server뿐만 아니라, client와 client, server와 server사이에서의 송수신을 활용할 수 있고
이를 통해 보다 다양한 기능들을 구현할 수 있었다.

