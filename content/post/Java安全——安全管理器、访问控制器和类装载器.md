---
title: "Java安全——安全管理器、访问控制器和类装载器"
date: 2017-07-31T21:02:11+08:00
subtitle: ""
tags: ["java"]
---

<!--more-->

## 安全管理器：SecurityManager
安全管理器在Java语言中的作用就是检查操作是否有权限执行。是Java沙箱的基础组件。我们一般所说的打开沙箱，也是加`-Djava.security.manager`选项。

其实日常的很多API都涉及到安全管理器，它的工作原理一般是：
 
1. 请求Java API 
2. Java API使用安全管理器判断许可权限 
3. 通过则顺序执行，否则抛出一个Exception。

比如在之前的“理解沙箱”这一章提到的，开启沙箱后，会限制文件访问，那这个代码是如何的呢？看下源码：

```java
public FileInputStream(File file) throws FileNotFoundException {
        String name = (file != null ? file.getPath() : null);
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkRead(name);
        }
        if (name == null) {
            throw new NullPointerException();
        }
        if (file.isInvalid()) {
            throw new FileNotFoundException("Invalid file path");
        }
        fd = new FileDescriptor();
        fd.attach(this);
        path = name;
        open(name);
    }
```
比较清晰的API会先去获取安全管理器，如果开启沙箱，则安全管理器不是空，检查checkRead(name)。而checkRead方法最内层的实现，其实利用了后面要说的访问控制器。

具体点，我们看下SecurityManager的主要方法列表：

```java
checkAccept(String, int)
checkAccess(Thread)
checkAccess(ThreadGroup)
checkAwtEventQueueAccess()
checkConnect(String, int)
checkConnect(String, int, Object)
checkCreateClassLoader()
checkDelete(String)
checkExec(String)
checkExit(int)
checkLink(String)
checkListen(int)
checkMemberAccess(Class<?>, int)
checkMulticast(InetAddress)
checkMulticast(InetAddress, byte)
checkPackageAccess(String)
checkPackageDefinition(String)
checkPermission(Permission)
checkPermission(Permission, Object)
checkPrintJobAccess()
checkPropertiesAccess()
checkPropertyAccess(String)
checkRead(FileDescriptor)
checkRead(String)
checkRead(String, Object)
checkSecurityAccess(String)
checkSetFactory()
checkSystemClipboardAccess()
checkTopLevelWindow(Object)
checkWrite(FileDescriptor)
checkWrite(String)
```
都是check方法，分别囊括了文件的读写删除和执行、网络的连接和监听、线程的访问、以及其他包括打印机剪贴板等系统功能。而这些check代码也基本横叉到了所有的核心Java API上。

安全管理器可以自定义，作为核心API调用的部分，我们可以自己为自己的业务定制安全管理逻辑。举个例子如下：

```java
public class SecurityManagerTest {
    static class MySM extends SecurityManager {
        public void checkExit(int status) {
            throw new SecurityException("no exit");
        }
    }
    public static void main(String[] args) {
        MySM sm = new MySM();
        System.out.println(System.getSecurityManager());
        System.setSecurityManager(sm);//注释掉测一下
        System.exit(0);
    }
}
```
注释掉代码中的注释行，系统打印null，然后正常退出。当我们打开注释，并且自己扩展一个SecurityManager——MySM，它做的事情很简单，就是覆盖了checkExit方法，在系统退出时抛出一个“no exit”的异常。再执行，结果变成了

```java
null
Exception in thread "main" java.lang.SecurityException: no exit
    at com.taobao.cd.security.SecurityManagerTest$MySM.checkExit(SecurityManagerTest.java:7)
    at java.lang.Runtime.exit(Runtime.java:107)
    at java.lang.System.exit(System.java:971)
    at com.taobao.cd.security.SecurityManagerTest.main(SecurityManagerTest.java:16)
```
显然，安全管理器生效了。

## 访问控制器：AccessController
揭开沙箱面纱，第一步是安全管理器，那么第二步就是访问控制器了。因为沙箱的所有check方法实现，都是基于`AccessController`的。

要了解`AccessController`，需要理解4个概念：代码源、权限、策略和保护域。

### 代码源CodeSource

CodeSource就是一个简单的类，用来声明从哪里加载类。

### 权限Permission

Permission类是AccessController处理的基本实体。Permission类本身是抽象的，它的一个实例代表一个具体的权限。权限有两个作用，一个是允许Java API完成对某些资源的访问。另一个是可以为自定义权限提供一个范本。权限包含了权限类型、权限名和一组权限操作。具体可以看看BasicPermission类的代码。典型的也可以参看FilePermission的实现。

### 策略Policy

策略是一组权限的总称，用于确定权限应该用于哪些代码源。话说回来，代码源标识了类的来源，权限声明了具体的限制。那么策略就是将二者联系起来，策略类Policy主要的方法就是`getPermissions(CodeSource)`和`refresh()`方法。Policy类在老版本中是abstract的，且这两个方法也是。在jdk1.8中已经不再有abstract方法。这两个方法也都有了默认实现。

在JVM中，任何情况下只能安装一个策略类的实例。安装策略类可以通过`Policy.setPolicy()`方法来进行，也可以通过`java.security`文件里的`policy.provider=sun.security.provider.PolicyFile`来进行。jdk1.6以后，Policy引入了PolicySpi，后续的扩展基于SPI进行。

### 保护域ProtectionDomain
保护域可以理解为代码源和相应权限的一个组合。表示指派给一个代码源的所有权限。看概念，感觉和策略很像，其实策略要比这个大一点，保护域是一个代码源的一组权限，而策略是所有的代码源对应的所有的权限的关系。

JVM中的每一个类都一定属于且仅属于一个保护域，这由ClassLoader在define class的时候决定。但不是每个ClassLoader都有相应的保护域，核心Java API的ClassLoader就没有指定保护域，可以理解为属于系统保护域。

### AccessController
了解了组成，再回头看AccessController。这是一个无法实例化的类——仅仅可以使用其static方法。AccessController最重要的方法就是`checkPermission()`方法，作用是基于已经安装的Policy对象，能否得到某个权限。

回到理解沙箱那一篇文章里的例子，FileInputStream的构造方法就利用SecurityManager来checkRead。而SecurityManager的checkRead方法则

```java
public void checkPermission(Permission perm) {
    java.security.AccessController.checkPermission(perm);
}
```

这样来检查权限。

然而，AccessController的使用还是重度关联类加载器的。如果都是一个类加载器且都从一个保护域加载类，那么你构造的checkPermission的方法将正常返回。

当使用了其他类加载器或者使用了Java扩展包时，这种情况比较普遍。AccessController另一个比较实用的功能是doPrivilege（授权）。假设一个保护域A有读文件的权限，另一个保护域B没有。那么通过`AccessController.doPrivileged`方法，可以将该权限临时授予B保护域的类。而这种授权是单向的。也就是说，它可以为调用它的代码授权，但是不能为它调用的代码授权。

## 类装载器ClassLoader
ClassLoader对安全模型有三方面的影响：

第一，可以结合JVM定义名称空间，以保护Java语言本身安全特性的完整性。

第二，在必要时调用SecurityManager保证代码在定义或者访问类时有适当的权限。

第三，建立了权限与类对象之间的映射，这样AccessController就知道哪些类拥有哪些权限了。而这可以绕过建立自定义Policy类，通过自定义ClassLoader并在其中定义类权限而实现。

先来说说名称空间，其实就是包名，但是不同的是，不同的ClassLoader可以装在相同包名的类，而这时，其实对于每个ClassLoader，有一个自己的名字空间。为啥这么干？显然啊，就不说包冲突这事了，从安全角度看，你冒名顶替个com.sun.xx咋办？肯定得按照ClassLoader来分。从不同网站加载的Applet类，就是不同的ClassLoader来做。

## 老生常谈类加载器
类加载器是个层次结构，最基础的是系统类加载器，下面有很多子类比如URLClassLoader。加载一个类时，以委托的形式逐层询问——总结来就一句话：父亲优先。爹能加载，爹先来，不行再由儿子上。一旦为一个域的类定义类加载器，那么其他域的类加载器的整个祖先链路上不包含对应域，也就隔离了彼此的类加载。

## 类装载器加载类时要做哪些事
总的来说，需要完成以下工作： 

1， 询问安全管理器是否允许访问当前处理的类。如果不行，抛一个安全异常。这一步可选，一般在loadClass()方法开始处实现。对应accessClassInPackage权限。 

2， 如果类装载器已经载入了此类，它将寻找以前定义的类对象，并返回该对象。这一步在loadClass()内部实现。 

3，否则，类装载器将询问其父亲，递归查看父类装载器是否知道如何载入此类。因此总会是系统类加载器最先加载，从而避免了核心Java API中的类被其他自定义的类冒充。这一步也在loadClass()里实现。 
2和3对应代码如下

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
4，询问安全管理器是否允许程序创建当前处理的类。如果不行，则抛出一个安全异常。这一步可选，如果实现，则需要在findClass()的开始处完成。这一步不是在操作开始时完成，而是在询问父类装载器之后进行。这一步对应为defineClassInPackage权限。 

5，向一个字节数组中读入类文件。读取文件以及创建字节数组的方式因类加载器不同而不同。在findClass()中完成。 

6，为该类创建合适的保护域。保护域可以来自默认安全模型（即从策略文件中得到），也可以由类加载器扩展。还有一种方法是可以创建一个代码源对象，并采用其保护域定义。这一步也在findClass()中完成。 

7，在findClass()方法中，通过调用defineClass()方法，可以由字节码构造一个Class对象。如果使用的是第6步中的代码源，则需要调用getPermissions()方法查找与代码源相关的权限。defineClass()方法还保证了字节码必须通过字节码校验器的检查。 

8，最后还需要解析该类。即它所直接引用的类也应由当前类加载器找到。只有直接引用的才算，作为实例变量、方法参数或局部变量来使用的类不算。这一步在loadClass()中完成。对应上面代码中的resovleClass()。

## 以URLClassLoader举例

1，对应的代码如下：

```java
public final Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        // First check if we have permission to access the package. This
        // should go away once we've added support for exported packages.
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            int i = name.lastIndexOf('.');
            if (i != -1) {
                sm.checkPackageAccess(name.substring(0, i));
            }
        }
        return super.loadClass(name, resolve);
    }
```

URLClassLoader的newInstance()方法会构造一个内部的工厂加载器类。这个类的loadClass()方法做了checkPackageAccess的事情。 

2，3 两步与超级父类ClassLoader相同。就是上面ClassLoader的loadClass()做的事情。 

4，5，6，7 四个步骤涉及到findClass()方法，URLClassLoader覆盖了findClass()方法，但是最新版的jdk，其实将这几个步骤做的事情都在defineClass()里做掉了。里面的逻辑实现如下：

```java
protected Class<?> findClass(final String name)
        throws ClassNotFoundException
    {
        final Class<?> result;
        try {
            result = AccessController.doPrivileged(
                new PrivilegedExceptionAction<Class<?>>() {
                    public Class<?> run() throws ClassNotFoundException {
                        String path = name.replace('.', '/').concat(".class");
                        Resource res = ucp.getResource(path, false);
                        if (res != null) {
                            try {
                                return defineClass(name, res);
                            } catch (IOException e) {
                                throw new ClassNotFoundException(name, e);
                            }
                        } else {
                            return null;
                        }
                    }
                }, acc);
        } catch (java.security.PrivilegedActionException pae) {
            throw (ClassNotFoundException) pae.getException();
        }
        if (result == null) {
            throw new ClassNotFoundException(name);
        }
        return result;
    }
```

defineClass()的逻辑：

```java
/*
     * Defines a Class using the class bytes obtained from the specified
     * Resource. The resulting Class must be resolved before it can be
     * used.
     */
    private Class<?> defineClass(String name, Resource res) throws IOException {
        long t0 = System.nanoTime();
        int i = name.lastIndexOf('.');
        URL url = res.getCodeSourceURL();
        if (i != -1) {
            String pkgname = name.substring(0, i);
            // Check if package already loaded.
            Manifest man = res.getManifest();
            definePackageInternal(pkgname, man, url);
        }
        // Now read the class bytes and define the class
        java.nio.ByteBuffer bb = res.getByteBuffer();
        if (bb != null) {
            // Use (direct) ByteBuffer:
            CodeSigner[] signers = res.getCodeSigners();
            CodeSource cs = new CodeSource(url, signers);
            sun.misc.PerfCounter.getReadClassBytesTime().addElapsedTimeFrom(t0);
            return defineClass(name, bb, cs);
        } else {
            byte[] b = res.getBytes();
            // must read certificates AFTER reading bytes.
            CodeSigner[] signers = res.getCodeSigners();
            CodeSource cs = new CodeSource(url, signers);
            sun.misc.PerfCounter.getReadClassBytesTime().addElapsedTimeFrom(t0);
            return defineClass(name, b, 0, b.length, cs);
        }
    }
```
而这里面所有的defineClass都在ClassLoader这个超级父类里做了实现。

## 总结

通常的Java的安全性，都是从类加载器、安全管理器和访问控制器之间的关系考虑的。一般来说类加载器的作用更重要。 

如果需要灵活的安全策略，往往要自定义类加载器。自定义类加载器允许在定义类时调整安全策略。这与实现一个新的Policy类相似。一般认为自定义类加载器会比修改一个Policy类要容易。



原文见： 

* [Java安全——安全管理器、访问控制器和类装载器](https://www.zybuluo.com/changedi/note/417132)
* [Java安全——理解Java沙箱](https://www.zybuluo.com/changedi/note/237999#fn:1)