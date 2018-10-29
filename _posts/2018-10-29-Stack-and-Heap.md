---
layout:     post                    
title:      基础知识梳理 - Stack 和 Heap
subtitle:   
date:       2018-10-29
author:     Xiongbin Zhao
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - iOS
---

# 基础知识梳理 - Stack 和 Heap

## 一、C 程序的内存结构 
为了能充分了解 Stack 和 Heap, 先看看在一个 C 程序中，内存是如何分布的。

![内存结构](https://cdncontribute.geeksforgeeks.org/wp-content/uploads/memoryLayoutC.jpg)

如图所示，一个 C 程序的内存分布共有5个区:

1. **code/代码区：** 代码区包含程序编译后的所有方法，包括系统的方法。通常以机器码的形式储存。代码区通常是只读的，并且内存大小是固定的。代码区处于内存的地位，以防止 Stack 与 Heap 覆盖了代码区。

2. **initialized data/全局初始化数据区/静态数据区：** 全局初始化数据区可简称为数据区，数据区只初始化一次，用来保存全局与静态变量。进一步划分可划分为 只读数据区 与 读写数据区，例如，当创建 char s[] = "Hello"; 时，字符串的指针 s 会储存于读写数据区，而 "Hello" 字符串会储存于只读数据区。

3. **uninitialized data(bss)/未初始化数据区：** 未初始化数据区用来储存没有赋值的全局与静态变量。例如，global int j; 与 static int i; 都会储存于这个区。

4. **stack/栈：** 栈是一片动态的内存区域，从内存的高位往地位扩张，一般与堆的扩张方向相反。类似于栈的数据结构，具有LIFO的特性。每当一个方法被调用时，一个 **stack frame/栈帧** 就会被压入栈中，**栈帧** 里有所调用方法的返回地址、参数与局部变量等相关信息。当当前方法返回时，相应的栈帧会被销毁。栈内的内存管理都是由系统进行调用和销毁的，所以程序员不需要进行操作。

5. **heap/堆：** **堆**一般处于内存的低位往高位扩张（与**栈**的扩张方向相反）。堆是动态内存区，内存由程序员进行分配管理。开发时可使用 malloc(), calloc(), realloc() 进行内存的分配，并使用 free() 进行内存的销毁。

## Stack 和 Heap 的主要区别
1. 访问速度：stack 的访问速度比 heap 快
2. 内存限制：stack 一般有内存大小限制，而heap没有，除非是硬件的性能限制
3. 作用域：stack 中的变量只会在方法调用时存在，当方法返回时，变量会被销毁。而 heap 中的变量是全局的，能一直访问，直至该内存块被程序员手动销毁。
4. 内存碎片化：Stack 上的内存由 CPU 管理，不会出现内存碎片化的情况。而处于 Heap 上的内存不能保证能充分利用，可能会出现内存碎片化

## 何时使用 Stack 和 Heap
1. 本地变量首选使用 Stack, 因为读写更快，而且内存由系统自动管理。
2. 当需要使用占用内存大的数据结构，使用 Heap。如 array, set 等， 因为 Stack 的内存大小有限制。当使用的内存超出 Stack 可使用范围内，便会出现 StackOverflow.
3. 当需要一个作用域大于当前方法作用域的变量时，使用 Heap。例如，一个方法需要返回一个数组。该数组必须创建于 Heap，因为 Stack 内的局部变量会在返回时被销毁，返回 Stack 上的变量会在编译时报错。
4. 当一个变量的内存大小不确定时，使用 Heap。如 动态数组。同理，因为 stack 上的内存限制，当变量的内存大小不确定时，最好选用 Heap。

## 二、Objective-C 中的 Stack 与 Heap
在 Objective-C 中，NSObject 包含了一个类指针 Class *isa, 里面指向对象类的属性列表、方法列表等信息，得益于 Objective-C 强大的 runtime，运行时可以再对类添加方法、添加属性。

换句话说，NSObject 类在编译时是无法确定其占用内存大小的，因为在运行时可以再对类进行修改，所以理所当然的，所有继承于 NSObject 的类，在新建对象时都是分配在 Heap 上的。而在 Objective-C 中，所有的类都是继承于 NSObject，所以我们新建的对象都处于 Heap 上，必须使用引用计数进行内存管理。

### Block 的内存管理

Objetive-C 中 Block 的内存管理与其他的类稍有不同，以下是 Block 的三个类以及他们的使用场景：（这里讨论的都是使用 ARC 时的情况，使用 MRC 的话会稍有不同）

- **NSConcreateGlobalBlock:** 内存分配在静态数据区，当 Block 不需要访问访问外部变量时。

```
id block = ^(){};
NSLog(@"This is a global block: %@", block);
// This is a global block: <__NSGlobalBlock__: 0x1054480a0>
```

- **NSConcreateStackBlock:** 内存分配在 Stack，当 Block 引用外部变量，而且没有 owner 时，即没有指针引用当前 Block。

```
int b = 100;
NSLog(@"This is a stack block: %@", ^(){
        int a = b;
    });
// This is a stack block: : <__NSStackBlock__: 0x7ffeea7b68d8>
```

- **NSConcreateMallocBlock:** 内存分配在 Heap，当 Block 引用外部变量，并被引用或作为返回值时

```
int b = 100;
id block = ^(){
        int a = b;
    };
NSLog(@"This is a Malloc Block being referenced: %@", block);
// This is a Malloc Block being referenced: <__NSMallocBlock__: 0x600003a193e0>
```

```
- (void(^)(void))block
{
    int b = 100;
    return ^(){
        int a = b;
    };
}
NSLog(@"This is a Malloc Block being returned: %@", [self block]);
// This is a Malloc Block being returned: <__NSMallocBlock__: 0x600003a19140>
```





## 参考资料：
[Memory Layout of C Programs](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

[
C语言内存分配-通俗理解](https://blog.csdn.net/farsight2009/article/details/53082190)

[What's the difference between a stack and a heap?](https://www.programmerinterview.com/index.php/data-structures/difference-between-stack-and-heap/)