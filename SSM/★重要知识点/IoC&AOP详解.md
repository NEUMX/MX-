# IoC&AOP详解

# IoC & AOP详解
这篇文章会从下面从以下几个问题展开对 IoC & AOP 的解释

+ 什么是 IoC？
+ IoC 解决了什么问题？
+ IoC 和 DI 的区别？
+ 什么是 AOP？
+ AOP 解决了什么问题？
+ AOP 的应用场景有哪些？
+ AOP 为什么叫做切面编程？
+ AOP 实现方式有哪些？

首先声明：IoC & AOP 不是 Spring 提出来的，它们在 Spring 之前其实已经存在了，只不过当时更加偏向于理论。Spring 在技术层次将这两个思想进行了很好的实现。

## IoC （Inversion of control ）
### 什么是 IoC?
IoC （Inversion of Control ）即控制反转/反转控制。它是一种思想不是一个技术实现。描述的是：Java 开发领域对象的创建以及管理的问题。

例如：现有类 A 依赖于类 B

+ **传统的开发方式**：往往是在类 A 中手动通过 new 关键字来 new 一个 B 的对象出来
+ **使用 IoC 思想的开发方式**：不通过 new 关键字来创建对象，而是通过 IoC 容器(Spring 框架) 来帮助我们实例化对象。我们需要哪个对象，直接从 IoC 容器里面过去即可。

从以上两种开发方式的对比来看：我们 “丧失了一个权力” (创建、管理对象的权力)，从而也得到了一个好处（不用再考虑对象的创建、管理等一系列的事情）

**为什么叫控制反转?**

+ **控制**：指的是对象创建（实例化、管理）的权力
+ **反转**：控制权交给外部环境（IoC 容器）

![1732497746115-e378bdc4-df84-456a-b2fc-73fdd479731d.png](./img/Rrotawkz5168smJ3/1732497746115-e378bdc4-df84-456a-b2fc-73fdd479731d-940169.png)

IoC 图解

### IoC 解决了什么问题?
IoC 的思想就是两方之间不互相依赖，由第三方容器来管理相关资源。这样有什么好处呢？

1. 对象之间的耦合度或者说依赖程度降低；
2. 资源变的容易管理；比如你用 Spring 容器提供的话很容易就可以实现一个单例。

例如：现有一个针对 User 的操作，利用 Service 和 Dao 两层结构进行开发

在没有使用 IoC 思想的情况下，Service 层想要使用 Dao 层的具体实现的话，需要通过 new 关键字在UserServiceImpl中手动 new 出IUserDao的具体实现类UserDaoImpl（不能直接 new 接口类）。

![1732497746194-ba9372b4-5652-4fba-99cf-f99e5d0ebd66.png](./img/Rrotawkz5168smJ3/1732497746194-ba9372b4-5652-4fba-99cf-f99e5d0ebd66-967493.png)

很完美，这种方式也是可以实现的，但是我们想象一下如下场景：

开发过程中突然接到一个新的需求，针对对IUserDao接口开发出另一个具体实现类。因为 Server 层依赖了IUserDao的具体实现，所以我们需要修改UserServiceImpl中 new 的对象。如果只有一个类引用了IUserDao的具体实现，可能觉得还好，修改起来也不是很费力气，但是如果有许许多多的地方都引用了IUserDao的具体实现的话，一旦需要更换IUserDao的实现方式，那修改起来将会非常的头疼。

![1732497746276-3691aa4f-a6e9-4f07-83dc-a877e82d910e.png](./img/Rrotawkz5168smJ3/1732497746276-3691aa4f-a6e9-4f07-83dc-a877e82d910e-738833.png)

使用 IoC 的思想，我们将对象的控制权（创建、管理）交有 IoC 容器去管理，我们在使用的时候直接向 IoC 容器 “要” 就可以了

![1732497746360-cd3f5969-0cb1-4a0b-b1b1-0e4805da6c84.png](./img/Rrotawkz5168smJ3/1732497746360-cd3f5969-0cb1-4a0b-b1b1-0e4805da6c84-477179.png)

### IoC 和 DI 有区别吗？
IoC（Inverse of Control:控制反转）是一种设计思想或者说是某种模式。这个设计思想就是**将原本在程序中手动创建对象的控制权交给第三方比如 IoC 容易。**对于我们常用的 Spring 框架来说， IoC 容器实际上就是个 Map（key，value）,Map 中存放的是各种对象。不过，IoC 在其他语言中也有应用，并非 Spring 特有。

IoC 最常见以及最合理的实现方式叫做依赖注入（Dependency Injection，简称 DI）。

老马（Martin Fowler）在一篇文章中提到将 IoC 改名为 DI，原文如下，原文地址：[https://martinfowler.com/articles/injection.html](https://martinfowler.com/articles/injection.html)[open in new window](https://martinfowler.com/articles/injection.html)。

![1732497746439-5980193b-d0f4-48f6-b402-98c889015813.png](./img/Rrotawkz5168smJ3/1732497746439-5980193b-d0f4-48f6-b402-98c889015813-919912.png)

老马的大概意思是 IoC 太普遍并且不表意，很多人会因此而迷惑，所以，使用 DI 来精确指名这个模式比较好。

## AOP（Aspect oriented programming）
这里不会涉及太多专业的术语，核心目的是将 AOP 的思想说清楚。

### 什么是 AOP？
AOP（Aspect Oriented Programming）即面向切面编程，AOP 是 OOP（面向对象编程）的一种延续，二者互补，并不对立。

AOP 的目的是将横切关注点（如日志记录、事务管理、权限控制、接口限流、接口幂等等）从核心业务逻辑中分离出来，通过动态代理、字节码操作等技术，实现代码的复用和解耦，提高代码的可维护性和可扩展性。OOP 的目的是将业务逻辑按照对象的属性和行为进行封装，通过类、对象、继承、多态等概念，实现代码的模块化和层次化（也能实现代码的复用），提高代码的可读性和可维护性。

### AOP 为什么叫面向切面编程？
AOP 之所以叫面向切面编程，是因为它的核心思想就是将横切关注点从核心业务逻辑中分离出来，形成一个个的**切面（Aspect）**。

这里顺带总结一下 AOP 关键术语（不理解也没关系，可以继续往下看）：

+ **横切关注点（cross-cutting concerns）**：多个类或对象中的公共行为（如日志记录、事务管理、权限控制、接口限流、接口幂等等）。
+ **切面（Aspect）**：对横切关注点进行封装的类，一个切面是一个类。切面可以定义多个通知，用来实现具体的功能。
+ **连接点（JoinPoint）**：连接点是方法调用或者方法执行时的某个特定时刻（如方法调用、异常抛出等）。
+ **通知（Advice）**：通知就是切面在某个连接点要执行的操作。通知有五种类型，分别是前置通知（Before）、后置通知（After）、返回通知（AfterReturning）、异常通知（AfterThrowing）和环绕通知（Around）。前四种通知都是在目标方法的前后执行，而环绕通知可以控制目标方法的执行过程。
+ **切点（Pointcut）**：一个切点是一个表达式，它用来匹配哪些连接点需要被切面所增强。切点可以通过注解、正则表达式、逻辑运算等方式来定义。比如execution(* com.xyz.service..*(..))匹配com.xyz.service包及其子包下的类或接口。
+ **织入（Weaving）**：织入是将切面和目标对象连接起来的过程，也就是将通知应用到切点匹配的连接点上。常见的织入时机有两种，分别是编译期织入（AspectJ）和运行期织入（AspectJ）。

### AOP 解决了什么问题？
OOP 不能很好地处理一些分散在多个类或对象中的公共行为（如日志记录、事务管理、权限控制、接口限流、接口幂等等），这些行为通常被称为**横切关注点（cross-cutting concerns）**。如果我们在每个类或对象中都重复实现这些行为，那么会导致代码的冗余、复杂和难以维护。

AOP 可以将横切关注点（如日志记录、事务管理、权限控制、接口限流、接口幂等等）从**核心业务逻辑（core concerns，核心关注点）**中分离出来，实现关注点的分离。

![1732497746508-0ee0ac4a-5bf2-472b-a34f-08a964fc4f1c.png](./img/Rrotawkz5168smJ3/1732497746508-0ee0ac4a-5bf2-472b-a34f-08a964fc4f1c-934089.png)

以日志记录为例进行介绍，假如我们需要对某些方法进行统一格式的日志记录，没有使用 AOP 技术之前，我们需要挨个写日志记录的逻辑代码，全是重复的的逻辑。

```plain
public CommonResponse<Object> method1() {
      // 业务逻辑
      xxService.method1();
      // 省略具体的业务处理逻辑
      // 日志记录
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // 省略记录日志的具体逻辑 如：获取各种信息，写入数据库等操作...
      return CommonResponse.success();
}

public CommonResponse<Object> method2() {
      // 业务逻辑
      xxService.method2();
      // 省略具体的业务处理逻辑
      // 日志记录
      ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = attributes.getRequest();
      // 省略记录日志的具体逻辑 如：获取各种信息，写入数据库等操作...
      return CommonResponse.success();
}

// ...
```

使用 AOP 技术之后，我们可以日志记录的逻辑封装成一个切面，然后通过切入点和通知来指定在哪些方法需要执行日志记录的操作。

```plain
// 日志注解
@Target({ElementType.PARAMETER,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Log {

    /**
     * 描述
     */
    String description() default "";

    /**
     * 方法类型 INSERT DELETE UPDATE OTHER
     */
    MethodType methodType() default MethodType.OTHER;
}

// 日志切面
@Component
@Aspect
public class LogAspect {
  // 切入点，所有被 Log 注解标注的方法
  @Pointcut("@annotation(cn.javaguide.annotation.Log)")
  public void webLog() {
        }

   /**
   * 环绕通知
   */
  @Around("webLog()")
  public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
    // 省略具体的处理逻辑
  }

  // 省略其他代码
}
```

这样的话，我们一行注解即可实现日志记录：

```plain
@Log(description = "method1",methodType = MethodType.INSERT)
public CommonResponse<Object> method1() {
      // 业务逻辑
      xxService.method1();
      // 省略具体的业务处理逻辑
      return CommonResponse.success();
}
```

### AOP 的应用场景有哪些？
+ 日志记录：自定义日志记录注解，利用 AOP，一行代码即可实现日志记录。
+ 性能统计：利用 AOP 在目标方法的执行前后统计方法的执行时间，方便优化和分析。
+ 事务管理：@Transactional注解可以让 Spring 为我们进行事务管理比如回滚异常操作，免去了重复的事务管理逻辑。@Transactional注解就是基于 AOP 实现的。
+ 权限控制：利用 AOP 在目标方法执行前判断用户是否具备所需要的权限，如果具备，就执行目标方法，否则就不执行。例如，SpringSecurity 利用@PreAuthorize注解一行代码即可自定义权限校验。
+ 接口限流：利用 AOP 在目标方法执行前通过具体的限流算法和实现对请求进行限流处理。
+ 缓存管理：利用 AOP 在目标方法执行前后进行缓存的读取和更新。
+ ……

### AOP 实现方式有哪些？
AOP 的常见实现方式有动态代理、字节码操作等方式。

Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用**Cglib**生成一个被代理对象的子类来作为代理，如下图所示：

![1732497746616-d326f9a0-5cad-4177-9a2a-2ad50b48c019.jpeg](./img/Rrotawkz5168smJ3/1732497746616-d326f9a0-5cad-4177-9a2a-2ad50b48c019-504771.jpeg)

SpringAOPProcess

当然你也可以使用**AspectJ**！Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。**Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多。



> 更新: 2024-01-02 22:56:47  
原文: [https://www.yuque.com/vip6688/neho4x/vtupaxdfgbm3w53h](https://www.yuque.com/vip6688/neho4x/vtupaxdfgbm3w53h)
>



> 更新: 2024-11-25 09:22:27  
> 原文: <https://www.yuque.com/neumx/laxg2e/ecf67d89dd7f975e754793d243ce23af>