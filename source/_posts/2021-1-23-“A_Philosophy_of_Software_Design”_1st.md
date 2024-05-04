---
layout: post
title: A Philosophy of Software Design读后感 0-4.4
tag: 读书
date: 2021-1-23
category: Read
---

#### **The Nature of Complexity**（关于复杂性的本质）

首先关于复杂的本质定义，原文之类的定义为：复杂性是指与软件系统的结构相关的任何东西，它使得理解和修改系统变得困难。

Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system. 

复杂性带来的有三个症状

- **Change amplification** (改动放大)
- **Cognitive load** （认知负荷）
- **Unknown unknowns**（未知的未知）

同样，继续阐述了系统复杂性的来源：复杂性是由两个因素造成的:依赖性和晦涩性

Complexity is caused by two things: *dependencies* and *obscurity*.

依赖导致变化的放大和高认知负荷。晦涩会产生未知的未知，也会增加认知负荷。如果我们能够找到最小化依赖和模糊的设计技术，那么我们就可以减少软件的复杂性。

- Dependencies lead to change amplification and a high cognitive load. Obscurity creates unknown unknowns, and also contributes to cognitive load. If we can find design techniques that minimize dependencies and obscurity, then we can reduce the complexity of software.

同时表明复杂性是会增长，复杂性的递增性使其难以控制。你很容易说服自己，当前的变化带来的一点点复杂性没什么大不了的。然而，如果每个开发人员对每个变更都采用这种方法，复杂性就会迅速累积。一旦复杂性积累起来，就很难消除了，因为修复单一的依赖关系或晦涩性本身不会产生很大的影响。为了减缓复杂性的增长，您必须采取“零容忍”。

- Complexity comes from an accumulation of dependencies and obscurities. As complexity increases, it leads to change amplification, a high cognitive load, and unknown unknowns. As a result, it takes more code modifications to implement each new feature. In addition, developers spend more time acquiring enough information to make the change safely and, in the worst case, they can’t even find all the information they need. The bottom line is that complexity makes it difficult and risky to modify an existing code base.

复杂性来自依赖和模糊的积累。随着复杂性的增加，它会导致变化的放大、高认知负荷和未知的未知。因此，要实现每个新特性需要进行更多的代码修改。此外，开发人员要花更多的时间来获取足够的信息来安全地进行更改，在最坏的情况下，他们甚至无法找到他们需要的所有信息。归根结底，复杂性使得修改现有代码库变得困难和有风险。

- Complexity comes from an accumulation of dependencies and obscurities. As complexity increases, it leads to change amplification, a high cognitive load, and unknown unknowns. As a result, it takes more code modifications to implement each new feature. In addition, developers spend more time acquiring enough information to make the change safely and, in the worst case, they can’t even find all the information they need. The bottom line is that complexity makes it difficult and risky to modify an existing code base.

#### **Strategic vs. Tactical Programming**（战略规划以及战术规划）

**Tactical Programming**

大多数程序员以一种我称之为战术编程的心态来进行软件开发。在战术方法中，您的主要关注点是使某些东西能够工作，例如新特性或bug修复。乍一看，这似乎完全合理:还有什么比编写有效的代码更重要呢?然而，战术编程几乎不可能产生好的系统设计。

- Most programmers approach software development with a mindset I call *tactical programming*. In the tactical approach, your main focus is to get something working, such as a new feature or a bug fix. At first glance this seems totally reasonable: what could be more important than writing code that works? However, tactical programming makes it nearly impossible to produce a good system design.

**Strategic programming**

为了更快地完成你当前的任务。最重要的是体系的长期结构。任何系统中的大多数代码都是通过扩展现有的代码库来编写的，所以作为开发人员，您最重要的工作是促进这些未来的扩展。因此，您不应该将“工作代码”视为您的主要目标，尽管您的代码当然必须工作。你的首要目标必须是创造出出色的设计，而这也恰好能够发挥作用。这是战略规划。

- in order to finish your current task faster. The most important thing is the long-term structure of the system. Most of the code in any system is written by extending the existing code base, so your most important job as a developer is to facilitate those future extensions. Thus, you should not think of “working code” as your primary goal, though of course your code must work. Your primary goal must be to produce a great design, which also happens to work. This is *strategic programming*.

**How much**

那我们一般需要多少时间来进行战术规划的前期投资，这里给出的答案10%-20%，你最初的项目将比纯粹的战术方法多花费10-20%的时间。这些额外的时间将会带来更好的软件设计，你将会在几个月内开始体验这些好处。用不了多久，您的开发速度就会比采用战术编程时至少快10-20%。在这一点上，你的投资是免费的:你过去投资的收益将节省足够的时间来支付未来的投资成本。您将很快收回初始投资的成本。

- Your initial projects will thus take 10–20% longer than they would in a purely tactical approach. That extra time will result in a better software design, and you will start experiencing the benefits within a few months. It won’t be long before you’re developing at least 10–20% faster than you would if you had programmed tactically. At this point your investments become free: the benefits from your past investments will save enough time to cover the cost of future investments. You will quickly recover the cost of the initial investment.

**Conclusion**

本章主要是在强调，软件设计在编码前的重要性，好的设计不是免费的。它必须是你不断投入的东西，这样小问题才不会累积成大问题。幸运的是，好的设计最终会带来回报，而且比你想象的要快。

- Good design doesn’t come for free. It has to be something you invest in continually, so that small problems don’t accumulate into big ones. Fortunately, good design eventually pays for itself, and sooner than you might think.
#### **Modules Should Be Deep**（模块应该具有相应的深度）


在软件设计中为什么需要引入Modules(模块)?


管理软件复杂性最重要的技术之一是设计系统，以便开发人员在任何给定时间只需要面对整体复杂性的一小部分。这种方法被称为模块化设计。


- One of the most important techniques for managing software complexity is to design systems so that developers only need to face a small fraction of the overall complexity at any given time. This approach is called *modular design*


**Modular design**


理想的模块化设计


在模块化设计中，软件系统被分解成一组相对独立的模块。模块可以采取多种形式，比如类、子系统或服务。在理想情况下，每个模块都应该完全独立于其他模块:开发人员可以在任何模块中工作，而不需要了解任何其他模块。在这个世界上，一个系统的复杂性就是它最坏模块的复杂性。


- In modular design, a software system is decomposed into a collection of *modules* that are relatively independent. Modules can take many forms, such as classes, subsystems, or services. In an ideal world, each module would be completely independent of the others: a developer could work in any of the modules without knowing anything about any of the other modules. In this world, the complexity of a system would be the complexity of its worst module.


在这里还阐述了接口与实现的意义，为了管理依赖关系，我们将每个模块分为两部分:接口和实现。该接口包含了工作在不同模块中的开发人员为了使用给定模块必须知道的所有内容。通常，接口描述模块做什么，而不是它如何做。实现由执行接口所作承诺的代码组成。在特定模块中工作的开发人员必须了解该模块的接口和实现，以及由给定模块调用的任何其他模块的接口。一个开发者，因此，不管内部的实现逻辑多么的复杂，模块向外提供的始终应该是统一理解的。


- In order to manage dependencies, we think of each module in two parts: an *interface* and an *implementation*. The interface consists of everything that a developer working in a different module must know in order to use the given module. Typically, the interface describes *what* the module does but not *how* it does it. The implementation consists of the code that carries out the promises made by the interface. A developer working in a particular module must understand the interface and implementation of that module, plus the interfaces of any other modules invoked by the given module. A developer should not need to understand the implementations of modules other than the one he or she is working in.


一个好的模块设计，当实现的改动不会导致接口的改动，那么其他模块所关联这个接口的逻辑，即没有影响所以也无需改变。


优秀的接口设计能够有效地避免 **未知的未知**


- One of the benefits of a clearly specified interface is that it indicates exactly what developers need to know in order to use the associated module. This helps to eliminate the “unknown unknowns”


**Abstractions**


抽象是一个实体的简化视图，它省略了不重要的细节。


An abstraction is a simplified view of an entity, which omits unimportant details.


在日常生活中，我们通过抽象化来提升用户的体验。微波炉含有复杂的电子元件，可以将交流电转换成微波辐射，并将辐射散布到整个烹饪腔内。幸运的是，用户看到了一个简单得多的抽象概念，由几个按钮组成，用来控制微波的时间和强度。汽车提供了一个简单的抽象概念，允许我们在不了解电机、电池电源管理、防抱死刹车、巡航控制等机制的情况下驾驶汽车


- We depend on abstractions to manage complexity not just in programming, but pervasively in our everyday lives. A microwave oven contains complex electronics to convert alternating current into microwave radiation and distribute that radiation throughout the cooking cavity. Fortunately, users see a much simpler abstraction, consisting of a few buttons to control the timing and intensity of the microwaves. Cars provide a simple abstraction that allows us to drive them without understanding the mechanisms for electrical motors, battery power management, anti-lock brakes, cruise control, and so on.


在抽象概念的定义中，“不重要”这个词是至关重要的。抽象概念中省略的不重要细节越多越好。然而，如果一个细节是不重要的，它只能从抽象中被忽略。抽象可能在两方面出错。首先，它可以包含一些并不重要的细节;当这种情况发生时，它使抽象变得比必要的更复杂，这增加了使用抽象的开发人员的认知负荷。


- In the definition of abstraction, the word “unimportant” is crucial. The more unimportant details that are omitted from an abstraction, the better. However, a detail can only be omitted from an abstraction if it is unimportant. An abstraction can go wrong in two ways. First, it can include details that are not really important; when this happens, it makes the abstraction more complicated than necessary, which increases the cognitive load on developers using the abstraction.


第二个错误是抽象忽略了真正重要的细节。这导致了模糊:只看抽象的开发人员不会得到正确使用抽象所需要的所有信息。忽略重要细节的抽象是错误的抽象:它可能看起来很简单，但实际上并非如此。设计抽象的关键是理解什么是重要的，并寻找最小化重要信息的设计。


- The second error is when an abstraction omits details that really are important. This results in obscurity: developers looking only at the abstraction will not have all the information they need to use the abstraction correctly. An abstraction that omits important details is a *false abstraction*: it might appear simple, but in reality it isn’t. The key to designing abstractions is to understand what is important, and to look for designs that minimize the amount of information that is important.