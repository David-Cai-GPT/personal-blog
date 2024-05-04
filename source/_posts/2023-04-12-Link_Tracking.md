---
layout: post
title: 链路追踪
tag: 技术笔记
date: 2023-4-12
category: Technology
---

### 分布式链路追踪技术

#### 场景

​     为了支撑日益增长的庞大业务量，我们会使用微服务架构设计我们的系统，使得我们的系统不仅能够通过集群部署抵挡流量的冲击，有能根据业务进行灵活的扩展。那么，在微服务架构下，一次请求少则经过三四次服务调用完成，多则跨越几十个甚至是上百个服务节点。那么问题接踵而来：

●如何动态展示服务的整个调用链路

●如何直观的分析调用链路中的瓶颈节点并可以对其调优

●快速故障发现

#### 简介

​     业务完整链路各个节点都能够记录下日志，并最终将日志进行集中可视化展示，那么我们想监控调用链路中的一些指标就有希望了，比如，请求到达哪个服务实例？请求被处理的状态怎样？处理耗时怎样？

​     分布式环境下基于这种想法实现的监控技术就是就是分布式链路追踪（全链路追踪）。

#### 约定

**Trace**: 服务追踪的单元，从客户发起请求（request）抵达被追踪系统的边界开始，到被追踪的系统向客户返回响应（response）为止的过程

**Trace ID**：为了实现请求跟踪，当请求发送到分布式系统的入口端点时，只需要服务跟踪组件为该请求创建一个唯一的跟踪标识Trace ID，同时在分布式系统内部流转的时候，链路追踪组件始终保持这一唯一标识，知道返回给请求方。

​     一个Trace由一个或者多个Span组成，每一个Span都有一个SpanId，Span中会记录TraceId，同时还有一个叫做ParentId，指向了另外一个Span的SpanId，表明父子关系，其实本质表达了依赖关系。

**Span ID**：为了统计各个单元的时间延迟，当请求到达各个服务组件时，也是通过一个唯一标识Span ID来标记它的开始，具体过程以及结束。对每一个Span来说，它必须又开始和结束两个节点，通过记录开始Span和结束Span的时间戳，就能统计出该Span的时间延迟，除了时间戳记录之外，它还可以包含一些其他元数据，比如时间名称以及请求信息等。

每一个Span都会有一个唯一跟踪标识Span ID，若干个有序span就组成了一个trace。

**Annotation：**

Span可以认为是一个日志数据结构，在一些特殊的时机点会记录了一些日志信息，比如时间戳、SpanId、TraceId，parentId等，Span中也抽象出了另外一个概念，叫做事件，核心事件如下：

- CS ：client send/start 客户端/消费者发出一个请求，描述的是一个span开始
- SR: server received/start 服务端/生产者接收请求 SR-CS属于请求发送的网络延迟
- SS: server send/finish 服务端/生产者发送应答 SS-SR属于服务端消耗时间
- CR：client received/finished 客户端/消费者接收应答 CR-SS表示回复需要的时间(响应的网络延迟)

#### 实践

这里使用了spring切面来实现一个简单的链路追踪

首先实现一个切面注解，加入方法名，方法描述，方法标识

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LinkTrackingAnnotation {

    /**
     * 方法名
     *
     * @return
     */
    String methodName() default "";

    /**
     * 方法描述
     *
     * @return
     */
    String desc() default "";

    /**
     * 方法标识
     *
     * @return
     */
    String flag() default "";

}
```

实现切面的具体逻辑

```java
@Aspect
@Component
@Slf4j
public class LinkTrackingAspect {

    public static final String TRACE_ID = "traceID";

    public static final String SYSTEM_NAME = "evo-vehicle";

    @Pointcut("@annotation(com.dahua.evo.vehicle.common.annotation.LinkTrackingAnnotation)")
    public void logPointCut() {
    }

    @Around(value = "logPointCut()")
    public void linkTrackingCut(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Signature signature = pjp.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        Method method = methodSignature.getMethod();
        LinkTrackingAnnotation linkTrackingAnnotation = method.getAnnotation(LinkTrackingAnnotation.class);
        String desc = linkTrackingAnnotation.desc();
        String flag = linkTrackingAnnotation.flag();
        String methodName = StringUtil.isEmpty(linkTrackingAnnotation.methodName()) ? signature.getDeclaringTypeName() +
                StringKit.UNDERLINE + signature.getName() : linkTrackingAnnotation.methodName();

        String traceId = UUID.randomUUID().toString().replace("-", "");
        if (StringUtil.isNotEmpty(flag)) {
            traceId = flag + StringKit.UNDERLINE + traceId;
        }
        traceId = SYSTEM_NAME + StringKit.UNDERLINE + traceId;

        // 随机数放到此线程的上下文中，可以在每条日志前加入,需要配置log4j2.xml
        ThreadContext.put(TRACE_ID, traceId);
        if (Objects.nonNull(pjp.getArgs())) {
            log.debug("方法: {}请求入参: {}", methodName, JSON.toJSONString(pjp.getArgs()));
        }
        Object result = pjp.proceed();
        long end = System.currentTimeMillis();
        if (Objects.nonNull(result)) {
            log.debug("方法: {}请求入参: {}", methodName, JSON.toJSONString(result));
        }

        log.info("方法{}，描述{}，总耗时: {}ms，链路追踪标识：{}", methodName, desc, end - begin, traceId);
    }

}
```

#### 常见主流技术

目前主流的链路追踪工具：

- Google的Dapper
- 阿里的鹰眼
- 大众点评的CAT
- Twitter的Zipkin
- LINE的pinpoint

[分布式链路追踪技术](https://blog.csdn.net/weixin_52851967/article/details/126597746)
