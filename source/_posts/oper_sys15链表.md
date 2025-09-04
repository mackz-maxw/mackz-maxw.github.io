---
title: 操作系统基础 | 4.1 内核数据结构-链表
categories: os basic
lang: zh-CN
---

### 链表（Linked Lists）

#### 单链表与双链表

- **单链表**：每个节点只包含指向下一个节点的指针。
  ```c
  struct list_element {
      void *data;                   // 数据
      struct list_element *next;    // 指向下一个节点
  };
  ```
- **双链表**：每个节点包含指向前一个和后一个节点的指针。
  ```c
  struct list_element {
      void *data;                   // 数据
      struct list_element *next;    // 指向下一个节点
      struct list_element *prev;    // 指向前一个节点
  };
  ```

#### 循环链表（Circular Linked Lists）

- 普通链表的最后一个节点的 next 指针通常指向 NULL，表示结束。
- 循环链表的最后一个节点的 next 指针指向第一个节点，形成环状结构。循环链表可以是单向或双向的。
- Linux 内核的链表实现本质上是循环双链表，灵活性最高。

#### 链表的遍历

- 链表的遍历是线性的：从头节点开始，依次通过 next 指针访问每个节点。
- 非循环链表的最后一个节点 next 为 NULL；循环链表的最后一个节点 next 指向头节点。
- 双链表可以支持从尾节点向前遍历。
- 链表适合需要频繁插入、删除和遍历全部元素的场景，不适合随机访问。



### Linux 内核链表实现方式

#### 1. 与传统链表的区别

- 传统链表通常是在数据结构里直接加 `next` 和 `prev` 指针，把结构本身变成链表节点。
- Linux 内核采用**嵌入链表节点**的方式：在自定义结构体里嵌入一个 `struct list_head` 成员，而不是直接用 `next`/`prev` 指针。

#### 2. 内核链表节点结构

```c
struct list_head {
    struct list_head *next;
    struct list_head *prev;
};
```
- 只包含指向前后节点的指针，不存储数据。

#### 3. 如何使用

- 在你的结构体里嵌入 `struct list_head` 成员，例如：
  ```c
  struct fox {
      unsigned long tail_length;
      unsigned long weight;
      bool is_fantastic;
      struct list_head list; // 链表节点
  };
  ```
- 这样，`fox.list.next` 指向下一个 fox，`fox.list.prev` 指向上一个 fox。

#### 4. 链表操作

- 内核提供了丰富的链表操作函数（如 `list_add()`），这些函数只操作 `list_head`，不关心具体数据类型。
- 通过 `container_of` 宏，可以从 `list_head` 指针反查到包含它的结构体：
  ```c
  #define container_of(ptr, type, member) \
      ((type *)((char *)(ptr) - offsetof(type, member)))
  #define list_entry(ptr, type, member) \
      container_of(ptr, type, member)
  ```
- 用 `list_entry()` 可以从链表节点指针获取到完整的结构体数据。

#### 5. 链表初始化

- 动态分配结构体后，用 `INIT_LIST_HEAD(&obj->list)` 在运行时初始化链表节点。
```cpp
struct fox *red_fox; 
red_fox = kmalloc(sizeof(*red_fox), GFP_KERNEL); 
red_fox->tail_length = 40; 
red_fox->weight = 6; 
red_fox->is_fantastic = false; 
INIT_LIST_HEAD(&red_fox->list);
```
- 静态定义时可用 `LIST_HEAD_INIT()` 宏在编译时初始化。
```cpp
 struct fox red_fox = { 
    .tail_length = 40, 
    .weight = 6, 
    .list  = LIST_HEAD_INIT(red_fox.list),
 };
```

#### 6. 链表头

- 通常会定义一个专门的 `list_head` 变量作为链表头，用于管理整个链表：
  ```c
  static LIST_HEAD(fox_list);
  ```
- 这个链表头本质上也是一个 `list_head` 节点，但不存储实际数据，只作为入口。


### Linux 内核链表的操作方法

1. **所有链表操作函数都只操作 `struct list_head` 指针，和具体数据类型无关。**
   - 这些函数都定义在 `<linux/list.h>`，实现为内联函数，效率高。
   - 所有操作都是 O(1) 常数时间，无论链表长度如何，插入、删除等操作速度都一样。

2. **常用操作函数：**

   - **添加节点**（底层的循环链表特性，每一个节点都可以填入head）
     - `list_add(new, head)`：把新节点插入到 head 节点之后（将最后一个节点填入head，实现栈）。
     - `list_add_tail(new, head)`：把新节点插入到 head 节点之前（将第一个节点填入head，实现队列）。

   - **删除节点**
     - `list_del(entry)`：把 entry 节点从链表中移除，但不释放内存。
     - `list_del_init(entry)`：移除节点并重新初始化它，方便后续复用。

   - **移动节点**
     - `list_move(list, head)`：把节点 list 移到 head 节点之后。
     - `list_move_tail(list, head)`：把节点 list 移到 head 节点之前。

   - **拼接链表**
     - `list_splice(list, head)`：把 list 指向的链表拼接到 head 节点之后。
     - `list_splice_init(list, head)`：拼接后把原链表重新初始化。

   - **判断链表是否为空**
     - `list_empty(head)`：判断链表是否为空，返回非零表示空。

3. **内部函数优化**
   - 如果你已经有了节点的 next 和 prev 指针，可以直接调用内部函数（如 `__list_del(prev, next)`），省去多余的指针解引用。
   - 这些内部函数一般以双下划线 `__` 开头，只有在你已经拿到指针时才建议用。

### Linux 内核链表的遍历
完整遍历包含n个节点的链表是O(n)时间复杂度操作。

#### 基础遍历方法
最基础的遍历宏是`list_for_each()`，接收两个`list_head`参数：
- **第一个参数**：指向当前项的指针（需调用者提供的临时变量）
- **第二个参数**：要遍历的链表头节点

```c
struct list_head *p;
list_for_each(p, fox_list) {
    /* p指向链表中的某个节点 */
}
```

但仅获取`list_head`指针通常无用，我们需要的是包含该链表节点的结构体（前文已讨论）指针。此时应使用`list_entry()`宏：

```c
struct list_head *p;
struct fox *f;
list_for_each(p, &fox_list) {
    f = list_entry(p, struct fox, list); // 获取包含list_head的fox结构体
}
```

#### 实用遍历方法
上述方法不够直观，因此内核主要使用`list_for_each_entry()`宏，该宏自动完成`list_entry()`转换：

```c
list_for_each_entry(pos, head, member)
```
参数说明：
- `pos`：包含`list_head`的结构体指针（相当于`list_entry()`返回值）
- `head`：链表头节点指针（如`fox_list`）
- `member`：`list_head`在结构体中的成员名（如`list`）

示例：
```c
struct fox *f;
list_for_each_entry(f, &fox_list, list) {
    /* 每次迭代f指向下一个fox结构体 */
}
```

**实际案例**（来自内核文件系统通知系统inotify）：
```c
static struct inotify_watch *inode_find_handle(struct inode *inode, 
                                              struct inotify_handle *ih)
{
    struct inotify_watch *watch;
    list_for_each_entry(watch, &inode->inotify_watches, i_list) {
        if (watch->ih == ih)
            return watch;
    }
    return NULL;
}
```
此函数遍历`inode->inotify_watches`链表，查找匹配`inotify_handle`的节点。

#### 反向遍历
`list_for_each_entry_reverse()`功能与正向遍历相同，但沿`prev`指针逆向移动：
```c
list_for_each_entry_reverse(pos, head, member)
```
适用场景：
1. 性能优化：当目标节点更靠近链表尾部时
2. 顺序要求：如实现LIFO栈操作
若无特殊需求，建议使用正向遍历。

#### 安全删除遍历
标准遍历方法在迭代过程中删除节点会导致问题（后续迭代无法获取正确的next/prev指针）。内核提供安全版本：
```c
list_for_each_entry_safe(pos, next, head, member)
```
参数`next`用于临时存储下一节点指针，确保当前节点可安全删除。

**inotify示例**：
```c
void inotify_inode_is_dead(struct inode *inode)
{
    struct inotify_watch *watch, *next;
    mutex_lock(&inode->inotify_mutex);
    list_for_each_entry_safe(watch, next, &inode->inotify_watches, i_list) {
        struct inotify_handle *ih = watch->ih;
        mutex_lock(&ih->mutex);
        inotify_remove_watch_locked(ih, watch); // 删除watch节点
        mutex_unlock(&ih->mutex);
    }
    mutex_unlock(&inode->inotify_mutex);
}
```

逆向安全遍历版本：
```c
list_for_each_entry_safe_reverse(pos, n, head, member)
```

#### 注意事项
1. "safe"宏仅防护循环体内的删除操作，若存在并发操作仍需加锁
2. 更多链表操作方法详见`<linux/list.h>`

