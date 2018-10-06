---
layout: post
title:  "通过一个简单的例子,推导出SpringAop底层实现细节"
date:   2018-10-06 12:06:25 +0800
categories: Java
---

[TOC]

### 前提条件
本文不解释```Aop是什么?```、```为什么要使用Aop?```这些概念, 看本文前, 要求大家对这些概念要非常清楚, 不清楚的自行在网上搜索学习。

本文将通过一个简单的例子入手, 一步一步的剖析出SpringAop底层实现细节。由于SpringAop底层实现重要技术为动态代理(JDK动态代理、CGLIB), 在剖析中重点以JDK动态代理为切入点, 读本文前大家一定要熟悉JDK动态代理技术, 之前写过一篇博文<a href="http://joshsulin.github.io/java/2018/08/31/%E5%89%96%E6%9E%90JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%8A%80%E6%9C%AF/" target="_blank">《剖析JDK动态代理技术》</a>, 不明白的可以看看, 希望对大家有帮助。熟悉JDK动态代理技术的朋友, 一定对Proxy类、InvocationHandler接口、invoke方法、Proxy.newProxyInstance方法非常清楚, 每个类及方法调用时机及调用流程都要心中清楚, 这是阅读本文的基础。

### 从一个简单例子入手

#### 业务逻辑
```java
// 一个简单的商品接口
public interface IProductService {
    // 保存商品
    void save();
    // 删除商品
    void delete();
}
```

```java
// 一个简单的IProductService实现类
public class ProductService implements IProductService {
    public void save() {
        System.out.println("save product");
    }
    public void delete() {
        System.out.println("delete product");
    }
}
```

#### 定义一个切面
```java
@Aspect
public class PkgTypeAspectConfig {
    @Pointcut("within(com.josh.aop.service.ProductService)")
    public void matchType() {
    }

    @Before("matchType()")
    public void before() {
        System.out.println("");
        System.out.println("###before");
    }

    @After("matchType()")
    public void after() {
        System.out.println("");
        System.out.println("###after");
    }
}
```

#### XML配置
```java
// 文件名: spring-aop.xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd  
				http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
        
    <aop:aspectj-autoproxy />
		<bean id="productService" class="com.josh.aop.service.ProductService"/>
		<bean id="pkgTypeAspectConfig" class="com.josh.aop.config.PkgTypeAspectConfig" />
</beans>
```

#### 运行
```java
public class ProductServiceTest {
    @Test
    public void adminInsertTest() throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-aop.xml");
        IProductService productService = (IProductService) context.getBean("productService");
        productService.save();
        productService.delete();
    }
}
```

#### 运行结果
```java
###before
insert product
###after

###before
delete product
###after
```

以上例子不难理解, 这是编写SpringAop功能时的能用流程, 相信大家肯定都用过, 同时体现了框架的强大之处, 仅仅使用了几行代码, 就完成了如此强大的功能, 使用之余, 不知大家是否思考过, 为什么依赖几个简单的配置(@Aspect、@Pointcut、@Before、@After、aop:aspectj-autoproxy), 强大的功能就能实现, 这背后到底发生了什么?

网上几乎每篇文章都会说, SpringAop实现的核心是动态代理技术, <a href="http://joshsulin.github.io/java/2018/08/31/%E5%89%96%E6%9E%90JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%8A%80%E6%9C%AF/" target="_blank">《剖析JDK动态代理技术》</a>这篇文章让我们知道可以把内存中动态代理生成的类保存到文件中, 反编译可以查看生成的动态类。

```java
public class ProductServiceTest {
    @Test
    public void adminInsertTest() throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-aop.xml");
        IProductService productService = (IProductService) context.getBean("productService");
        byte[] proxyClassBytes = ProxyGenerator.generateProxyClass(productService.getClass().getSimpleName(), new Class<?>[]{IProductService.class});
        FileOutputStream fileOutputStream;
        fileOutputStream = new FileOutputStream(productService.getClass().getSimpleName() + ".class");
        fileOutputStream.write(proxyClassBytes);
        productService.save();
        productService.delete();
    }
}
```

运行上面代码, 可以在目录中看到生成了一个class文件, 通过对该class文件进行反编译, 反编译后的java文件代码如下:

```java
/*
 * Decompiled with CFR 0_132.
 *
 * Could not load the following classes:
 *  com.josh.aop.service.IProductService
 */
import com.josh.aop.service.IProductService;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy9 extends Proxy implements IProductService {
    private static Method m1;
    private static Method m4;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    public $Proxy9(InvocationHandler invocationHandler) throws  {
        super(invocationHandler);
    }

    public final boolean equals(Object object) throws  {
        try {
            return (Boolean)this.h.invoke(this, m1, new Object[]{object});
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final void save() throws  {
        try {
            this.h.invoke(this, m4, null);
            return;
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final String toString() throws  {
        try {
            return (String)this.h.invoke(this, m2, null);
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final void delete() throws  {
        try {
            this.h.invoke(this, m3, null);
            return;
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)this.h.invoke(this, m0, null);
        }
        catch (Error | RuntimeException throwable) {
            throw throwable;
        }
        catch (Throwable throwable) {
            throw new UndeclaredThrowableException(throwable);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m4 = Class.forName("com.josh.aop.service.IProductService").getMethod("save", new Class[0]);
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m3 = Class.forName("com.josh.aop.service.IProductService").getMethod("delete", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            return;
        }
        catch (NoSuchMethodException noSuchMethodException) {
            throw new NoSuchMethodError(noSuchMethodException.getMessage());
        }
        catch (ClassNotFoundException classNotFoundException) {
            throw new NoClassDefFoundError(classNotFoundException.getMessage());
        }
    }
}
```

我们再次来回顾一下使用JDK动态代理的方式, 结合<a href="http://joshsulin.github.io/java/2018/08/31/%E5%89%96%E6%9E%90JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%8A%80%E6%9C%AF/" target="_blank">《剖析JDK动态代理技术》</a>这篇文章, 是否产生了这样一个联想, Spring的AOP实现其实也是用了Proxy和InvocationHandler这两个东西, 在整个过程中, 对于InvocationHandler的创建是最为核心的, 在自定义的InvocationHandler中需要重写3个函数。<br/>
>1、构造函数, 将代理的对象传入。<br/>
>2、invoke方法, 此方法中实现了AOP增强的所有逻辑.<br/>
>3、Proxy.newProxyInstance() 方法, 此方法千篇一律, 但是必不可少。

通过己有知识推导SpringAOP的实现, 心中不停自问, SpringAOP中的JDK代理是不是也是这么做的呢? 如果是那以上这些代码在哪里呢? 带着这些问题又陷入沉思。到底实现了InvocationHandler接口的类是哪个？通过断点我们可以一看究竟。

<a href='/img/post/20181006/01.png' target="_blank"><img src='/img/post/20181006/01.png' /></a>

h -> JdkDynamicAopProxy, <a href="http://joshsulin.github.io/java/2018/08/31/%E5%89%96%E6%9E%90JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%E6%8A%80%E6%9C%AF/" target="_blank">《剖析JDK动态代理技术》</a>这篇文件里明确写了h这个成员变量是什么, 看到这里太高兴了, JdkDynamicAopProxy不就是实现了 InvocationHandler 的那个类嘛。接下来我们好好分析一下 JdkDynamicAopProxy 这个类。

### JdkDynamicAopProxy 类分析

```java
// 
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
    ......
    ......
    @Override
    public Object getProxy() {
        return getProxy(ClassUtils.getDefaultClassLoader());
    }

    @Override
    public Object getProxy(@Nullable ClassLoader classLoader) {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
        }
        Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
        findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
        return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
    }
    
    @Override
    @Nullable
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        ......
        ......
        ......
    }
    ......
    ......
}
```

通过该类, 可以看到上面要寻找的东西, Proxy.newProxyInstance()、invoke() 等等这些方法。

截止现在代理生成的类也清楚了, 调用方法时, 实际调用的代理类里面的invoke方法, invoke方法也找到了, 接下来我们好好分析一下invoke方法。

### JdkDynamicAopProxy.invoke() 方法分析(方法太长,就不列出方法体了)

#### 逻辑猜想
之前我们手写的两个例子, invoke方法的逻辑为. 在方法调用前写一段逻辑, 接着 method.invoke(targetClass, args), 最后在方法结束前写一段逻辑。顺着这个思路, 但是SpringAop中一个方法可以支持很多增强(advise), 如果当只有一个前置增强时, 可以将增强方法放在目标方法调用前执行, 如果当只有一个后置增强时, 可以将增强方法放在目标方法调用后执行, 如果当前置增强和后置增强都存在时, 可以将增强方法放在目标方法调用前后执行。但是如果有10个前置增强呢，或者10个后置增强，或者有无数个前置增强和后置增强呢，再如果有很多环绕增强呢, 硬编码肯定无法满足要求。

很多方法都需要执行, 但是按照顺序依次执行, 是不是我们把所有要执行的方法放在数组里面, 遍历该数组, 达到链式执行的效果, 这样再多的增强都可以顺序的执行完了。SpringAop里面真的是这样实现的吗? 带着疑问继续探索。

#### invoke核心逻辑
```java
public Object invoke(Object proxy, Method method, Object[] args) {
    ......
    ......
    // 获取当前方法的拦截器(各种增强方法)链
    List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
    ......
    ......
    // 将拦截器封装在ReflectiveMethodInvocation, 以便于使用其proceed进行链式调用拦截器
    invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
    // 执行拦截器链
    retVal = invocation.proceed();
    ......
    ......
    // 返回结果
    return retVal
}
```
针对以上代码, 我们先暂时不去细探, 下面我先罗列几个类, 因为通过执行拦截器链最终都会执行到下面几个类里面的相应方法。<br/>
@Before -> AtBefore -> AspectJMethodBeforeAdvice extends AbstractAspectJAdvice implements MethodBeforeAdvice, Serializable;<br/>
@After -> AtAfter -> AspectJAfterAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, AfterAdvice, Serializable;<br/>
@AfterReturning -> AtAfterReturning -> AspectJAfterReturningAdvice extends AbstractAspectJAdvice implements AfterReturningAdvice, AfterAdvice, Serializable;<br/>
@AfterThrowing -> AtAfterThrowing -> AspectJAfterThrowingAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, AfterAdvice, Serializable;<br/>
@Around -> AtAround -> AspectJAroundAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, Serializable;

以上5种增强方式中, @After、@AfterThrowing、@Around 这三个对应的类都实现了MethodInterceptor接口, 都具有invoke方法, 但是@Before、@AfterReturning所对应的类没有实现MethodInterceptor接口, 为了在调用链中统一能执行相同的方法, 我们需要将@Before、@AfterReturning这两个注解i所对应的类进行转化, 让其具备同样实现MethodInterceptor接口。getInterceptorsAndDynamicInterceptionAdvice()方法就是干了这个事情, 接着分析 getInterceptorsAndDynamicInterceptionAdvice 方法.

#### getInterceptorsAndDynamicInterceptionAdvice(method, targetClass) 
根据该方法可以看到如下代码:
```java
@Override
public MethodInterceptor[] getInterceptors(Advisor advisor) throws UnknownAdviceTypeException {
    List<MethodInterceptor> interceptors = new ArrayList<>(3);
    Advice advice = advisor.getAdvice();
    // 如果实现了MethodInterceptor接口的, 直接加入
    if (advice instanceof MethodInterceptor) {
        interceptors.add((MethodInterceptor) advice);
    }
    // 遍历所有的适配器, 满足要求的适配器, 将进行转化。
    // this.adapters => {
    //      new MethodBeforeAdviceAdapter(),
    //      new AfterReturningAdviceAdapter(),
    //      new ThrowsAdviceAdapter()
    // }
    for (AdvisorAdapter adapter : this.adapters) {
        if (adapter.supportsAdvice(advice)) {
            interceptors.add(adapter.getInterceptor(advisor));
        }
    }
    if (interceptors.isEmpty()) {
        throw new UnknownAdviceTypeException(advisor.getAdvice());
    }
    return interceptors.toArray(new MethodInterceptor[0]);
}
```
经过上面代码转化后, 所有的增强注解, 都会变成都实现了MethodInterceptor接口的类, 都具有invoke方法。<br/>
@Before -> AtBefore -> AspectJMethodBeforeAdvice extends AbstractAspectJAdvice implements MethodBeforeAdvice, Serializable -> MethodBeforeAdviceInterceptor implements <font color="red">MethodInterceptor</font>, BeforeAdvice, Serializable;<br/>
@After -> AtAfter -> AspectJAfterAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, AfterAdvice, Serializable;<br/>
@AfterReturning -> AtAfterReturning -> AspectJAfterReturningAdvice extends AbstractAspectJAdvice implements AfterReturningAdvice, AfterAdvice, Serializable -> AfterReturningAdviceInterceptor implements <font color="red">MethodInterceptor</font>, AfterAdvice, Serializable;<br/>
@AfterThrowing -> AtAfterThrowing -> AspectJAfterThrowingAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, AfterAdvice, Serializable;<br/>
@Around -> AtAround -> public class AspectJAroundAdvice extends AbstractAspectJAdvice implements <font color="red">MethodInterceptor</font>, Serializable;

接下来我们继续分析 执行拦截器链 方法, invocation.proceed()

#### invocation.proceed()
```java
public Object proceed() throws Throwable {
    ......
    // 递归调用proceed方法时, 当满足下面方法时, 调用目标方法
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        // 方法内实现方案为: 反射
        return invokeJoinpoint();
    }
    ......
    Object interceptorOrInterceptionAdvice = this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    ......
    return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
    ......
}
```
对该方法进行分析, 看到了invoke方法, 也看到了+1操作, <font color="red">猜想一下</font>, 是不是通过++操作, 不停的遍历增强器, 调用属于该bean的增强器的 invoke 方法。达到链式调用的效果。

当@After增强器, invoke代码如下:
```java
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        return mi.proceed();
    }
    finally {
        invokeAdviceMethod(getJoinPointMatch(), null, null);
    }
}
```
递归调用proceed()方法, finally所有的时机是否与After语义一致.

当@Before增强器, invoke代码如下:
```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
    return mi.proceed();
}

// this.advice.before 方法如下。
@Override
public void before(Method method, Object[] args, @Nullable Object target) throws Throwable {
    invokeAdviceMethod(getJoinPointMatch(), null, null);
}
```
先执行before增强所对应的方式, 然后再递归调用proceed(), 是否实现了Before语议?

当@AfterReturning增强器, invoke代码如下:
```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed();
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    return retVal;
}

// this.advice.afterReturning 方法如下。
@Override
public void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable {
    if (shouldInvokeOnReturnValueOf(method, returnValue)) {
        invokeAdviceMethod(getJoinPointMatch(), returnValue, null);
    }
}
```
其它具体的方式就不列举了, 都是相同的执行流程。

接下来, 我们模拟一下实际场景, 看看拦截器链的调用过程。
假设有4个拦截器, 两个@Before, 两个@After. 大家可以在线上画一下, 是如何进行递归调用的。

到目前为止, 代理类的方法调用代码就己分析完了, 但是我们还有很多疑问。<br/>
1、为什么当前代理类在执行时, 就能知道它具有哪些拦截器?<br/>
2、如何找到拦截器?<br/>
3、哪些bean需要被代理? 判断条件是什么?

### aop:aspectj-autoproxy 标签及bean始化化
怎样才能搞清楚上面3个问题, 其实<font color='red'>猜想一下</font>, 上面3个问题, 都可以归类于初始化阶段便能完成的操作, 在初始化所有非延迟单例bean时, 通过beanFactory获取到ioc里面所有的beanDefinition, 遍历所有的beanDefinition, 找到有@Aspect注解的bean, 解析出该bean所有的方法, 将@Before、@After等注解的方法封装成一个一个的增强器(advisor), 保存到缓存里面, 待bean初始化时, 取出切面表达式与beanClass、beanName进行匹配, 看是否能匹配到切面表达式, 如果能匹配就将满足条件的advisor与beanName进行K-V存储。同时需要将该beanName生成代理类，生成代理类时, 将能匹配上目标类的增强器赋值给InvocationHandler类的成员变量, 当执行代理类方法时好使用。

对熟悉Spring初始化流程的开发来说, 上面的解决方案应该不难想到, 初始化肯定涉及到标签解析及bean初始化, 接着往下分析.

#### aop:aspectj-autoproxy 标签逻辑分析
关于自定义标签的解析, 这里不做详细说明。通过aop命名空间, 不难找到这句配置
```java
http\://www.springframework.org/schema/aop=org.springframework.aop.config.AopNamespaceHandler
```
接下来
```java
public class AopNamespaceHandler extends NamespaceHandlerSupport {
    @Override
    public void init() {
        ......
        registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
        ......
    }
}
```
接下来
```java
class AspectJAutoProxyBeanDefinitionParser implements BeanDefinitionParser {
    public BeanDefinition parse(Element element, ParserContext parserContext) {
        AopNamespaceUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(parserContext, element);
        extendBeanDefinition(element, parserContext);
        return null;
    }
}
```
以上代码,比较简单,大家去断点跟一下就大体能明白,我在这里总结一下,aop:aspectj-autoproxy该标签完成了对AnnotationAwareAspectJAutoProxyCreator类型的自动注册, 那么这个类到底做了什么工作来完成AOP的操作呢? 这才是我们接下来的重点。

#### AnnotationAwareAspectJAutoProxyCreator类分析
<a href='/img/post/20181006/02.png' target="_blank"><img src='/img/post/20181006/02.png' /></a>
我们看到AnnotationAwareAspectJAutoProxyCreator实现了BeanPostProcessor接口,而实现BeanPostProcessor后, 当Spring加载Bean时会在实例前调用其postProcessAfterInitialization方法。不明白的请大家看我之前的博文<a href="http://joshsulin.github.io/java/2018/10/01/%E8%AF%BB%E6%87%82Spring%E6%A1%86%E6%9E%B6%E5%90%84%E7%BB%84%E4%BB%B6%E5%AE%9E%E7%8E%B0%E6%BA%90%E7%A0%81%E7%9A%84%E4%B8%89%E4%B8%AA%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AF%86%E7%82%B9/" target="_blank">《学习基于Spring框架开发的核心功能源码时, 首先得清楚以下3个知识点》</a> 。而我们对于AOP初始化逻辑的分析也由此开始。

#### AbstractAutoProxyCreator的postProcessAfterInitialization方法
```java
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (!this.earlyProxyReferences.contains(cacheKey)) {
            // 如果它适合被代理, 则需要封装指定bean
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    ......
    ......
    // Create proxy if we have advice.
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        Object proxy = createProxy(bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }
    ......
    ......
}
```
函数中我们己经看到了代理创建的雏形。当然, 真正开始之前还需要经过一些判断, 比如是否己经处理过或者是否是需要跳过的bean, 而真正创建代理的代码是从getAdvicesAndAdvisorsForBean开始的。
创建代理主要包含两个步骤:
>1、获取增强方法或者增强器;<br/>
>2、根据获取的增强进行代理。

#### 获取增强器及能匹配当前bean
调用的类关系如下:<br/>
AbstractAdvisorAutoProxyCreator.getAdvicesAndAdvisorsForBean<br/>
=> AbstractAdvisorAutoProxyCreator.findEligibleAdvisors<br/>
==> AbstractAdvisorAutoProxyCreator.findCandidateAdvisors<br/>
===> BeanFactoryAdvisorRetrievalHelper.findAdvisorBeans<br/>
==> AbstractAdvisorAutoProxyCreator.findAdvisorsThatCanApply<br/>
===> AopUtils.findAdvisorsThatCanApply<br/>
大体流程如下, 由于以上源码太多并且过于复杂, 这里以流程图的形式来梳理逻辑:<br/>
<a href='/img/post/20181006/03.png' target="_blank"><img src='/img/post/20181006/03.png' /></a>
<a href='/img/post/20181006/04.png' target="_blank"><img src='/img/post/20181006/04.png' /></a>
<a href='/img/post/20181006/05.png' target="_blank"><img src='/img/post/20181006/05.png' /></a>
<a href='/img/post/20181006/06.png' target="_blank"><img src='/img/post/20181006/06.png' /></a>

通过上面流程图及图上注释, 相信大家能明白这个逻辑。

### 创建代理
这里只写JDK动态代理, 上面的事例也只会走到这个逻辑里面来, CGLIB不写了。动态代理需要实现了InvocationHandler接口的类, 代码如下:
```java
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    ......
    ......
    return new JdkDynamicAopProxy(config);
    ......
    ......
}
```

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {
    // 构造函数
    public JdkDynamicAopProxy(AdvisedSupport config) throws AopConfigException {
        ......
        // 将满足该bean的增强器赋值给了成员变量advised
        this.advised = config;
    }
}
```

```java
@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
    if (logger.isDebugEnabled()) {
        logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
    }
    Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
    findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
    return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```
Proxy.newProxyInstance函数所涉及的3个参数、含义及逻辑就非常的清晰了。

### 总结

通过上面分析, SpringAop涉及以下三大块功能, 简单画了一下流程图.
<a href='/img/post/20181006/07.png' target="_blank"><img src='/img/post/20181006/07.png' /></a>

  [7]: http://joshsulin.github.io/java/2018/10/01/%E8%AF%BB%E6%87%82Spring%E6%A1%86%E6%9E%B6%E5%90%84%E7%BB%84%E4%BB%B6%E5%AE%9E%E7%8E%B0%E6%BA%90%E7%A0%81%E7%9A%84%E4%B8%89%E4%B8%AA%E5%89%8D%E6%8F%90%E7%9F%A5%E8%AF%86%E7%82%B9/
