Exception

java.lang.IllegalArgumentException: URI is not hierarchical
当

    File file = new File(url.toURI());
时，内部会执行：
    
    public File(URI uri) {
    // Check our many preconditions
    if (!uri.isAbsolute())
        throw new IllegalArgumentException("URI is not absolute");
    if (uri.isOpaque())
        throw new IllegalArgumentException("URI is not hierarchical");
其中uri.isOpaque()的逻辑如下：

    /**
     * Tells whether or not this URI is opaque.
     *
     * <p> A URI is opaque if, and only if, it is absolute and its
     * scheme-specific part does not begin with a slash character ('/').
     * An opaque URI has a scheme, a scheme-specific part, and possibly
     * a fragment; all other components are undefined. </p>
     *
     * @return  {@code true} if, and only if, this URI is opaque
     */
    public boolean isOpaque() {
        return path == null;
    }

URI语法：[scheme:] scheme-specific-part [#fragment]
URI分不透明URI和分层URI。
不透明URI：不透明的URI指scheme-specific-part不是以正斜杠（/）开头的绝对的URI。不透明的URI并不是用于分解的。不透明的URI与其它的URI不同，它不服从标准化、分解和相对化。
分层URI：分层的URI可以是以正斜杠开头的绝对的URI或相对的URL。scheme-specific-part的语法：[//authority] [path] [?query]。

从classpath中读取出来的文件路径为透明的。