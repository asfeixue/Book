# Netty
* MsgPack
* Google ProtoBuf
* jboss-marshalling

## MsgPack
### 依赖
```
compile group: 'org.msgpack', name: 'msgpack', version: '0.6.12'
```

### 编码器
```
public class MsgpackEncoder extends MessageToByteEncoder<Object> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Object msg, ByteBuf out) throws Exception {
        MessagePack messagePack = new MessagePack();
        byte[] raw = messagePack.write(msg);
        out.writeBytes(raw);
    }
}
```

### 解码器
```
public class MsgPackDecoder extends MessageToMessageDecoder<ByteBuf> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf msg, List<Object> out) throws Exception {
        final byte[]  array;
        final  int length = msg.readableBytes();
        array = new byte[length];
        msg.getBytes(msg.readerIndex(), array, 0, length);
        MessagePack messagePack = new MessagePack();
        out.add(messagePack.read(array));
    }
}
```

### 传输对象
```
@Message
public class UserInfo implements Serializable {
    private int age;

    private String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "UserInfo{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```
@Message注解是一定需要带上的，否则框架不会发起传输。

## Google ProtoBuf
使用需要安装protobuf，IDE有相应插件支持，idea中需要安装Protobuf support plugin。
gradle支持的配置如下：
### 依赖
```
apply plugin: "com.google.protobuf"

buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'com.google.protobuf:protobuf-gradle-plugin:0.8.1'
    }
}
dependencies {
    ......
    compile group: 'com.google.protobuf', name: 'protobuf-java', version: '3.3.1'
}

sourceSets {
    main {
     proto {
            srcDir 'src/main/proto'
        }
        java {
            srcDir 'src/main/java'
        }
    }
}

```

### 编码器 & 解码器
Netty自带的ProtobufDecoder，ProtobufEncoder可以提供支持，故此处的编码为：
```
ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
ch.pipeline().addLast(new ProtobufDecoder(Person.getDefaultInstance()));
ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
ch.pipeline().addLast(new ProtobufEncoder());
ch.pipeline().addLast(new 自定义Handler());
```

### 传输对象
```
syntax="proto3";

option java_package = "com.feixue.mbridge.monitor.protobuf";
option java_multiple_files = true;
option java_outer_classname = "PersonDO";

message Person {
    int32 id = 1;
    string name = 2;
    string email = 3;
}
```

## jboss-marshalling
### 依赖
```
compile group: 'org.jboss.marshalling', name: 'jboss-marshalling', version: '1.4.11.Final'
compile group: 'org.jboss.marshalling', name: 'jboss-marshalling-serial', version: '1.4.11.Final'
```
### 编解码
```
public class MarshallingCodeCFactory {

    /**
     * 创建解码器
     * @return
     */
    public static MarshallingDecoder buildMarshallingDecoder() {
        MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
        MarshallingConfiguration configuration = new MarshallingConfiguration();
        configuration.setVersion(5);
        UnmarshallerProvider provider = new DefaultUnmarshallerProvider(marshallerFactory, configuration);

        MarshallingDecoder decoder = new MarshallingDecoder(provider, 1024);

        return decoder;
    }

    /**
     * 创建编码器
     * @return
     */
    public static MarshallingEncoder buildMarshallingEncoder() {
        MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
        MarshallingConfiguration configuration = new MarshallingConfiguration();
        configuration.setVersion(5);
        MarshallerProvider provider = new DefaultMarshallerProvider(marshallerFactory, configuration);

        MarshallingEncoder encoder = new MarshallingEncoder(provider);

        return encoder;
    }
}
```
如果classpath没有“jboss-marshalling-serial”，会创建factory失败，但是居然没异常，应该是在某个环节被吃掉了。用这个需要完善整个上下文才能正常使用。
### 传输对象
```
public class MarshallingDO implements Serializable {
    private static final long serialVersionUID = -2263135845181737943L;

    private int age;

    private String name;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
需要实现Serializable，否则无法正常工作。