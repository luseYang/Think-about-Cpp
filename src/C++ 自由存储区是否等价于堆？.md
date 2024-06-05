# "free store" and "heap"
当谈及 C++ 的内存分布时，通常会想到：
> C++ 中，内存分区为 5 个区，分别为堆，栈，自由存储区，全局/静态存储区，常量存储区。

自由存储区(free store) 和堆区(heap) 有什么区别呢？
> malloc 在堆上分配内存块，使用 free 释放内存，而 new 申请的内存则是在自由存储区上，使用 delete 来释放


**自由存储区是与堆区是两块不同的区域吗？它们有可能相同吗？**
这时候需要知道：所谓的划分自由存储区与堆区的分界线是 new/delete 与 malloc/free ，然而 C++ 没有标准要求，很多编译器的 new/delete 都是以 malloc/free 为基础来实现的。

借以 malloc 实现的 new ，所申请的内存实在堆上还是在自由存储区上？

从技术上来说，堆是 C语言 和操作系统的术语。堆是一块操作系统维护的特殊内存，提供了动态分配的功能，当运行程序调用malloe()时就会从中分配，稍后调用free可把内存交还。  

而自由存储是 C++ 中通过 new 和 delete 动态分配和释放对象的抽象概念，通过 new 来申请的内存区域可称为自由存储区。基本上，所有的 C++ 编译器默认使用堆来实现自由存储，也即是缺省的全局运算符( C++ 中的默认全局内存分配和释放操作符。在 C++ 中，如果没有显式地定义全局的 operator new 和 operator delete 函数，编译器会提供默认的实现，这些默认的实现即为所谓的 "缺省的全局运算符"。) new 和 delete 也许会按照 malloc 和 free 的方式来被实现，这时藉由 new 运算符分配的对象，说它在堆上也对，说它在自由存储区上也正确。

但程序员也可以通过重载操作符，改用其他内存来实现自由存储，例如全局变量做的对象池，这时自由存储区就区别于堆了。我们所需要记住的就是：
> 堆是操作系统维护的一块内存，而自由存储是C++中通过new与delete动态分配和释放对象的抽象概念。堆与自由存储区并不等价。

## 问题来源
最先我们使用C语言的时候，并没有这样的争议，很明确地知道malloc/free是在堆上进行内存操作。  
而在Herb Sutter的《exceptional C++》中，明确指出了free store（自由存储区） 与 heap（堆） 是有区别的。关于自由存储区与堆是否等价的问题讨论，大概就是从这里开始的：

详细一点：引用 [exceptional C++](http://www.gotw.ca/gotw/009.htm) 中对于 Free Store 和 Heap 的描述：
> Free Store  
  The free store is one of the two dynamic memory areas, allocated/freed by new/delete. Object lifetime can be less than the time the storage is allocated; that is, free store objects can have memory allocated without being immediately initialized, and can be destroyed without the memory being immediately deallocated. During the period when the storage is allocated but outside the object's lifetime, the storage may be accessed and manipulated through a void* but none of the proto-object's nonstatic members or member functions may be accessed, have their addresses taken, or be otherwise manipulated.
  
> Free Store 是两个动态内存区域之一，由 new/delete 分配/释放。对象的生存期可以小于分配存储的时间；也就是说，自由存储对象可以在不立即初始化的情况下分配内存，并且可以在不立即释放内存的情况下销毁该对象。在分配存储但在对象生命周期之外的期间，可以通过 void* 访问和操作存储，但不能访问原始对象的非静态成员或成员函数、获取其地址或以其他方式操作。

> Heap  
  The heap is the other dynamic memory area, allocated/freed by malloc/free and their variants. Note that while the default global new and delete might be implemented in terms of malloc and free by a particular compiler, the heap is not the same as free store and memory allocated in one area cannot be safely deallocated in the other. Memory allocated from the heap can be used for objects of class type by placement-new construction and explicit destruction. If so used, the notes about free store object lifetime apply similarly here.  

> 堆是另一个动态内存区域，由 malloc/free 及其变体分配/释放。请注意，虽然默认的全局 new 和 delete 可能由特定编译器以 malloc 和 free 的形式实现，但堆与自由存储不同，并且在一个区域中分配的内存无法在另一区域中安全地释放。从堆分配的内存可以通过放置新构造和显式销毁来用于类类型的对象。如果这样使用，关于自由存储对象生存期的注释同样适用于此处。

作者也指出，之所以把堆与自由存储区要分开来，是因为在C++标准草案中关于这两种区域是否有联系的问题一直很谨慎地没有给予详细说明，而且特定情况下new和delete是按照malloc和free来实现，或者说是反过来malloc和free是按照new和delete来实现的也没有定论。这两种内存区域的运作方式不同、访问方式不同，所以应该被当成不一样的东西来使用。

# 结论
- 自由存储是C++中通过new与delete动态分配和释放对象的抽象概念，而堆（heap）是C语言和操作系统的术语，是操作系统维护的一块动态分配内存。

- new所申请的内存区域在C++中称为自由存储区。藉由堆实现的自由存储，可以说new所申请的内存区域在堆上。

- 堆与自由存储区还是有区别的，它们并非等价。

但是，随着现代 C++ 的发展，自由存储区的概念并不明确，现在大概已经被删除了，[参考](https://github.com/cplusplus/draft/pull/7041)
