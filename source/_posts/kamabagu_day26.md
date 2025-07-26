---
title: 代码随想录 | 八股-c++动态内存分配
categories: comp basic
lang: zh-CN
---

### new和malloc有什么区别
new和malloc在C++中都用于动态内存分配，但它们之间有几个关键的区别：

语法层面：
new是C++的操作符，可以直接用来分配对象或数组。
malloc是一个函数，通常需要包含头文件<cstdlib>，并且只分配原始内存。
类型安全：
new是类型安全的，它会根据分配的对象类型进行正确的内存分配和构造函数调用。
malloc 不是类型安全的，它只分配原始内存，不调用构造函数。返回类型是 void*，需要强制类型转换为具体的指针类型。
构造与析构：
使用 new 分配的对象在对象生命周期结束时需要使用 delete 来释放，delete 会自动调用对象的析构函数。
使用 malloc 分配的内存需要使用 free 来释放，free 不会自动调用析构函数，因此如果分配的是对象数组，需要手动调用析构函数。
异常安全性：
new在分配失败时会抛出std::bad_alloc异常。
malloc在分配失败时返回NULL指针。
管理机制：
C++中的new和delete通常由编译器实现，可能包含一些额外的内存管理机制。
C语言的malloc和free由C标准库提供，与编译器无关。
总结来说，new和malloc都是动态内存分配的手段，但new提供了类型安全和构造/析构的自动化，而malloc则提供了更底层的内存分配方式，需要手动管理构造和析构。在C++中，推荐使用new来分配对象，以保持类型安全和自动化的资源管理。

### delete 和 free 有什么区别？
delete和free都是用来释放动态分配的内存，但它们有不同的使用方式：

语法：

delete是C++中的关键字，用于释放由new分配的对象。
free是C语言中的函数，通常包含在<stdlib.h>头文件中，用于释放由malloc分配的内存。
对象销毁：

当使用 delete 释放对象内存时，C++  编译器会自动调用对象的析构函数，释放与对象相关的资源，并执行对象的清理工作。
free 仅释放内存，不调用析构函数。因此，如果使用 malloc 分配了 C++ 对象的内存，需要手动调用析构函数后再调用 free。
数组处理：

如果是数组，C++提供了delete[]来释放整个数组的内存，而C语言中仍然使用free，没有区分单个对象和数组。
返回值:

free 没有返回值，即使内存释放失败，也不会反馈任何信息。

delete 之后，指针会自动置为 nullptr

类型检查:

delete 进行类型检查，确保删除的对象类型与 new 分配时的类型一致。

free 不进行类型检查，因为它只处理 void* 类型的指针。

总结来说，delete和free都是用来释放动态内存的，但它们分别用于C++和C语言中的内存管理。delete适用于C++对象，会自动调用析构函数；而free适用于C语言分配的内存，不涉及对象的析构。

### 堆区和栈区的区别
堆 （Heap） 和栈 （Stack） 是程序运行时两种不同的内存分配区域

内存分配：

栈
是由编译器自动管理的，用于存储局部变量和函数调用的上下文信息。
栈上的对象在定义它们的块或函数调用结束后自动销毁。
栈的内存分配和释放速度很快，因为栈的内存管理是连续的，不需要搜索空闲内存。
堆
由程序员手动管理的，用于存储动态分配的对象。
堆上的对象需要程序员手动释放，否则可能导致内存泄漏。
堆的内存分配和释放速度通常比栈慢，因为可能需要搜索合适的内存块，并且涉及内存碎片整理。
大小限制：

栈的大小通常有限制，远小于堆的大小，且在不同系统和编译器中可能不同。
堆的大小通常很大，受限于系统可用内存。
使用场景：

栈主要用于存储函数参数、局部变量等。
堆用于存储生存期不受限于单个函数调用的对象，如使用 new 或 malloc 分配的对象。

### 补充： 如何定义只能在堆上或栈上生成对象的类
在 C++ 中，可以通过控制构造函数和析构函数的访问权限来限制对象只能在堆上或栈上创建。以下是两种实现方式的详细说明和代码示例：

#### 1. 只能在堆上创建对象（禁止栈上创建）
**核心原理**：私有析构函数 + 智能指针管理生命周期  
**关键点**：
- 私有析构函数阻止栈对象自动销毁
- 静态工厂函数返回智能指针
- 智能指针自定义删除器调用 `destroy()`
- `destroy()` 调用 `delete this`（需谨慎使用）
- 禁用拷贝构造/赋值

```cpp
#include <memory>

class HeapOnly {
public:
    // 工厂函数：返回带有自定义删除器的 unique_ptr
    static std::unique_ptr<HeapOnly, void(*)(HeapOnly*)> create() {
        return {
            new HeapOnly(), 
            [](HeapOnly* p) { p->destroy(); } // 自定义删除器
        };
    }

    // 销毁函数（必须公开）
    void destroy() {
        delete this; // 调用私有析构函数
    }

    // 禁用拷贝操作
    HeapOnly(const HeapOnly&) = delete;
    HeapOnly& operator=(const HeapOnly&) = delete;

private:
    HeapOnly() = default;       // 私有构造
    ~HeapOnly() = default;      // 私有析构
};

// 使用示例
auto obj = HeapOnly::create();  // 只能在堆上创建
```

#### 2. 只能在栈上创建对象（禁止堆上创建）
**核心原理**：禁用 `new` 操作符

```cpp
class StackOnly {
public:
    StackOnly() = default;
    ~StackOnly() = default;

private:
    // 禁用 new 操作符
    void* operator new(size_t) = delete;
    void* operator new[](size_t) = delete;
    void* operator new(size_t, void* p) = delete; // 禁用 malloc + placement new
};

// 使用示例
StackOnly stackObj;           // 合法：栈上创建
// auto heapObj = new StackOnly(); // 编译错误：new 被禁用
```

> ⚠️ 注意：`delete this` 是特殊技术，必须确保：
> - 对象通过 `new` 创建
> - 调用 `delete this` 后不再访问任何成员
> - 通常只用于精确控制生命周期的场景