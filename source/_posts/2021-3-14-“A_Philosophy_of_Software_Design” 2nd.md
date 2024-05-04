---
layout: post
title: A Philosophy of Software Design读后感 4.4-7.6
tag: 读书
date: 2021-3-14
category: Read
---

Deep modules


最好的模块是那些功能强大但接口简单的模块。


- The best modules are those that provide powerful functionality yet have simple interfaces.


最好的模块是有深度的:它们有很多隐藏在简单界面后面的功能。深度模块是一个很好的抽象，因为用户只能看到它内部复杂性的一小部分。


- The best modules are deep: they have a lot of functionality hidden behind a simple interface. A deep module is a good abstraction because only a small fraction of its internal complexity is visible to its users.


这里通过举例UNIX系统 I/O操作和java语言以及go语言的垃圾回收操作，解释了优秀的模块设计往往将难以理解的逻辑隐藏，对外提供的则是简单易操作的接口。


- Deep modules such as Unix I/O and garbage collectors provide powerful abstractions because they are easy to use, yet they hide significant implementation complexity.


**Shallow modules**


相对于它所提供的功能而言，浅模块的接口相对复杂。例如，实现链表的类是浅的。操作一个链表并不需要太多的代码(插入或删除一个元素只需要几行代码)，因此链表抽象不会隐藏很多细节。链表接口的复杂性几乎与其实现的复杂性一样大。浅层类有时是不可避免的，但它们在管理复杂性方面没有提供多少帮助。


- A shallow module is one whose interface is relatively complex in comparison to the functionality that it provides. For example, a class that implements linked lists is shallow. It doesn’t take much code to manipulate a linked list (inserting or deleting an element takes only a few lines), so the linked list abstraction doesn’t hide very many details. The complexity of a linked list interface is nearly as great as the complexity of its implementation. Shallow classes are sometimes unavoidable, but they don’t provide help much in managing complexity.


**Red Flag: Shallow Module** (需要标注小红旗)


浅模块是指接口相对于它所提供的功能来说比较复杂的模块。浅模块在与复杂性的斗争中没有多大帮助，因为它们提供的好处(不需要了解它们内部是如何工作的)被学习和使用它们的接口的成本所抵消。小模块往往是浅的。


- A shallow module is one whose interface is complicated relative to the functionality it provides. Shallow modules don’t help much in the battle against complexity, because the benefit they provide (not having to learn about how they work internally) is negated by the cost of learning and using their interfaces. Small modules tend to be shallow.


**Classitis**


在这里便论述了一定的矛盾性，深度课程的价值在今天并没有被广泛认识到。编程中的传统观点是类应该很小，而不是很深。学生们经常被教导，课堂设计中最重要的事情是把大班分成小班。关于方法也经常给出同样的建议:“任何超过N行的方法都应该被分成多个方法”(N可以低至10行)。这种方法导致大量的浅类和方法，这增加了整个系统的复杂性。


- Unfortunately, the value of deep classes is not widely appreciated today. The conventional wisdom in programming is that classes should be *small*, not deep. Students are often taught that the most important thing in class design is to break up larger classes into smaller ones. The same advice is often given about methods: “Any method longer than N lines should be divided into multiple methods” (N can be as low as 10). This approach results in large numbers of shallow classes and methods, which add to overall system complexity.


**Examples: Java and Unix I/O**


接口的设计应该使一般情况尽可能简单


**interfaces should be designed to make the common case as simple as possible**
在这里举了一个例子，首先我为这个方法设置默认常用的接口，例如I/O，几乎每个文件I/O的用户都需要缓冲，因此应该在默认情况下提供缓冲。对于那些不需要缓冲的少数情况，库可以提供一种机制来禁用它。任何禁用缓冲的机制都应该在接口中清晰地分开(例如，通过为FileInputStream提供不同的构造函数，或者通过禁用或替换缓冲机制的方法)，这样大多数开发人员甚至不需要知道它的存在。
- Information leakage occurs when the same knowledge is used in multiple places, such as two different classes that both understand the format of a particular type of file.

**Temporal decomposition**

信息泄漏的一个常见原因是一种我称之为步骤分解的设计风格。在步骤分解中，系统的结构对应于操作发生的时间顺序。考虑这样一个应用程序:读取特定格式的文件，修改文件的内容，然后再次将文件写出来。

- One common cause of information leakage is a design style I call *temporal decomposition*. In temporal decomposition, the structure of a system corresponds to the time order in which operations will occur. Consider an application that reads a file in a particular format, modifies the contents of the file, and then writes the file out again.

在设计模块时，应关注执行每个任务所需的知识，而不是任务发生的顺序。

**When designing modules, focus on the knowledge that’s needed to perform each task, not the order in which tasks occur.**

**Red Flag: Temporal Decomposition**（需要标注小红旗）

在步骤分解中，执行顺序反映在代码结构中:不同时间发生的操作在不同的方法或类中。如果在不同的执行点使用相同的知识，它会在多个地方被编码，导致信息泄漏。

- In temporal decomposition, execution order is reflected in the code structure: operations that happen at different times are in different methods or classes. If the same knowledge is used at different points in execution, it gets encoded in multiple places, resulting in information leakage.

信息隐藏通常可以通过使类稍大一些来改进。

**information hiding can often be improved by making a class slightly larger**

默认值说明了这样一个原则，即接口应该被设计成使常见情况尽可能简单。它们也是部分信息隐藏的一个例子:在正常情况下，调用者不需要知道默认项的存在。在很少的情况下，调用者需要覆盖默认值，它将必须知道值，它可以调用一个特殊的方法来修改它。

- Defaults illustrate the principle that interfaces should be designed to make the common case as simple as possible. They are also an example of partial information hiding: in the normal case, the caller need not be aware of the existence of the defaulted item. In the rare cases where a caller needs to override a default, it will have to know about the value, and it can invoke a special method to modify it.

**Red Flag: Overexposure** （需要标注小红旗）

如果一个常用特性的API迫使用户了解很少使用的其他特性，这会增加不需要这些很少使用的特性的用户的认知负担。

- If the API for a commonly used feature forces users to learn about other features that are rarely used, this increases the cognitive load on users who don’t need the rarely used features.

**Conclusion**

信息隐藏与深度模块密切相关。如果一个模块隐藏了很多信息，这往往会增加该模块提供的功能数量，同时也会减少其接口。这使得模块更加深入。相反，如果一个模块没有隐藏太多信息，那么它要么没有多少功能，要么有一个复杂的界面;无论哪种方式，模块都是浅的。

- Information hiding and deep modules are closely related. If a module hides a lot of information, that tends to increase the amount of functionality provided by the module while also reducing its interface. This makes the module deeper. Conversely, if a module doesn’t hide much information, then either it doesn’t have much functionality, or it has a complex interface; either way, the module is shallow.

在将系统分解成模块时，尽量不受运行时操作顺序的影响;这会让你走上步骤分解的道路，这将导致信息泄露和浅模块。相反，请考虑执行应用程序任务所需的不同知识片段，并设计每个模块来封装这些知识片段中的一个或几个。这将产生一个干净和简单的设计与深度模块。
- Almost every user of file I/O will want buffering, so it should be provided by default. For those few situations where buffering is not desirable, the library can provide a mechanism to disable it. Any mechanism for disabling buffering should be cleanly separated in the interface (for example, by providing a different constructor for FileInputStream, or through a method that disables or replaces the buffering mechanism), so that most developers do not even need to be aware of its existence.

**Conclusion**

通过将模块的接口与其实现分离，我们可以对系统的其余部分隐藏实现的复杂性。模块的用户只需要理解它的接口提供的抽象。在设计类和其他模块时，最重要的问题是要使它们更深入，这样它们就有了通用用例的简单接口，但仍然提供重要的功能。这将隐藏的复杂性最大化。

- By separating the interface of a module from its implementation, we can hide the complexity of the implementation from the rest of the system. Users of a module need only understand the abstraction provided by its interface. The most important issue in designing classes and other modules is to make them deep, so that they have simple interfaces for the common use cases, yet still provide significant functionality. This maximizes the amount of complexity that is concealed.

#### **Information Hiding (and Leakage)**(信息隐藏和泄露)

**Information Hiding**

实现深度模块最重要的技术是信息隐藏。这种技术最先由大卫·帕纳斯(David Parnas)描述。基本思想是每个模块都应该封装一些代表设计决策的知识。知识被嵌入到模块的实现中，但不会出现在它的接口中，因此对其他模块不可见。

- The most important technique for achieving deep modules is *information hiding*. This technique was first described by David Parnas[1](https://www.neat-reader.cn/part0008.html#fn1). The basic idea is that each module should encapsulate a few pieces of knowledge, which represent design decisions. The knowledge is embedded in the module’s implementation but does not appear in its interface, so it is not visible to other modules.

信息隐藏通过两种方式降低了复杂性。首先，它简化了模块的接口。该接口反映了一个更简单、更抽象的模块功能视图，并隐藏了细节;这减少了使用该模块的开发人员的认知负荷。

- Information hiding reduces complexity in two ways. First, it simplifies the interface to a module. The interface reflects a simpler, more abstract view of the module’s functionality and hides the details; this reduces the cognitive load on developers who use the module.

第二，信息隐藏使系统更容易发展。如果隐藏了一段信息，则该信息在包含该信息的模块之外不存在依赖关系，因此与该信息相关的设计更改将只影响一个模块

- Second, information hiding makes it easier to evolve the system. If a piece of information is hidden, there are no dependencies on that information outside the module containing the information, so a design change related to that information will affect only the one module.

**Information leakage**

信息隐藏的对立面是信息泄露。当设计决策反映在多个模块中时，就会发生信息泄漏。这就创建了模块之间的依赖关系:对该设计决策的任何更改都需要对所有相关模块进行更改。如果一条信息反映在模块的接口中，那么根据定义，它已经被泄漏了

- The opposite of information hiding is *information leakage*. Information leakage occurs when a design decision is reflected in multiple modules. This creates a dependency between the modules: any change to that design decision will require changes to all of the involved modules. If a piece of information is reflected in the interface for a module, then by definition it has been leaked

**Red Flag: Information Leakage**（需要标注小红旗）

信息泄漏是软件设计中最重要的危险信号之一。作为一名软件设计师，你能学到的最好的技能之一就是对信息泄露的高度敏感。如果您遇到类之间的信息泄漏，请问自己“我如何重新组织这些类，使这一特定知识只影响单个类?”“如果受影响的类别相对较小，且与泄露的信息密切相关，那么将它们合并为一个类别可能是有意义的。另一种可能的方法是从所有受影响的类中提取信息，然而，这种方法只有在您能够找到一个简单的接口，从细节中抽象出来时才有效;如果新类通过它的接口暴露了大部分知识，那么它就不会提供太多的价值(您只是简单地将后门泄漏替换为通过接口的泄漏)。

- Information leakage is one of the most important red flags in software design. One of the best skills you can learn as a software designer is a high level of sensitivity to information leakage. If you encounter information leakage between classes, ask yourself “How can I reorganize these classes so that this particular piece of knowledge only affects a single class?” If the affected classes are relatively small and closely tied to the leaked information, it may make sense to merge them into a single class. Another possible approach is to pull the information out of all of the affected classes and create a new class that encapsulates just that information. However, this approach will be effective only if you can find a simple interface that abstracts away from the details; if the new class exposes most of the knowledge through its interface, then it won’t provide much value (you’ve simply replaced back-door leakage with leakage through an interface).

当在多个地方使用相同的知识时，例如两个不同的类都理解特定类型文件的格式，就会发生信息泄漏。
- When decomposing a system into modules, try not to be influenced by the order in which operations will occur at runtime; that will lead you down the path of temporal decomposition, which will result in information leakage and shallow modules. Instead, think about the different pieces of knowledge that are needed to carry out the tasks of your application, and design each module to encapsulate one or a few of those pieces of knowledge. This will produce a clean and simple design with deep modules.

#### **General-Purpose Modules are Deeper**（通用模块更深入）

**Make classes somewhat general-purpose**

根据我的经验，最好的办法是以某种通用的方式实现新模块。短语“有点通用”意味着模块的功能应该反映你当前的需求，但它的接口不应该。相反，接口应该足够通用，以支持多种用途。该接口应该易于使用，以满足当今的需求，而不需要专门绑定到它们。“多少”这个词很重要:不要失去控制，不要构建如此通用的东西，以至于难以满足您当前的需求。

- In my experience, the sweet spot is to implement new modules in a *somewhat general-purpose*fashion. The phrase “somewhat general-purpose” means that the module’s functionality should reflect your current needs, but its interface should not. Instead, the interface should be general enough to support multiple uses. The interface should be easy to use for today’s needs without being tied specifically to them. The word “somewhat” is important: don’t get carried away and build something so general-purpose that it is difficult to use for your current needs.

**A more general-purpose API**

这里通过举例GUI文本编辑器的插入删除方法，来表现了实现通用方法的简便，即开发者或者用户只需要关注部分信息（因为已经实现了通用方法），而不是因为每一个接口都需要去查询它的使用方法。

**Generality leads to better information hiding**

通用方法在文本和用户界面类之间提供了更清晰的分离，从而更好地隐藏信息。text类不需要知道用户界面的细节，比如如何处理退格键;这些细节现在封装在user interface类中。无需在text类中创建新的支持函数，就可以添加新的用户界面特性。通用界面还减少了认知负担:开发用户界面的人员只需要学习一些简单的方法，这些方法可以被多种方法重用

- The general-purpose approach provides a cleaner separation between the text and user interface classes, which results in better information hiding. The text class need not be aware of specifics of the user interface, such as how the backspace key is handled; these details are now encapsulated in the user interface class. New user interface features can be added without creating new supporting functions in the text class. The general-purpose interface also reduces cognitive load: a developer working on the user interface only needs to learn a few simple methods, which can be reused for a variety of purposes.

软件设计最重要的元素之一是确定谁需要知道什么以及何时需要知道什么。当细节很重要时，最好使它们尽可能明确和明显，例如退格操作的修改实现。将这些信息隐藏在接口后面只会造成模糊。

- One of the most important elements of software design is determining who needs to know what, and when. When the details are important, it is better to make them explicit and as obvious as possible, such as the revised implementation of the backspace operation. Hiding this information behind an interface just creates obscurity.

**Questions to ask yourself**（to build general-purpose）

**What is the simplest interface that will cover all my current needs?**

如果您减少了API中的方法数量，但又不降低其整体功能，那么您可能创建了更多通用的方法。

- If you reduce the number of methods in an API without reducing its overall capabilities, then you are probably creating more general-purpose methods.

**In how many situations will this method be used?**

如果一个方法是为一个特定的用途设计的，比如退格方法，这是一个危险信号，说明它可能用途太特殊了。看看是否可以用一个通用方法替换多个特殊用途的方法。

- If a method is designed for one particular use, such as the backspace method, that is a red flag that it may be too special-purpose. See if you can replace several special-purpose methods with a single general-purpose method.

**Is this API easy to use for my current needs?**

这个问题可以帮助您确定，在使API变得简单和通用方面做得太过了。如果您必须编写大量额外的代码来使用一个用于当前目的的类，这是一个危险信号，表明该接口没有提供正确的功能。

- This question can help you to determine when you have gone too far in making an API simple and general-purpose. If you have to write a lot of additional code to use a class for your current purpose, that’s a red flag that the interface doesn’t provide the right functionality.

**Conclusion**

通用接口比专用接口有许多优点。它们往往更简单，更深入的方法更少。它们还提供了类之间更清晰的分离，而专用接口往往会在类之间泄漏信息。让您的模块具有一定的通用性是降低整个系统复杂性的最佳方法之一。

- General-purpose interfaces have many advantages over special-purpose ones. They tend to be simpler, with fewer methods that are deeper. They also provide a cleaner separation between classes, whereas special-purpose interfaces tend to leak information between classes. Making your modules somewhat general-purpose is one of the best ways to reduce overall system complexity.

#### **Different Layer, Different Abstraction**(不同的层次，不同的抽象)

软件系统由层组成，上层使用下层提供的设施。在一个设计良好的系统中，每一层都提供了不同于它上面和下面的层的抽象;如果您通过调用方法来跟踪单个操作在各个层中上下移动，那么每个方法调用都会改变抽象。

- Software systems are composed in layers, where higher layers use the facilities provided by lower layers. In a well-designed system, each layer provides a different abstraction from the layers above and below it; if you follow a single operation as it moves up and down through layers by invoking methods, the abstractions change with each method call.

这里举个例子（illustrate）

在文件系统中，最上层实现文件抽象。文件由可变长度的字节数组组成，可以通过读写可变长度的字节范围来更新该数组。文件系统的下一层实现固定大小磁盘块的内存缓存;调用者可以假定经常使用的块将保存在内存中，可以快速访问它们。最低层由设备驱动程序组成，它们在二级存储设备和内存之间移动块。

- In a file system, the uppermost layer implements a file abstraction. A file consists of a variable-length array of bytes, which can be updated by reading and writing variable-length byte ranges. The next lower layer in the file system implements a cache in memory of fixed-size disk blocks; callers can assume that frequently used blocks will stay in memory where they can be accessed quickly. The lowest layer consists of device drivers, which move blocks between secondary storage devices and memory.

如果一个系统包含具有类似抽象的相邻层，这是一个警告信号，表明类分解存在问题。本章将讨论这种情况，由此产生的问题，以及如何重构以消除这些问题。

If a system contains adjacent layers with similar abstractions, this is a red flag that suggests a problem with the class decomposition. This chapter discusses situations where this happens, the problems that result, and how to refactor to eliminate the problems.

**Pass-through methods**

当相邻层具有类似的抽象时，问题通常以传递方法的形式表现出来。传递方法是指除了调用另一个方法(其签名与调用方法相似或相同)之外几乎不做其他事情的方法。

- When adjacent layers have similar abstractions, the problem often manifests itself in the form of *pass-through methods*. A pass-through method is one that does little except invoke another method, whose signature is similar or identical to that of the calling method.

**Red Flag: Pass-Through Method**(需要标注小红旗)

传递方法是一种除了将其参数传递给另一个方法之外什么也不做的方法，通常使用与传递方法相同的API。这通常表明类之间没有明确的职责划分。

A pass-through method is one that does nothing except pass its arguments to another method, usually with the same API as the pass-through method. This typically indicates that there is not a clean division of responsibility between the classes.

**When is interface duplication OK?**

拥有相同签名的方法并不总是不好的。重要的是，每个新方法都应该贡献重要的功能。传递方法是不好的，因为它们没有提供新的功能。

- Having methods with the same signature is not always bad. The important thing is that each new method should contribute significant functionality. Pass-through methods are bad because they contribute no new functionality.

只要每个方法都提供有用的和不同的功能，那么多个方法拥有相同的签名是很好的。由调度程序调用的方法具有此属性。另一个例子是具有多种实现的接口

- It is fine for several methods to have the same signature as long as each of them provides useful and distinct functionality. The methods invoked by a dispatcher have this property. Another example is interfaces with multiple implementations.

**Decorators**

装饰器设计模式(也称为“包装器”)鼓励跨层复制API。decorator对象接受现有对象并扩展其功能;它提供了与基础对象类似或相同的API，其方法调用基础对象的方法。

- The decorator design pattern (also known as a “wrapper”) is one that encourages API duplication across layers. A decorator object takes an existing object and extends its functionality; it provides an API similar or identical to the underlying object, and its methods invoke the methods of the underlying object.

装饰器的动机是将类的特殊用途扩展与更通用的核心分离开来。然而，装饰器类往往很肤浅:它们为少量的新功能引入了大量的样板。装饰器类通常包含许多传递方法。很容易过度使用装饰器模式，为每个小的新特性创建一个新类。这导致了浅类的激增，比如Java I/O示例。

- The motivation for decorators is to separate special-purpose extensions of a class from a more generic core. However, decorator classes tend to be shallow: they introduce a large amount of boilerplate for a small amount of new functionality. Decorator classes often contain many pass-through methods. It’s easy to overuse the decorator pattern, creating a new class for every small new feature. This results in an explosion of shallow classes, such as the Java I/O example.

**Interface versus implementation**

“不同的层，不同的抽象”规则的另一个应用是，类的接口通常应该不同于它的实现:内部使用的表示应该不同于接口中出现的抽象。如果两者有相似的抽象，那么这个类可能不是很深。

- Another application of the “different layer, different abstraction” rule is that the interface of a class should normally be different from its implementation: the representations used internally should be different from the abstractions that appear in the interface. If the two have similar abstractions, then the class probably isn’t very deep.

**Pass-through variables**

传递变量增加了复杂性，因为它们迫使所有中间方法知道它们的存在，即使这些方法对变量没有用处。此外，如果出现了一个新的变量(例如，系统最初构建时不支持证书，但您后来决定添加该支持)，您可能必须修改大量接口和方法，以通过所有相关路径传递该变量。

- Pass-through variables add complexity because they force all of the intermediate methods to be aware of their existence, even though the methods have no use for the variables. Furthermore, if a new variable comes into existence (for example, a system is initially built without support for certificates, but you later decide to add that support), you may have to modify a large number of interfaces and methods to pass the variable through all of the relevant paths.

通用的解决方案：最常用的解决方案是引入一个上下文对象。上下文存储应用程序的所有全局状态(任何其他情况下可能是传递变量或全局变量的内容)。大多数应用程序在其全局状态中有多个变量，表示诸如配置选项、共享子系统和性能计数器等内容。系统的每个实例有一个上下文对象。上下文允许系统的多个实例共存于单个进程中，每个实例都有自己的上下文。

- The solution I use most often is to introduce a *context* object. A context stores all of the application’s global state (anything that would otherwise be a pass-through variable or global variable). Most applications have multiple variables in their global state, representing things such as configuration options, shared subsystems, and performance counters. There is one context object per instance of the system. The context allows multiple instances of the system to coexist in a single process, each with its own context.

**Conclusion**

添加到系统中的每个设计基础设施，比如接口、参数、函数、类或定义，都增加了复杂性，因为开发人员必须了解这个元素。为了让一个元素提供相对于复杂性的净收益，它必须消除一些在没有设计元素时可能出现的复杂性。否则，您最好在没有特定元素的情况下实现系统。例如，一个类可以通过封装功能来降低复杂性，这样类的用户就不需要知道它。

- Each piece of design infrastructure added to a system, such as an interface, argument, function, class, or definition, adds complexity, since developers must learn about this element. In order for an element to provide a net gain against complexity, it must eliminate some complexity that would be present in the absence of the design element. Otherwise, you are better off implementing the system without that particular element. For example, a class can reduce complexity by encapsulating functionality so that users of the class needn’t be aware of it.

“不同的层，不同的抽象”规则只是这一思想的应用:如果不同的层具有相同的抽象，例如传递方法或装饰器，那么很有可能它们没有提供足够的好处来弥补它们所代表的额外基础设施。类似地，传递参数要求几个方法中的每个方法都知道它们的存在(这增加了复杂性)，而不需要提供额外的功能。

- The “different layer, different abstraction” rule is just an application of this idea: if different layers have the same abstraction, such as pass-through methods or decorators, then there’s a good chance that they haven’t provided enough benefit to compensate for the additional infrastructure they represent. Similarly, pass-through arguments require each of several methods to be aware of their existence (which adds to complexity) without contributing additional functionality.