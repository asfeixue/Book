Websocket

#client side
基于Angular 1.x的前端功能，选择了angular-websocket.js作为支持库。
具体文档如下：
https://github.com/AngularClass/angular-websocket

#server side
服务端选择spring4。具体实现代码如下：
配置websocket支持如下：
```
@Service
@EnableWebSocket
public class WebsocketConfig extends WebMvcConfigurerAdapter implements WebSocketConfigurer {

    @Resource
    private KeyboxWebSocketHandler handler;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(handler, "/xxx/ws").addInterceptors(new HandShake()).setAllowedOrigins("*");
    }
}
```
在
```
registry.addHandler(handler, "/xxx/ws").addInterceptors(new HandShake()).setAllowedOrigins("*");
```
中，必须加上setAllowedOrigins("*")，否则会提示403错误。因为spring4会默认加上OriginHandshakeInterceptor的拦截器，基于同源策略，如果跨域，可能会失败。
```
public class HandShake implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {

    }
}
```
```
@Service
public class XXXWebSocketHandler implements WebSocketHandler {
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        System.err.println(session.getId() + " has connection!");
    }

    @Override
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
        System.err.println(message.getPayload().toString());
    }

    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        if (session.isOpen()) {
            session.close();
        }
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
        System.err.println(session.getId() + " is close.");
    }

    @Override
    public boolean supportsPartialMessages() {
        return false;
    }
}
```