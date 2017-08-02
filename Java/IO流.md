IO流

# 流关闭
Java本身是有独立的GC的，一个对象未存在引用了，就会被垃圾回收掉。但是如果这个对象持有了系统资源，比如文件句柄（文件流），网络端口（网络流）等情况下，垃圾回收无法释放，此时就需要主动执行关闭操作。

在Java7以下，是需要自己调用close()方法来执行关闭的。在Java7及以上，可以如下：
```
try (BufferedReader br = new BufferedReader(new FileReader(path)) {
    return br.readLine();
}
```
前提是关闭的对象必须实现java.io.Closeable接口。

# copy Jar file to Desktop

```
InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("explain/css/bootstrap.min.css");
    FileUtils.copyToFile(inputStream, new File("./" + path));
```
通过相对路径jar内的相对路径定位，然后直接获取流。不要通过URL中转。因为读取转换为File时，处理的URL会提示：java.lang.IllegalArgumentException: URI is not hierarchical。