Websocket

遇到如下异常：
```
Mixed Content: The page at 'https://xxxxxxxx.net/#/xxxx' was loaded over HTTPS, but attempted to connect to the insecure WebSocket endpoint 'ws://xxxxxxxx:443/xxxx/ws?token=68442701-ed69-4337-aa1a-d213056e776f'. This request has been blocked; this endpoint must be available over WSS.
```

因为应用链接是基于https协议的，那么ws也需要更换为wss，修改后兼容正常。