---
title: 操作系统基础 | 4.2 内核数据结构-队列
categories: os basic
lang: zh-CN
---

### 队列  
任何操作系统内核中常见的编程模式是**生产者与消费者**。实现此模式的最简单方式通常是使用队列，即**先进先出**（FIFO）。 

Linux内核的通用队列实现称为`kfifo`，代码位于`kernel/kfifo.c`，头文件为`<linux/kfifo.h>`。本节讨论2.6.33版本更新后的API（早期版本用法略有不同，编写代码前请确认头文件）。  

---

#### kfifo  
Linux的`kfifo`与其他队列抽象类似，提供两个核心操作：  
- **入队**（`in`）：将数据写入队列  
- **出队**（`out`）：从队列中读取数据  

`kfifo`对象维护两个偏移量：  
- **in偏移量**：下一次入队的起始位置  
- **out偏移量**：下一次出队的起始位置  

**规则**：  
1. `out`偏移量始终≤`in`偏移量（否则会读取未入队的数据）。  
2. 当`out == in`时，队列为空（无法出队）。  
3. 当`in`到达队列末尾时，需重置队列才能继续入队。  

---

#### 创建队列  
##### 动态初始化（常用）  
```c
int kfifo_alloc(struct kfifo *fifo, unsigned int size, gfp_t gfp_mask);
```  
- **功能**：分配大小为`size`字节的队列（`size`必须为2的幂次），内存分配标志为`gfp_mask`（参见第12章“内存管理”）。  
- **返回值**：成功返回`0`，失败返回错误码。  
- **示例**：  
  ```c
  struct kfifo fifo;
  int ret = kfifo_alloc(&fifo, PAGE_SIZE, GFP_KERNEL);
  if (ret)
      return ret; // 初始化失败
  ```

##### 自定义缓冲区  
```c
void kfifo_init(struct kfifo *fifo, void *buffer, unsigned int size);
```  
- **功能**：使用用户提供的`buffer`初始化队列，`size`必须为2的幂次。  

##### 静态声明（较少用）  
```c
DECLARE_KFIFO(name, size);  // 声明队列
INIT_KFIFO(name);           // 初始化队列
```  
- **要求**：`size`必须为2的幂次。  

---

#### 数据入队  
```c
unsigned int kfifo_in(struct kfifo *fifo, const void *from, unsigned int len);
```  
- **功能**：从`from`复制`len`字节到队列`fifo`。  
- **返回值**：实际入队的字节数（若空间不足，可能小于`len`）。  


#### 数据出队  
##### 标准出队  
```c
unsigned int kfifo_out(struct kfifo *fifo, void *to, unsigned int len);
```  
- **功能**：从队列`fifo`复制最多`len`字节到`to`缓冲区。  
- **返回值**：实际出队的字节数。  
- **注意**：出队后数据不再保留在队列中。  

##### 查看数据（不删除）  
```c
unsigned int kfifo_out_peek(struct kfifo *fifo, void *to, unsigned int len, unsigned offset);
```  
- **功能**：与`kfifo_out`类似，但**不移动out偏移量**，数据仍可后续读取。  
- **参数**：`offset`指定队列中的起始位置（0表示队头）。  

---

#### 获取队列信息
1. **获取队列总容量**
```c
static inline unsigned int kfifo_size(struct kfifo *fifo);
```
- 返回队列底层缓冲区的总字节数

2. **获取已入队数据量**
```c
static inline unsigned int kfifo_len(struct kfifo *fifo);
```
- 返回当前队列中已存储的字节数
- （注：内核命名存在不一致性，需特别注意）

3. **获取剩余空间**
```c
static inline unsigned int kfifo_avail(struct kfifo *fifo);
```
- 返回可继续写入的剩余字节数

4. **队列状态检查**
```c
static inline int kfifo_is_empty(struct kfifo *fifo);
static inline int kfifo_is_full(struct kfifo *fifo);
```
- 返回非零值表示队列为空/满
- 返回零表示非空/非满

#### 重置与销毁队列
1. **重置队列**
```c
static inline void kfifo_reset(struct kfifo *fifo);
```
- 清空队列所有内容（不释放内存）

2. **销毁动态分配的队列**
```c
void kfifo_free(struct kfifo *fifo);
```
- 释放通过`kfifo_alloc()`创建的队列
- 注意：使用`kfifo_init()`创建的队列需手动释放关联缓冲区

#### 实际应用示例
创建8KB大小的kfifo队列：
```c
#include <linux/kfifo.h>
struct kfifo my_fifo;  // 声明 kfifo 结构体
int ret;
// 分配 8KB 队列
ret = kfifo_alloc(&my_fifo, 8192, GFP_KERNEL);  // 8192 = 8 * 1024
if (ret) {
    pr_err("Failed to allocate kfifo: error %d\n", ret);
    return ret;
}
// 验证大小
pr_info("Created FIFO size: %u bytes\n", kfifo_size(&my_fifo));  // 输出 8192
```
> 队列总容量 = 8 KB = 8 × 1024 字节 = 8192 字节（环形缓冲区无需减一）
> `unsigned int`大小 = 4 字节（也可能是8字节，可以通过`sizeof(unsigned int)`获取）
> 这样一个队列最多可以容纳8192 字节 / 4 字节 = 2048 个`unsigned int`

**入队操作**：
```c
/* 入队0-31的整数 */
unsigned int i;
for (i = 0; i < 32; i++)
    kfifo_in(fifo, &i, sizeof(i));
```

**查看队首元素（不移除）**：
```c
unsigned int val;
int ret = kfifo_out_peek(fifo, &val, sizeof(val), 0);
if (ret != sizeof(val))
    return -EINVAL;
printk(KERN_INFO "%u\n", val); /* 输出: 0 */
```

**完整出队操作**：
```c
while (kfifo_avail(fifo)) {
    unsigned int val;
    int ret = kfifo_out(fifo, &val, sizeof(val));
    if (ret != sizeof(val))
        return -EINVAL;
    printk(KERN_INFO "%u\n", val); 
}
/* 输出: 0 1 2 ... 31 (严格保持FIFO顺序) */
```
- 输出顺序为0→31证明是标准的FIFO队列
- 若输出为31→0则变为栈结构（LIFO）
- 所有操作均保持原子性，适合生产者-消费者场景
（注：实际开发中通常入队复杂结构体而非基础类型）