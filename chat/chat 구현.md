# 웹소켓에 대하여(Web Socket)
두 프로그램간 메세지 교환을 위한 통신 방법<br/>
<br/>
서버와 클라이언트의 연결을 유지하고 양방향 통신 또는 데이터 전송 가능하게  한다.<br/>
<br/>
실시간 서비스에 주로 사용된다.<br/>
<br/>

### Web Socket 사용 이유
실시간 채팅앱의 요구사항을 반영하기 위해 클라이언트에서 매번 HTTP프로토콜 서버에 요청하는 것은 비효율적이다.<br/>
<br/>
Ajax 통신으로 보완할 수 있지만 결국 HTTP를 사용하는 것은 여전하다. 웹소켓을 통해 이러한 점을 해결할 수 있다.<br/>

### Web Socket 과 HTTP의 특징 및 차이점
- 웹소켓은 HTTP를 통해 최초로 연결되어 일정시간 후 자동으로 끊어지고 WerSocket connction은 유지된다.
- HTTP와 달리 웹소켓은 stateful 프로토콜이다.
- HTTP는 stateless하기 때문에 변경사항 시 클라이언트에서 요청하지 않으면 적용되지 않지만,
  웹소켓은 지속적으로 Connection을 유지해 실시간 변경사항이 적용된다.
- 웹소켓은 HTTP와 동일한 port(80)을 사용한다.
- HTTP는 Request/response기반의 stateless 이기 때문에 서버와 클라이언트 간의 Socket connection같은 영구적 연결이 불가하고
  클라이언트에서 Request 할때만 서버가 Response 방식으로 진행되는 한방향 통신이다. 이 경우 Refrash하지 않으면 데이터가 업데이트 되지 않는 문제가 발생한다.
  <br/>
  대체 방법으로Polling, Long Polling, Streaming 등이 있으나 HTTP를 통해 통신하기 때문에 Request, Response 둘 다 Header가 불필요하게 크다.
- 웹소켓은 stateful protocol이기 때문에 클라이언트와 한 번 연결이 되면 계속 같은 라인을 통해 통신하기 때문에 HTTP와 같이 불필요한 TCP연결 트래픽을 피할 수 있다.
- 또 웹소켓은 HTTP와 같은 포트를 사용하기 때문에 기업용 어플리케이션에 적용할 때 방화벽은 재설정하지 않아도 된다.
<br/>

### Web Socket 사용 시 주의점
- 웹소켓은 stateful 프로토콜로 connection을 항상 유지해야하지 때문에 트래픽이 많은 경우 서버에 부담이 될 수 있다.
- 연결이 계속 유지되어야 하기 때문에 연결이 끊어졌을 때 적절히 대응할 수 있어야 한다.

# Spring Boot WetSocket 서버 구축
### build.gradle에 라이브러리 추가
 ```
  implementation 'org.springframework.boot:spring-boot-starter-websocket'
 ```

<br/>
<br/>
<br/>

# WebSocket 접속과정
<p align="center">
  <img src="https://github.com/Jurioh0603/study/assets/148063470/80329643-012d-469f-ab89-9c5503c90cbf">
</p>

# 채팅 메세지 구현
채팅 메세지를 주고 받기 위한 DTO를 생성.<br/>
입장, 대화 두가지 상황이 있으므로 enum 선언
 ```
  import lombok.*;

@Builder
@Getter
@Setter
@RequiredArgsConstructor
@AllArgsConstructor
public class ChatMessage {
    private MessageType type;
    private String roomId;
    private String sender;
    private String message;

    public enum MessageType {
        ENTER, TALK
    }
}
 ```
> enum 이란?
> 제한된 값, 상수들의 목록을 갖는 타입
> 멀티 스레드 환경에서 불변(immutable)을 보장하는 싱글톤 인스턴스로서 가장 많이 사용하는 방법 중 하나
> 장점
> >Type Safety - 타입 안정성, 특정 범위의 값만 사용 가능하므로 컴파일 에러를 먼저 보여줘 런타임 예외를 줄임
> >Readability - 가독성, 값들이 명시적으로 정의되어있음
> >Maintainability - 관리의 용이, 값이 추가 또는 변경 될 때 한 곳에서만 변경하면 되기 때문에 유지보수에 용이
> >Performance - 컴파일 타임에 정적인 값으로 변환되기 때문에 실행시간에서 상수 검색의 오버헤드를 줄임
> >직렬화(싱글톤 보장) - 인스턴스가 JVM내에 하나만 존재한다는 것이 100% 보장됨

<br/>
<br/>
# 채팅방 구현
DTO 생성<br/><br>
채팅방은 입장한 클라이언트의 정보를 가지고 있어야 하므로 WebSocketSession정보 리스트를 갖는다. 그리고 필요한 필드를 선언<br/>
<br>
handleAction 메서드의 If문으로 입장과 대화를 분기 처리<br>
<br>
sendMessage 메서드로 해당 채팅방에 있는 session에 메세지를 전송한다.<br>
<br>
findAllRoom의 응답에 sessions가 들어가면 오류가 나므로 @JsonIgnore를 추가했다.<br>
<br>
 ```
import com.ichubtou.websocket.service.ChatService;
import lombok.Builder;
import lombok.Getter;
import org.springframework.web.socket.WebSocketSession;

import java.util.HashSet;
import java.util.Set;

@Getter
public class ChatRoom {
    private String roomId;
    private String name;
		@JsonIgnore
    private Set<WebSocketSession> sessions = new HashSet<>();

    @Builder
    public ChatRoom(String roomId, String name) {
        this.roomId = roomId;
        this.name = name;
    }

    public void handleActions(WebSocketSession session, ChatMessage chatMessage, ChatService chatService) {
        if (chatMessage.getType().equals(ChatMessage.MessageType.ENTER)) {
            sessions.add(session);
            chatMessage.setMessage(chatMessage.getSender() + "님이 입장했습니다.");
        }
        sendMessage(chatMessage, chatService);
    }

    public <T> void sendMessage(T message, ChatService chatService) {
        sessions.parallelStream().forEach(session -> chatService.sendMessage(session, message));
    }
}
 ```

# 채팅서비스 구현

생성, 조회, 하나의 세션에 메세지를 전송하는 기능을 가진 채팅 서비스를 생성한다.
 ```
import com.fasterxml.jackson.databind.ObjectMapper;
import com.ichubtou.websocket.entity.ChatRoom;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;

import javax.annotation.PostConstruct;
import java.io.IOException;
import java.util.*;

@Slf4j
@RequiredArgsConstructor
@Service
public class ChatService {

    private final ObjectMapper objectMapper;
    private Map<String, ChatRoom> chatRooms;

    @PostConstruct
    private void init() {
        chatRooms = new LinkedHashMap<>();
    }

    public List<ChatRoom> findAllRoom() {
        return new ArrayList<>(chatRooms.values());
    }

    public ChatRoom findRoomById(String roomId) {
        return chatRooms.get(roomId);
    }

    public ChatRoom createRoom(String name) {
        String randomId = UUID.randomUUID().toString();
        ChatRoom chatRoom = ChatRoom.builder()
                .roomId(randomId)
                .name(name)
                .build();
        chatRooms.put(randomId, chatRoom);
        return chatRoom;
    }

    public <T> void sendMessage(WebSocketSession session, T message) {
        try {
            session.sendMessage(new TextMessage(objectMapper.writeValueAsString(message)));
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        }
    }
}
 ```

# 채팅 컨트롤러 구현
채팅방의 생성 및 조회는 Rest API로 구현한다.
 ```
import com.ichubtou.websocket.entity.ChatRoom;
import com.ichubtou.websocket.service.ChatService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RequiredArgsConstructor
@RestController
@RequestMapping("/chat")
public class ChatController {

    private final ChatService chatService;

    @PostMapping
    public ChatRoom createRoom(@RequestParam String name) {
        return chatService.createRoom(name);
    }

    @GetMapping
    public List<ChatRoom> findAllRoom() {
        return chatService.findAllRoom();
    }
}
 ```

# WebSocket Handler 구현
socket 통신은 서버와 클라이언트가 1:N으로 관계를 맺는다. 따라서 한 서버에 여러 클라이언트가 접속할 수 있으며, 
서버에는 여러 클라이언트가 발송한 메세지를 받아 처리해줄 Handler의 작성이 필요하다.
<br><br>
TextWebSocketHandler를 상속받아 Handler를 작성해준다.
<br><br>
Client로부터 받은 메세지를 Console Log에 출력하고 입장했을때 입장 메세지를 출력한다.
 ```
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.ichubtou.websocket.service.ChatService;
import lombok.Builder;
import lombok.Getter;
import org.springframework.web.socket.WebSocketSession;

import java.util.HashSet;
import java.util.Set;

@Getter
public class ChatRoom {
    private String roomId;
    private String name;
    @JsonIgnore
    private Set<WebSocketSession> sessions = new HashSet<>();

    @Builder
    public ChatRoom(String roomId, String name) {
        this.roomId = roomId;
        this.name = name;
    }

    public void handleActions(WebSocketSession session, ChatMessage chatMessage, ChatService chatService) {
        if (chatMessage.getType().equals(ChatMessage.MessageType.ENTER)) {
            sessions.add(session);
            chatMessage.setMessage(chatMessage.getSender() + "님이 입장했습니다.");
        }
        sendMessage(chatMessage, chatService);
    }

    public <T> void sendMessage(T message, ChatService chatService) {
        sessions.parallelStream().forEach(session -> chatService.sendMessage(session, message));
    }
}
 ```


## 참고사이트
<https://velog.io/@ichubtou/Spring-Boot-Web-Socket-%EA%B0%84%EB%8B%A8%ED%95%9C-%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%8A%A5-%EA%B5%AC%ED%98%84>
<https://velog.io/@mooh2jj/Java-Enum%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0>
