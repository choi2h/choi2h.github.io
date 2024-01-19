---
layout: post
title: "[SpringBoot]채팅서버 만들기(1)_WebSocket"
author: hwa
categories: Spring
tags: Spring, Java, WebSocket
---

서브프로젝트를 진행하던 중 서로간의 대화가 가능해야 한다는 요구조건 개발을 할 때가 되었습니다.  
이 기회에 맨날 생각만하던..채팅서버를 구축 해보고자 합니다.

채팅을 하려면 전송자와 수신자 간의 지속적인 통신이 이어져야 합니다.  
저희는 기존 웹 서버 개발 시 HTTP 프로토콜을 사용하여 통신을 합니다.   
그러나 HTTP는 클라이언트가 요청을 해야만 서버로부터 응답을 받을 수 있습니다.

지금 만들고자 하는 채팅서버에서수신자는 요청을 하지 않고도 전송자가 전달한 메시지를 받을 수 있어야 합니다. 즉, HTTP는 클라이언트가 연결을 만드는 프로토콜이라 서버에서 클라이언트로 임의 시점에 메시지를 보내는 데는 쉽게 쓰일 수 없습니다.

### 어떤 프로토콜을 사용할 것인가?
서버가 연결을 만드는 것처럼 여러 방법이 제안되어 왔는데 그 방법들에 대해 알아보겠습니다.

#### Polling (폴링)
폴링은 클라이언트가 서버로 주기적인 요청을 보내는 방법입니다.
가장 단순한 방법이지만 지속성을 위해서는 짧은 주기로 요청을 보내게 되며 만약 클라이언트의 수가 많아진다면 서버의 부담이 증가할 것입니다. HTTP 요청으로 매 요청 시 마다 Handshake를 진행해야 하며 요청 시 전달해 줄 메시지가 없다면 무의미한 통신이 되므로 불필요한 자원이 낭비되는 것과 같습니다.
![Polling flow](https://github.com/choi2h/choi2h.github.io/assets/84762486/1f072948-2824-420e-a4aa-908080b273c5)





#### Long Polling(롱폴링)
기존의 폴링방법을 보안하고자 나온 것이 롱폴링입니다.
롱폴링도 폴링과 마찬가지로 클라이언트가 서버에 요청을 아래와 같은 경우에 응답합니다.
- 설정되어 있는 타임아웃이 지났을 경우 
- 새로운 메시지가 존재할 경우 메시지 반환
![long polling flow](https://github.com/choi2h/choi2h.github.io/assets/84762486/ae00270e-c8a9-4f1c-aefa-07eacbeea8f2)


그러나 폴링과 마찬가지로 메시지가 없더라도 주기적으로 요청을 보내야 하는 단점이 있습니다.
또한, 서버측에서는 대기시간동안 클라이언트의 연결 여부를 알기가 어렵습니다. 클라이언트가 응답을 기다리는 도중 연결을 해제할 수도 있다는 것입니다.


#### WebSocket
웹 소켓은 처음 클라이언트가 서버에 요청을 하고 한번 맺어진 연결은 끊어지지 않고 지속됩니다. 연결된 상태에서 서버는 언제든지 클라이언트에게 메시지를 전달할 수 있습니다. 기존 HTTP/HTTPS와 동일한 80/443 포트를 사용하여 기존 방화벽 환경에서도 잘 동작할 수 있습니다.
![WebSocket flow](https://github.com/choi2h/choi2h.github.io/assets/84762486/23bb1d83-a6a1-449e-aa73-b1ad5a3e2c83)
웹소켓은 지속적인 연결이 있기 때문에 클라이언트의 연결 관리를 효율적으로 해야 합니다.  

##### WebSocket의 연결 과정
- 프로토콜 요청은 `ws://`로 시작됩니다.

처음 클라이언트에서 HTTP 요청 시 아래와 같이 `Upgrade` 헤더를 사용하여 WebSocket 프로토콜로 전환을 요청합니다.
``` YAML
GET /chat HTTP/1.1
Connection: Upgrade
Host: localhost:8000
Origin: http://localhost:8000
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
Sec-WebSocket-Key: LBUHHGTTBcwLasdKMLpxOw==
Sec-WebSocket-Version: 13
Upgrade: websocket
```

응답시에는 `200 OK` 대신 `101 Switching Protocols` 로 응답이 오며 요청과 비슷한 형태의 헤더로 응답합니다.
> 상태코드 `101 Switching Protocols`는 Upgrade 헤더 필드에 명시된 프로토콜로 교환됩니다.
``` YAML
HTTP/1.1 101 Switching Protocols
Connection: upgrade
Date: Thu, 18 Jan 2024 06:08:02 GMT
Sec-WebSocket-Accept: T70T5gVS/7prwiF0RrsX4cuYv9w=
Sec-WebSocket-Extensions: permessage-deflate;client_max_window_bits=15
Upgrade: websocket
```

### WebSocket을 이용한 채팅 구현하기

1. build.gradle에 웹소켓 의존성을 추가합니다.
	```
	implementation 'org.springframework.boot:spring-boot-starter-websocket'
	```

2. WebSocketHandler 
	`WebSocketHandler`는 아래와 같은 구조를 가지고 있습니다. 추상 클래스를 구현한 핸들러는 두가지가 있는데, 바이너리 형태의 메시지를 지원하는 `BinaryWebSocketHandler` 와 텍스트 형태의 메시지를 지원하는 `TextWebSocketHandler`가 있습니다.  
	![WebSocketHandler](https://github.com/choi2h/choi2h.github.io/assets/84762486/b088be63-8398-4928-969a-15baabf79413)

	BinaryWebSocketHandler 내부를 확인해보면 handleTextMessage 메서드 요청 시 옳지 않은 데이터 타입의 요청으로 `NOT_ACCEPTABLE`상태로 세션이 종료되는 것을 확인할 수 있습니다.  
    ![BinaryWebSocketHandler](https://github.com/choi2h/choi2h.github.io/assets/84762486/738a8163-1ee7-46bf-a5d9-98c89d573763)
    
	반대로, TextWebSocketHandler에서는 handleBinaryMessage 메서드 요청 시에 `NOT_ACCEPTABLE`상태로 세션이 종료되는 것을 확인할 수 있습니다.
	![TextWebSocketHandler](https://github.com/choi2h/choi2h.github.io/assets/84762486/6b22dca9-f1d7-4960-ad16-7e193a4ad210)

	저희는 단순한 텍스트 형태의 메시지를 전달할 것이므로 `TextWebSocketHandler`를 상속받아 구현해보겠습니다.
	``` java
	@Slf4j  
	@Component  
	public class ChatHandler extends TextWebSocketHandler {  
	  
	    private static List<WebSocketSession> list = new ArrayList<>();  
	
	    @Override  
	    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {  
	        String payload = message.getPayload();  
	        log.debug("[ChatHandler] payload : {}", payload);  
	  
	        for(WebSocketSession webSocketSession : list) {  
	            webSocketSession.sendMessage(message);  
	        }  
	    }  
	  
	    /*클라이언트가 접속 시 호출되는 메서드*/  
	  
	    @Override  
	    public void afterConnectionEstablished(WebSocketSession session) throws Exception {  
	        list.add(session);  
	  
	        log.info("[ChatHandler] 클라이언트 접속 : {} ", session);  
	    }  
	  
	    /*클라이언트가 접속 해제 시 호출되는 메서드*/  
	    @Override  
	    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {  
	        log.info("[ChatHandler] 클라이언트 접속 해제 : {}", session);  
	        list.remove(session);  
	    }  
	}
	```


3. WebSocketConfig 
	``` java
	@Configuration  
	@RequiredArgsConstructor  
	@EnableWebSocket  
	public class WebSocketConfig implements WebSocketConfigurer {  
	  
	    private final ChatHandler chatHandler;  
	    @Override    
	    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {        
	    registry.addHandler(chatHandler, "ws/chat")                .setAllowedOrigins("http://*:8000", "http://*.*.*.*:8000")                .withSockJS()                .setClientLibraryUrl("http://localhost:8000/myapp/js/sock-client/js");    
		}
	}
	```

4. ChatController
	```java
	@Slf4j  
	@Controller  
	public class ChatController {  
	  
	    @GetMapping("/chat")  
	    public String chatGet(Model model) {  
	        log.info("ChatController, chat GET()");  

			// 접속한 회원의 이름을 UUID로 생성해준다.
	        model.addAttribute("name", UUID.randomUUID().toString());  
	        return "chat";  
	    }  
	}
	```
</br>

위 4가지를 구현하면 웹 소켓을 테스트해볼 수 있습니다.
 
#### HTML / JS
``` HTML
<!DOCTYPE html>  
<html lang="en" xmlns:th="http://www.thymeleaf.org">  
  
<head>  
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">  
    <title>hello</title>  
</head>  
  
  
<body>  
<div class="container">  
  
    <div class="col-6">  
        <label><b>채팅방</b></label>  
    </div>  
  
    <div>  
        <div id="msgArea" class="col"></div>  
  
        <div class="col-6">  
            <div class="input-group mb-3">  
                <input type="text" id="msg" class="form-control" aria-label="Recipient's username"  
                       aria-describedby="button-addon2">  
  
                <div class="input-group-append">  
                    <button class="btn btn-outline-secondary" type="button" id="button-send">전송</button>  
                </div>  
  
            </div>  
        </div>  
    </div>  
</div>  
  
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>  
<script src="https://cdn.jsdelivr.net/sockjs/1/sockjs.min.js"></script>  
<script th:inline="javascript">  
    $(document).ready(function () {  
  
        const username = [[${name}]];  
  
        $("#disconn").on("click", (e) => {  
            disconnect();  
        })  
  
        $("#button-send").on("click", (e) => {  
            send();  
        });  
  
        // var websocket = new WebSocket("ws://localhost:8000/ws/chat");  
        var sockJs = new SockJS("/ws/chat", {transports: ["websocket", "xhr-streaming", "xhr-polling"]});  
        var stomp = webstomp  
        sockJs.onmessage = onMessage;  
        sockJs.onopen = onOpen;  
        sockJs.onclose = onClose;  
  
        function send() {  
  
            let msg = document.getElementById("msg");  
  
            console.log(username + ":" + msg.value);  
            sockJs.send(username + ":" + msg.value);  
            msg.value = '';  
        }  
  
        //채팅창에서 나갔을 때  
        function onClose(evt) {  
            var str = username + ": 님이 방을 나가셨습니다.";  
            sockJs.send(str);  
        }  
  
        //채팅창에 들어왔을 때  
        function onOpen(evt) {  
            var str = username + ": 님이 입장하셨습니다.";  
            sockJs.send(str);  
        }  
  
        function onMessage(msg) {  
            var data = msg.data;  
            var sessionId = null;  
            //데이터를 보낸 사람  
            var message = null;  
            var arr = data.split(":");  
  
            for (var i = 0; i < arr.length; i++) {  
                console.log('arr[' + i + ']: ' + arr[i]);  
            }  
  
            var cur_session = username;  
  
            //현재 세션에 로그인 한 사람  
            console.log("cur_session : " + cur_session);  
            sessionId = arr[0];  
            message = arr[1];  
  
            console.log("sessionID : " + sessionId);  
            console.log("cur_session : " + cur_session);  
  
            //로그인 한 클라이언트와 타 클라이언트를 분류하기 위함  
            if (sessionId == cur_session) {  
                var str = "<div class='col-6'>";  
                str += "<div class='alert alert-secondary'>";  
                str += "<b>" + sessionId + " : " + message + "</b>";  
                str += "</div></div>";  
                $("#msgArea").append(str);  
            } else {  
                var str = "<div class='col-6'>";  
                str += "<div class='alert alert-warning'>";  
                str += "<b>" + sessionId + " : " + message + "</b>";  
                str += "</div></div>";  
                $("#msgArea").append(str);  
            }  
        }  
    })  
</script>  
  
</body>  
  
</html>

```




위 HTML/JS까지 작성 후 실행하면 아래와 같은 결과를 확인할 수 있습니다!

<img width="817" alt="image" src="https://github.com/choi2h/choi2h.github.io/assets/84762486/7fec5a28-6a55-4a9b-8fbf-db91f91037f0">



#### 참고  
가상면접 사례로 배우는 대규모 시스템 설계 기초 - 알렉스 쉬 지음  
https://docs.spring.io/spring-framework/reference/web/websocket.html  
https://dev-gorany.tistory.com/212  
https://junroot.github.io/programming/WebSocket-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0/#1-websocket-handshake  
https://velog.io/@koseungbin/WebSocket  
https://sdardew-valley.tistory.com/130  
https://heowc.dev/en/2021/07/15/spring-websocket-with-stomp/  