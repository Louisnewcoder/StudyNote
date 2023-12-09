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