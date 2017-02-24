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
中，必须加上setAllowedOrigins("*")，允许跨的域可以自定义，否则会提示403错误。因为spring4会默认加上OriginHandshakeInterceptor的拦截器，基于同源策略，如果跨域，可能会失败。
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
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
        System.err.println(session.getId() + " is close.");
    }
}
```

js端并不处理ping,pong消息，这块直接由浏览器托管了。后端定时主动发起ping消息，浏览器接收到之后，会即刻返回pong消息，故具体的心跳保活就交由后端来处理。
```
/**
 * 心跳保活
 * @param session
 */
private void keepAlive(WebSocketSession session) {
    executors.submit(() -> {
        while (session.isOpen()) {
            try {
                Thread.sleep(TIME_ALIVE_INTERVAL);
                byte[] bs = new byte[1];
                bs[0] = 'alive';
                ByteBuffer byteBuffer = ByteBuffer.wrap(bs);
                session.sendMessage(new PingMessage(byteBuffer));
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    });
}
```