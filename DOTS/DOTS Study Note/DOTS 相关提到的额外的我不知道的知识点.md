# Blittable 数据类型
Blittable 类型是一种在内存中具有固定布局和直接对应于硬件表示的数据类型。这些类型可以直接复制到内存的其他位置而无需进行特殊处理，因为它们的内存布局与它们在编程语言中的表示方式相同。

在 C# 中，Blittable 类型具有以下特征：

1. **固定内存布局：** Blittable 类型在内存中的布局是固定的，每个字段的偏移和大小是可预测的。这使得可以通过内存复制来移动这些数据，而不涉及任何特殊的序列化或反序列化过程。
    
2. **直接映射硬件表示：** Blittable 类型的内存布局直接对应于硬件表示。例如，整数、浮点数和其他基本数据类型通常是 Blittable 类型，因为它们的内存表示与硬件上的表示是直接映射的。
    

一些常见的 Blittable 类型包括：

- 基本数据类型（整数、浮点数、字符等）
- 枚举
- 结构体（如果其成员也是 Blittable 类型）

Blittable 类型在许多场景中非常有用，特别是在需要高性能、内存布局可控制或者需要与非托管代码进行交互的情况下。例如，与外部库进行数据交换时，Blittable 类型通常更容易处理。

需要注意的是，Blittable 类型并不适用于所有场景，某些情况下需要考虑内存对齐、字段的顺序等因素。在编写涉及 Blittable 类型的代码时，确保了解所使用类型的内部表示是很重要的。

# 托管堆数据和非托管堆数据
  
在 .NET 环境中，有托管堆内存（Managed Heap Memory）和非托管堆内存（Unmanaged Heap Memory）的概念，同时也存在托管内存数据类型和非托管内存数据类型。

### 托管堆内存（Managed Heap Memory）：

- **定义：** 托管堆是由 .NET 运行时（CLR - Common Language Runtime）负责管理的内存区域，用于存储托管对象（由 CLR 管理的对象）。
    
- **特点：** 托管堆内存的分配和释放由 CLR 自动进行垃圾回收，这意味着开发者不需要手动管理内存。垃圾回收器负责在运行时识别和回收不再使用的对象，以防止内存泄漏。
    
- **托管内存数据类型：** 在托管堆上分配的数据类型是托管数据类型，包括 C# 中的所有基本数据类型、类对象等。这些数据类型的内存管理由 CLR 管理。
    

### 非托管堆内存（Unmanaged Heap Memory）：

- **定义：** 非托管堆是由开发者手动管理的内存区域，不受 CLR 的控制，通常用于与非托管代码或外部库进行交互。
    
- **特点：** 在非托管堆上分配的内存需要手动释放，不受垃圾回收器的管理。这可能导致内存泄漏或野指针等问题，因此需要谨慎使用。
    
- **非托管内存数据类型：** 非托管内存中的数据类型是非托管数据类型，通常是通过使用指针或 API 从系统调用中获得的数据。这些数据类型不受 CLR 管理。
    

在进行托管和非托管内存之间的转换时，可能需要使用 `Marshal` 类等工具来进行数据的序列化和反序列化。托管内存和非托管内存的使用会根据开发场景的不同而有所变化，需要开发者根据具体情况进行选择

### 托管类型：

1. **所有基本数据类型：** `int`、`float`、`double`、`bool` 等基本数据类型都是托管类型。
    
2. **类和对象：** 通过 `class` 关键字定义的类和创建的对象都是托管类型。
    
3. **数组：** C# 中的数组也是托管类型。
    
4. **接口、委托：** 接口和委托等数据类型同样是托管类型。
    
5. **字符串：** 字符串（`string`）是托管类型。
    

### 非托管类型：

1. **指针类型：** 使用 `unsafe` 关键字定义的指针类型允许直接访问内存地址，这部分是与非托管内存交互的一种手段。
    
    csharpCopy code
    
    `unsafe {     int x = 10;     int* ptr = &x; // 指针类型，非托管 }`
    
2. **固定大小的数组：** 固定大小的数组可以使用 `fixed` 关键字，这同样是用于与非托管代码进行交互的手段。
    
    csharpCopy code
    
    `fixed (int* arr = myArray) {     // 操作非托管内存 }`
    
3. **`Marshal` 类型：** `Marshal` 类型提供了一些用于与非托管代码交互的方法，包括内存复制、结构体转换等。
    

总体而言，C# 是一门托管语言，绝大多数情况下开发者不需要直接操作非托管内存。然而，在需要与非托管代码进行交互或执行一些系统级操作时，可以使用上述手段。在这些情况下，需要小心谨慎，确保正确管理内存，以防止内存泄漏和其他问题。


# 什么是Burst， Burst Compile
Unity的一种编译技术，将C#代码编译成高度优化的本地代码(Native代码)，以提高运行时的性能。
基于LLVM技术 Low-Level Virtual Machine，使用SIMD指定加速数学与向量的运算操作。它使用了多种优化技术，入循环展开，常量传播和内联，以生成高效的机器代码。

这里的Native代码是指特定硬件平台上（承载程序运行的平台）相关的机器码。

一般的代码编译过程是，在.Net 平台中C# 一般会被编译成中间代码，然后在运行时进行JIT编译从而转换为Native代码。而使用Burst技术可以将方法或类直接编译为Native代码，避免了JIT编译的性能损失。从而提高了应用程序的性能。

Unity 安装Burst包之后，只需要使用 Burst Attribute取标记要使用Burst编译的代码即可。
```C#
[BusrtCompile]
public int Calculate(int a ,int b)
{
	// example code
	return a+b;
}
```

但是Burst Compile 只适合需要高性能的算发和数学计算的领域，对于复杂的游戏逻辑部分，并不会带来显著的性能提升。===**避免用burst编译包含大量的循环和条件语句等代码结构

**遗憾的是LLVM对C#的GC做的不好，所以Burst只支持值类型的数据编译，不支持引用类型数据编译。**
Support reading materials:
https://zhuanlan.zhihu.com/p/623274986

# 什么是SIMD指令
Single Instruction Multiple Data
单指令多数据流，即可以使用一条指令同时完成多个数据的运算操作。

Support reading materials:
https://zhuanlan.zhihu.com/p/556131141

# 什么是LLVM
Low-Level Virtual Machine
LLVM不仅仅是一个虚拟机，而是一个综合的编译器工具链。LLVM提供了一套通用的工具和库，用于开发编译器、优化器、代码生成器等。
LLVM的核心思想是基于中间表示（Intermediate Representation, IR），它定义了一种与机器和语言无关的中间代码表示形式。LLVM IR是一种**低级别的静态单赋值（Static Single Assignment, SSA）形式，它使用基本块和指令的层次结构来表示程序的结构和行为。

Support reading materials:
https://zhuanlan.zhihu.com/p/629139721