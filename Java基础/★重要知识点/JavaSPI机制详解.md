# JavaSPI机制详解

# Java SPI 机制详解
在面向对象的设计原则中，一般推荐模块之间基于接口编程，通常情况下调用方模块是不会感知到被调用方模块的内部具体实现。一旦代码里面涉及具体实现类，就违反了开闭原则。如果需要替换一种实现，就需要修改代码。

为了实现在模块装配的时候不用在程序里面动态指明，这就需要一种服务发现机制。Java SPI 就是提供了这样一个机制：**为某个接口寻找服务实现的机制。这有点类似 IoC 的思想，将装配的控制权移交到了程序之外。**

## SPI 介绍
### 何谓 SPI?
SPI 即 Service Provider Interface ，字面意思就是：“服务提供者的接口”，我的理解是：专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口。

SPI 将服务接口和具体的服务实现分离开来，将服务调用方和服务实现者解耦，能够提升程序的扩展性、可维护性。修改或者替换服务实现并不需要修改调用方。

很多框架都使用了 Java 的 SPI 机制，比如：Spring 框架、数据库加载驱动、日志接口、以及 Dubbo 的扩展实现等等。

![1732497563706-b37cc196-076f-47e0-9dcc-86704765e480.jpeg](./img/VXQyxLabAnxJ5HyB/1732497563706-b37cc196-076f-47e0-9dcc-86704765e480-823747.jpeg)

### SPI 和 API 有什么区别？
**那 SPI 和 API 有啥区别？**

说到 SPI 就不得不说一下 API 了，从广义上来说它们都属于接口，而且很容易混淆。下面先用一张图说明一下：

![1732497563824-b942c386-52b2-4418-96f0-16f685741e9f.png](./img/VXQyxLabAnxJ5HyB/1732497563824-b942c386-52b2-4418-96f0-16f685741e9f-280537.png)

一般模块之间都是通过通过接口进行通讯，那我们在服务调用方和服务实现方（也称服务提供者）之间引入一个“接口”。

当实现方提供了接口和实现，我们可以通过调用实现方的接口从而拥有实现方给我们提供的能力，这就是 API ，这种接口和实现都是放在实现方的。

当接口存在于调用方这边时，就是 SPI ，由接口调用方确定接口规则，然后由不同的厂商去根据这个规则对这个接口进行实现，从而提供服务。

举个通俗易懂的例子：公司 H 是一家科技公司，新设计了一款芯片，然后现在需要量产了，而市面上有好几家芯片制造业公司，这个时候，只要 H 公司指定好了这芯片生产的标准（定义好了接口标准），那么这些合作的芯片公司（服务提供者）就按照标准交付自家特色的芯片（提供不同方案的实现，但是给出来的结果是一样的）。

## 实战演示
SLF4J （Simple Logging Facade for Java）是 Java 的一个日志门面（接口），其具体实现有几种，比如：Logback、Log4j、Log4j2 等等，而且还可以切换，在切换日志具体实现的时候我们是不需要更改项目代码的，只需要在 Maven 依赖里面修改一些 pom 依赖就好了。

![1732497563967-a7fef905-e965-42f3-9b0a-d67a32f157b3.png](./img/VXQyxLabAnxJ5HyB/1732497563967-a7fef905-e965-42f3-9b0a-d67a32f157b3-018275.png)

这就是依赖 SPI 机制实现的，那我们接下来就实现一个简易版本的日志框架。

### Service Provider Interface
新建一个 Java 项目service-provider-interface目录结构如下：（注意直接新建 Java 项目就好了，不用新建 Maven 项目，Maven 项目会涉及到一些编译配置，如果有私服的话，直接 deploy 会比较方便，但是没有的话，在过程中可能会遇到一些奇怪的问题。）

```plain
│  service-provider-interface.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
└─src
    └─edu
        └─jiangxuan
            └─up
                └─spi
                        Logger.java
                        LoggerService.java
                        Main.class
```

新建Logger接口，这个就是 SPI ， 服务提供者接口，后面的服务提供者就要针对这个接口进行实现。

```plain
package edu.jiangxuan.up.spi;

public interface Logger {
    void info(String msg);
    void debug(String msg);
}
```

接下来就是LoggerService类，这个主要是为服务使用者（调用方）提供特定功能的。这个类也是实现 Java SPI 机制的关键所在，如果存在疑惑的话可以先往后面继续看。

```java
package edu.jiangxuan.up.spi;

import java.util.ArrayList;
import java.util.List;
import java.util.ServiceLoader;

public class LoggerService {
    private static final LoggerService SERVICE = new LoggerService();

    private final Logger logger;

    private final List<Logger> loggerList;

    private LoggerService() {
        ServiceLoader<Logger> loader = ServiceLoader.load(Logger.class);
        List<Logger> list = new ArrayList<>();
        for (Logger log : loader) {
            list.add(log);
        }
        // LoggerList 是所有 ServiceProvider
        loggerList = list;
        if (!list.isEmpty()) {
            // Logger 只取一个
            logger = list.get(0);
        } else {
            logger = null;
        }
    }

    public static LoggerService getService() {
        return SERVICE;
    }

    public void info(String msg) {
        if (logger == null) {
            System.out.println("info 中没有发现 Logger 服务提供者");
        } else {
            logger.info(msg);
        }
    }

    public void debug(String msg) {
        if (loggerList.isEmpty()) {
            System.out.println("debug 中没有发现 Logger 服务提供者");
        }
        loggerList.forEach(log -> log.debug(msg));
    }
}
```

新建Main类（服务使用者，调用方），启动程序查看结果。

```plain
package org.spi.service;

public class Main {
    public static void main(String[] args) {
        LoggerService service = LoggerService.getService();

        service.info("Hello SPI");
        service.debug("Hello SPI");
    }
}
```

程序结果：

info 中没有发现 Logger 服务提供者debug 中没有发现 Logger 服务提供者

此时我们只是空有接口，并没有为Logger接口提供任何的实现，所以输出结果中没有按照预期打印相应的结果。

你可以使用命令或者直接使用 IDEA 将整个程序直接打包成 jar 包。

### Service Provider
接下来新建一个项目用来实现Logger接口

新建项目service-provider目录结构如下：

```plain
│  service-provider.iml
│
├─.idea
│  │  .gitignore
│  │  misc.xml
│  │  modules.xml
│  └─ workspace.xml
│
├─lib
│      service-provider-interface.jar
|
└─src
    ├─edu
    │  └─jiangxuan
    │      └─up
    │          └─spi
    │              └─service
    │                      Logback.java
    │
    └─META-INF
        └─services
                edu.jiangxuan.up.spi.Logger
```

新建Logback类

```plain
package edu.jiangxuan.up.spi.service;

import edu.jiangxuan.up.spi.Logger;

public class Logback implements Logger {
    @Override
    public void info(String s) {
        System.out.println("Logback info 打印日志：" + s);
    }

    @Override
    public void debug(String s) {
        System.out.println("Logback debug 打印日志：" + s);
    }
}
```

将service-provider-interface的 jar 导入项目中。

新建 lib 目录，然后将 jar 包拷贝过来，再添加到项目中。

![1732497564063-0e07e0f7-f384-4760-857c-888bf0c7b6f4.png](./img/VXQyxLabAnxJ5HyB/1732497564063-0e07e0f7-f384-4760-857c-888bf0c7b6f4-027553.png)

再点击 OK 。

![1732497564132-2b6a3044-053a-4626-9277-2785e28c9d6d.png](./img/VXQyxLabAnxJ5HyB/1732497564132-2b6a3044-053a-4626-9277-2785e28c9d6d-459261.png)

接下来就可以在项目中导入 jar 包里面的一些类和方法了，就像 JDK 工具类导包一样的。

实现Logger接口，在src目录下新建META-INF/services文件夹，然后新建文件edu.jiangxuan.up.spi.Logger（SPI 的全类名），文件里面的内容是：edu.jiangxuan.up.spi.service.Logback（Logback 的全类名，即 SPI 的实现类的包名 + 类名）。

**这是 JDK SPI 机制 ServiceLoader 约定好的标准。**

这里先大概解释一下：Java 中的 SPI 机制就是在每次类加载的时候会先去找到 class 相对目录下的META-INF文件夹下的 services 文件夹下的文件，将这个文件夹下面的所有文件先加载到内存中，然后根据这些文件的文件名和里面的文件内容找到相应接口的具体实现类，找到实现类后就可以通过反射去生成对应的对象，保存在一个 list 列表里面，所以可以通过迭代或者遍历的方式拿到对应的实例对象，生成不同的实现。

所以会提出一些规范要求：文件名一定要是接口的全类名，然后里面的内容一定要是实现类的全类名，实现类可以有多个，直接换行就好了，多个实现类的时候，会一个一个的迭代加载。

接下来同样将service-provider项目打包成 jar 包，这个 jar 包就是服务提供方的实现。通常我们导入 maven 的 pom 依赖就有点类似这种，只不过我们现在没有将这个 jar 包发布到 maven 公共仓库中，所以在需要使用的地方只能手动的添加到项目中。

### 效果展示
为了更直观的展示效果，我这里再新建一个专门用来测试的工程项目：java-spi-test

然后先导入Logger的接口 jar 包，再导入具体的实现类的 jar 包。

![1732497564228-955209a8-dd21-4081-bbd1-6143aed50f3d.png](./img/VXQyxLabAnxJ5HyB/1732497564228-955209a8-dd21-4081-bbd1-6143aed50f3d-943387.png)

新建 Main 方法测试：

```plain
package edu.jiangxuan.up.service;

import edu.jiangxuan.up.spi.LoggerService;

public class TestJavaSPI {
    public static void main(String[] args) {
        LoggerService loggerService = LoggerService.getService();
        loggerService.info("你好");
        loggerService.debug("测试Java SPI 机制");
    }
}
```

运行结果如下：

Logback info 打印日志：你好Logback debug 打印日志：测试 Java SPI 机制

说明导入 jar 包中的实现类生效了。

如果我们不导入具体的实现类的 jar 包，那么此时程序运行的结果就会是：

info 中没有发现 Logger 服务提供者debug 中没有发现 Logger 服务提供者

通过使用 SPI 机制，可以看出服务（LoggerService）和 服务提供者两者之间的耦合度非常低，如果说我们想要换一种实现，那么其实只需要修改service-provider项目中针对Logger接口的具体实现就可以了，只需要换一个 jar 包即可，也可以有在一个项目里面有多个实现，这不就是 SLF4J 原理吗？

如果某一天需求变更了，此时需要将日志输出到消息队列，或者做一些别的操作，这个时候完全不需要更改 Logback 的实现，只需要新增一个服务实现（service-provider）可以通过在本项目里面新增实现也可以从外部引入新的服务实现 jar 包。我们可以在服务(LoggerService)中选择一个具体的 服务实现(service-provider) 来完成我们需要的操作。

那么接下来我们具体来说说 Java SPI 工作的重点原理——**ServiceLoader**。

## ServiceLoader
### ServiceLoader 具体实现
想要使用 Java 的 SPI 机制是需要依赖ServiceLoader来实现的，那么我们接下来看看ServiceLoader具体是怎么做的：

ServiceLoader是 JDK 提供的一个工具类， 位于package java.util;包下。

```plain
A facility to load implementations of a service.
```

这是 JDK 官方给的注释：**一种加载服务实现的工具。**

再往下看，我们发现这个类是一个final类型的，所以是不可被继承修改，同时它实现了Iterable接口。之所以实现了迭代器，是为了方便后续我们能够通过迭代的方式得到对应的服务实现。

```plain
public final class ServiceLoader<S> implements Iterable<S>{ xxx...}
```

可以看到一个熟悉的常量定义：

private static final String PREFIX = "META-INF/services/";

下面是load方法：可以发现load方法支持两种重载后的入参；

```plain
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader) {
    return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
```

根据代码的调用顺序，在reload()方法中是通过一个内部类LazyIterator实现的。先继续往下面看。

ServiceLoader实现了Iterable接口的方法后，具有了迭代的能力，在这个iterator方法被调用时，首先会在ServiceLoader的Provider缓存中进行查找，如果缓存中没有命中那么则在LazyIterator中进行查找。

```plain
public Iterator<S> iterator() {
    return new Iterator<S>() {

        Iterator<Map.Entry<String, S>> knownProviders
                = providers.entrySet().iterator();

        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext(); // 调用 LazyIterator
        }

        public S next() {
            if (knownProviders.hasNext())
                return knownProviders.next().getValue();
            return lookupIterator.next(); // 调用 LazyIterator
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    };
}
```

在调用LazyIterator时，具体实现如下：

```plain
public boolean hasNext() {
    if (acc == null) {
        return hasNextService();
    } else {
        PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
            public Boolean run() {
                return hasNextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private boolean hasNextService() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            //通过PREFIX（META-INF/services/）和类名 获取对应的配置文件，得到具体的实现类
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
    }
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}


public S next() {
    if (acc == null) {
        return nextService();
    } else {
        PrivilegedAction<S> action = new PrivilegedAction<S>() {
            public S run() {
                return nextService();
            }
        };
        return AccessController.doPrivileged(action, acc);
    }
}

private S nextService() {
    if (!hasNextService())
        throw new NoSuchElementException();
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
                "Provider " + cn + " not found");
    }
    if (!service.isAssignableFrom(c)) {
        fail(service,
                "Provider " + cn + " not a subtype");
    }
    try {
        S p = service.cast(c.newInstance());
        providers.put(cn, p);
        return p;
    } catch (Throwable x) {
        fail(service,
                "Provider " + cn + " could not be instantiated",
                x);
    }
    throw new Error();          // This cannot happen
}
```

可能很多人看这个会觉得有点复杂，没关系，我这边实现了一个简单的ServiceLoader的小模型，流程和原理都是保持一致的，可以先从自己实现一个简易版本的开始学：

### 自己实现一个 ServiceLoader
我先把代码贴出来：

```plain
package edu.jiangxuan.up.service;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.lang.reflect.Constructor;
import java.net.URL;
import java.net.URLConnection;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

public class MyServiceLoader<S> {

    // 对应的接口 Class 模板
    private final Class<S> service;

    // 对应实现类的 可以有多个，用 List 进行封装
    private final List<S> providers = new ArrayList<>();

    // 类加载器
    private final ClassLoader classLoader;

    // 暴露给外部使用的方法，通过调用这个方法可以开始加载自己定制的实现流程。
    public static <S> MyServiceLoader<S> load(Class<S> service) {
        return new MyServiceLoader<>(service);
    }

    // 构造方法私有化
    private MyServiceLoader(Class<S> service) {
        this.service = service;
        this.classLoader = Thread.currentThread().getContextClassLoader();
        doLoad();
    }

    // 关键方法，加载具体实现类的逻辑
    private void doLoad() {
        try {
            // 读取所有 jar 包里面 META-INF/services 包下面的文件，这个文件名就是接口名，然后文件里面的内容就是具体的实现类的路径加全类名
            Enumeration<URL> urls = classLoader.getResources("META-INF/services/" + service.getName());
            // 挨个遍历取到的文件
            while (urls.hasMoreElements()) {
                // 取出当前的文件
                URL url = urls.nextElement();
                System.out.println("File = " + url.getPath());
                // 建立链接
                URLConnection urlConnection = url.openConnection();
                urlConnection.setUseCaches(false);
                // 获取文件输入流
                InputStream inputStream = urlConnection.getInputStream();
                // 从文件输入流获取缓存
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                // 从文件内容里面得到实现类的全类名
                String className = bufferedReader.readLine();

                while (className != null) {
                    // 通过反射拿到实现类的实例
                    Class<?> clazz = Class.forName(className, false, classLoader);
                    // 如果声明的接口跟这个具体的实现类是属于同一类型，（可以理解为Java的一种多态，接口跟实现类、父类和子类等等这种关系。）则构造实例
                    if (service.isAssignableFrom(clazz)) {
                        Constructor<? extends S> constructor = (Constructor<? extends S>) clazz.getConstructor();
                        S instance = constructor.newInstance();
                        // 把当前构造的实例对象添加到 Provider的列表里面
                        providers.add(instance);
                    }
                    // 继续读取下一行的实现类，可以有多个实现类，只需要换行就可以了。
                    className = bufferedReader.readLine();
                }
            }
        } catch (Exception e) {
            System.out.println("读取文件异常。。。");
        }
    }

    // 返回spi接口对应的具体实现类列表
    public List<S> getProviders() {
        return providers;
    }
}
```

关键信息基本已经通过代码注释描述出来了，

主要的流程就是：

1. 通过 URL 工具类从 jar 包的/META-INF/services目录下面找到对应的文件，
2. 读取这个文件的名称找到对应的 spi 接口，
3. 通过InputStream流将文件里面的具体实现类的全类名读取出来，
4. 根据获取到的全类名，先判断跟 spi 接口是否为同一类型，如果是的，那么就通过反射的机制构造对应的实例对象，
5. 将构造出来的实例对象添加到Providers的列表中。

## 总结
其实不难发现，SPI 机制的具体实现本质上还是通过反射完成的。即：*_我们按照规定将要暴露对外使用的具体实现类在_***META-INF/services/****文件下声明。**

另外，SPI 机制在很多框架中都有应用：Spring 框架的基本原理也是类似的方式。还有 Dubbo 框架提供同样的 SPI 扩展机制，只不过 Dubbo 和 spring 框架中的 SPI 机制具体实现方式跟咱们今天学得这个有些细微的区别，不过整体的原理都是一致的，相信大家通过对 JDK 中 SPI 机制的学习，能够一通百通，加深对其他高深框的理解。

通过 SPI 机制能够大大地提高接口设计的灵活性，但是 SPI 机制也存在一些缺点，比如：

1. 遍历加载所有的实现类，这样效率还是相对较低的；
2. 当多个ServiceLoader同时load时，会有并发问题。



> 更新: 2024-01-02 20:03:05  
原文: [https://www.yuque.com/vip6688/neho4x/bcclfpn7mqi95s3x](https://www.yuque.com/vip6688/neho4x/bcclfpn7mqi95s3x)
>



> 更新: 2024-11-25 10:08:51  
> 原文: <https://www.yuque.com/neumx/laxg2e/cedd97824508849b717cd363309f625c>