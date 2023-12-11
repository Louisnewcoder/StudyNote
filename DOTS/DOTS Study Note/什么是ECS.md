Entity  -- 是一个索引，一个标识符。 游戏对象的标识符。 System可以用它来操作，通过组件来分配某些属性
Component -- 容器，是数据容器，并不包含逻辑
system -- 负责执行操作的部分。堆具有特性属性（组件）的特定实体执行的操作

DOTS --- 面向数据编程的思想
ECS 是DOTS 的一部分


# ECS专有名词

**Archetype :** 也是标识符， 是所有具有相同Components 组合的实体类型
**Chunk**：是每种 Archetype 所标记的 固定大小的，连续非托管内存块。 每个Chunk固定16k。Chunk中会包含共享同一archetype原型的实体组件数组。如果实体组件数组填充不满，也要有留白。其目的是为了实现并行计算，实现缓存的 pre-fetch。是面向缓存友好的重要概念
**World:** 是一系列 Entity的组合，每一个entity在一个world中是唯一的，受到world中的Entity Manager管理。
**Entity Manager:** 负责创效，销毁，修改，世界中的实体。

**Structural Change:**  所有导致重新组织内存块，或内存块内容的操作都称之为structural change。有两个维度：改变结构，改变内容。 ===***这两个改变只能在主线程中做！！不能在工作线程中做***=== 、这两个操作属于资源密集型操作，效率很差。 比如：删除了某个archetype的一个组件，或者设置，销毁entity实体，设置，share component的值

**当确定没有structual change的情况发生时，就可以bake操作

Query相关的内容将会是重点

相关术语：
![[Pasted image 20231209153128.png]]
![[Pasted image 20231209152956.png]]
![[Pasted image 20231209153013.png]]
![[Pasted image 20231209153028.png]]

![[Pasted image 20231209153046.png]]

# 工具包与环境

## package
### 核心包
Entities -- 安装完成后，会在editor的 preferences里面多出 一个 Entities 选项标签，以及一个 Jobs 选项标签
![[Pasted image 20231209154326.png]]
![[Pasted image 20231209154412.png]]



# Jobs System -- Unity

Unity Jobs System 下：氛围 C# Jobs System 与 引擎内部C++ Jobs System
它利用多喝计算平台来简单安全的编写与执行多线程代码
可以单独使用，也可以与ECS结合使用
不需要关心运行平台的CPU情况和线程资源情况

Jobs System 是通过**创建Job 任务来管理代码，而不是创建线程**。Unity引擎内部会跨多个核心来管理一组工作线程，以避免上下文的切换。执行时， Jobs System 会将我们创建的Job任务放到Job队列中来等待执行。工作线程会从Job队列中获取Job任务并执行响应的工作。

我们要学的是 C#部分的Jobs System

一般情况下（有可能会出现），在C# 部分的 Jobs System中不会出现
Race Conditions的情况  <--- 什么是 ===Race Conditions=== 据说手写代码的时候会遇到
因为每个Job任务中，只会取访问一份数据的copy或者转换一段buffer的所有权给这个Job任务
另外，因为C#部分在Unity引擎内就是使用 C++ JobsSystem系统的代码，所以不会出现引擎与游戏之间的上下文切换开销。但是因为C#与C++的内存管理和值的定义不同，所以在写C# Jobs System的代码时，要区别哪些是 Blittable的类型，那些是 Non-Blittable的类型。
核心区别方式：
==**Blittable 类型的数据，在托管代码和非托管代码在内存中具有相同的表示形式**==
关于托管内存和非托管内存 与托管类型和非托管类型 [[DOTS 相关提到的额外的我不知道的知识点#托管堆数据和非托管堆数据]]
*什么是Blittable [[DOTS 相关提到的额外的我不知道的知识点#Blittable 数据类型]]*
C#下的区分
![[Pasted image 20231209171051.png]]
这里要注意的是 C#中的 bool 占4个自建 C++中1个字节。不一样，需要做转化。 如果没有进行转化，那么程序不会崩溃，但是可能会有异常表现。

在写Job程序时除了使用Blittable类型外，还可以使用C++上的非托管内存堆数据。但是需要使用另一个DOTS核心包 Collections.NativeContainer,以及Unity为特殊用途下特殊内容，特殊定制的一些类型
![[Pasted image 20231209172927.png]]
这些容器都是非托管堆上分配的
![[Pasted image 20231209173030.png]]
这些类型容器在创建时还会涉及分配类型的设置
![[Pasted image 20231209173140.png]]

## 其他Job相关的知识

### 优化
1 如果Jobs的代码采用dots框架下的 Unity.Mathematics核心包中的函数进行编写的话，会被编译器优化

### Job类型
Job有不同的类型

### Job的调度方式
Run方式 ---- 主线程立即顺序执行
Schedule方式 ---- 单个工作线程或者主线程按每个Job的顺序，一次执行
ScheduleParallel ---- 再多个工作线程上同时执行，性能最好。 ==但是多个工作线程访问同一数据时可能胡i发生冲突==

另外，本身名字带有 Parallel的Job类型，只能通过Schedule方式调用，但是功能与ScheduleParallel调度的方式是一样的。这类Job任务会在多个工作线程上同时执行。 *这类Job类型，也不会提供Run和ScheduleParallel的调度方式。*

### Job Dependencies
当两个Job同时（即使顺序由先后）访问同一个数据并试图堆该数据进行操作时，会产生 Race Condition 的情况。Jobs System的安全检查就会因为这种冲突抛出异常
![[Pasted image 20231210191203.png]]

为了解决这种冲突， 可以再调度Job B前 将 Job A 进行 Complete()。但是这样不够高效。
更好的方式是，让理应限制性的Job成为后执行的依赖。==*所谓的依赖，就是 Job执行顺序的先后逻辑条件。 应该先完成的Job是 被依赖方， 需要等待其他Job完成的Job是依赖方*==
使用 Handle 类型，将JobA的Handle句柄传入 JobB，使 Job A成为Job B的依赖
![[Pasted image 20231210191558.png]]
代码示例 情景1: 链式依赖
![[Pasted image 20231210191727.png]]

代码示例 情景2: 多个Job 依赖同一个Job
![[Pasted image 20231210191832.png]]

代码示例 情景3: 一个Job 依赖多个Job
![[Pasted image 20231210191919.png]]
注: 当需要多个依赖同时存在时，需要用 CombineDependencies()构建一个虚拟的JobHandle

===***重要！！！
Job依赖必须是无环的， 否则Jobs System的安全检查会报错***===

### Job Safety Checks
它会进行：
数据访问冲突检查；
数据访问越界检查；
Jobs依赖关系检查；
内存释放泄露检查；
如果开启全局安全检查，Jobs System会在出现race condition的情况时抛出异常。仅在编辑器环境下执行。
![[Pasted image 20231210193032.png]]
==但是这种安全检查相对严格，比如当 两个Job 访问同一个数据(NativeArray)的不同切片时，这种没有冲突的情况也会抛出异常==
如果开发者确定某些情况下不会产生 冲突，则可以使用 关闭安全检查标签，来避免这样的问题发生
![[Pasted image 20231210220627.png]]
**如果能确定某些数据（Native Array）不会进行任何写入操作的话**，更好的方式是，是使用 `[ReadOnly] 标签
