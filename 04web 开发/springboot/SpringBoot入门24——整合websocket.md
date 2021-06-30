# Springboot+WebSocket

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    /**
     * 配置消息代理-中介
     * @param config
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        //服务端推送给客户端连接的前缀
        config.enableSimpleBroker("/topic");
        //客户端发送给服务端连接的前缀
        config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/websocket").setAllowedOriginPatterns("*").withSockJS();
//
    }

}


@RestController
public class GameInfoController {


    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
        // simulated delay
        Thread.sleep(1000);
        return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
    }

}

```

## @SendTo和 SimpMessagingTemplate的区别

@SendTo注解和SimpMessagingTemplate的区别

- 1、SendTo 不通用，固定发送给指定的订阅者
- 2、SimpMessagingTemplate 灵活，支持多种发送方式