# Exception

## java.lang.IllegalArgumentException: URI is not hierarchical
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

## java.io.IOException: invalid constant type: 18
```
Caused by: java.lang.RuntimeException: java.io.IOException: invalid constant type: 18
    at javassist.CtClassType.getClassFile2(CtClassType.java:204)
    at javassist.CtClassType.subtypeOf(CtClassType.java:304)
    at javassist.CtClassType.subtypeOf(CtClassType.java:319)
    at javassist.compiler.MemberResolver.compareSignature(MemberResolver.java:248)
    at javassist.compiler.MemberResolver.lookupMethod(MemberResolver.java:120)
    at javassist.compiler.MemberResolver.lookupMethod(MemberResolver.java:97)
    at javassist.compiler.TypeChecker.atMethodCallCore(TypeChecker.java:711)
    at javassist.compiler.TypeChecker.atCallExpr(TypeChecker.java:688)
    at javassist.compiler.JvstTypeChecker.atCallExpr(JvstTypeChecker.java:157)
    at javassist.compiler.ast.CallExpr.accept(CallExpr.java:46)
    at javassist.compiler.JvstTypeChecker.atCastToWrapper(JvstTypeChecker.java:126)
    at javassist.compiler.JvstTypeChecker.atCastExpr(JvstTypeChecker.java:98)
    at javassist.compiler.ast.CastExpr.accept(CastExpr.java:55)
    at javassist.compiler.CodeGen.doTypeCheck(CodeGen.java:242)
    at javassist.compiler.CodeGen.compileExpr(CodeGen.java:229)
    at javassist.compiler.CodeGen.atReturnStmnt2(CodeGen.java:598)
    at javassist.compiler.JvstCodeGen.atReturnStmnt(JvstCodeGen.java:425)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:363)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.CodeGen.atIfStmnt(CodeGen.java:391)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:355)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.MemberCodeGen.atTryStmnt(MemberCodeGen.java:204)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:367)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.CodeGen.atStmnt(CodeGen.java:351)
    at javassist.compiler.ast.Stmnt.accept(Stmnt.java:50)
    at javassist.compiler.CodeGen.atMethodBody(CodeGen.java:292)
    at javassist.compiler.CodeGen.atMethodDecl(CodeGen.java:274)
    at javassist.compiler.ast.MethodDecl.accept(MethodDecl.java:44)
    at javassist.compiler.Javac.compileMethod(Javac.java:169)
    at javassist.compiler.Javac.compile(Javac.java:95)
    at javassist.CtNewMethod.make(CtNewMethod.java:74)
    at javassist.CtNewMethod.make(CtNewMethod.java:45)
    at com.alibaba.dubbo.common.bytecode.ClassGenerator.toClass(ClassGenerator.java:318)
    at com.alibaba.dubbo.common.bytecode.Wrapper.makeWrapper(Wrapper.java:346)
    at com.alibaba.dubbo.common.bytecode.Wrapper.getWrapper(Wrapper.java:89)
    at com.alibaba.dubbo.config.ReferenceConfig.init(ReferenceConfig.java:269)
    at com.alibaba.dubbo.config.ReferenceConfig.get(ReferenceConfig.java:138)
    at com.alibaba.dubbo.config.spring.ReferenceBean.getObject(ReferenceBean.java:65)
    at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:144)
    ... 130 more
Caused by: java.io.IOException: invalid constant type: 18
    at javassist.bytecode.ConstPool.readOne(ConstPool.java:1113)
    at javassist.bytecode.ConstPool.read(ConstPool.java:1056)
    at javassist.bytecode.ConstPool.<init>(ConstPool.java:150)
    at javassist.bytecode.ClassFile.read(ClassFile.java:765)
    at javassist.bytecode.ClassFile.<init>(ClassFile.java:109)
    at javassist.CtClassType.getClassFile2(CtClassType.java:191)
    ... 174 more
```
dubbo进程启动报如上错误堆栈。
项目jdk版本是8，而javassist-3.15.0-GA不支持jdk8.有两种办法解决：
1.项目jdk版本降级；
2.排除dubbo依赖的javassist版本，升级到javassist-3.18.0-GA。