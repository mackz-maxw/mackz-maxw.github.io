---
title: 操作系统基础 | 4.4 内核数据结构-二叉树
categories: os basic
lang: zh-CN
---

### 二叉树

#### 1. 二叉搜索树（BST）核心特性
```plaintext
根节点左子树的所有节点值 < 根节点值
根节点右子树的所有节点值 > 根节点值
所有子树都是二叉搜索树
```
- **操作复杂度**：
  - 查找：O(log n)  
  - 有序遍历：O(n)
- **缺陷**：不平衡树可能导致操作退化到O(n)

#### 2. 自平衡二叉搜索树
```plaintext
叶子节点深度差 ≤ 1   →   树高度 = O(log n)
```
- **平衡机制**：在插入/删除时自动调整结构
- **常见类型**：
  - AVL树（严格平衡）
  - **红黑树**（半平衡，Linux首选）

---

### 红黑树（Red-Black Trees）
#### 六大约束条件
1. 节点非红即黑
2. 叶子节点（NIL）为黑
3. 叶子节点不存储数据
4. 非叶子节点必有双子
5. **红节点的子节点必为黑**（核心约束）
6. 根到任意叶子的黑节点数相同


#### 优势
- 插入/删除只需O(1)次旋转（AVL需O(log n)）
- 查找效率稳定在O(log n)
- 内存开销小（仅1bit存储颜色）

---

### Linux实现（rbtree）
#### 初始化
```c
#include <linux/rbtree.h>
struct rb_root root = RB_ROOT;  // 声明并初始化根节点
```

#### 节点定义
```c
struct my_node {
    int key;
    struct rb_node rb;  // 嵌入红黑树节点
};
```

#### 查找实现（页缓存示例）
```c
struct page *rb_search_page_cache(struct inode *inode, unsigned long offset)
{
    struct rb_node *n = inode->i_rb_page_cache.rb_node;
    while (n) {
        struct page *page = rb_entry(n, struct page, rb_page_cache);
        if (offset < page->offset)
            n = n->rb_left;
        else if (offset > page->offset)
            n = n->rb_right;
        else
            return page;  // 命中
    }
    return NULL;  // 未命中
}
```

#### 插入实现
```c
struct page *rb_insert_page_cache(struct inode *inode, unsigned long offset, 
                                  struct rb_node *node)
{
    struct rb_node **p = &inode->i_rb_page_cache.rb_node;
    struct rb_node *parent = NULL;
    // 查找插入位置
    while (*p) {
        parent = *p;
        struct page *page = rb_entry(parent, struct page, rb_page_cache);
        
        if (offset < page->offset)
            p = &(*p)->rb_left;
        else if (offset > page->offset)
            p = &(*p)->rb_right;
        else
            return page;  // 已存在
    }
    // 执行插入
    rb_link_node(node, parent, p);      // 链接新节点
    rb_insert_color(node, &inode->i_rb_page_cache);  // 重平衡
    return NULL;  // 插入成功
}
```

---

### XArray对比说明
#### 适用场景差异
| **数据结构** | **最佳场景**                  | **内核应用实例**       | **XArray替代性** |
|--------------|-----------------------------|----------------------|------------------|
| 红黑树       | 范围查询/有序遍历            | 进程调度CFS          | ❌ 不可替代       |
| XArray       | 稀疏ID映射/快速点查          | 页缓存/文件描述符    | ✅ 专精领域       |

#### 性能对比
```plaintext
操作        红黑树        XArray
---------------------------------
插入        O(log n)      O(k)  // k=键长
范围查询     O(log n + m)  O(m)  // m=结果数
内存开销     40字节/节点   8字节/条目
```

#### XArray替代红黑树的条件
1. **键为整数类型**（非复杂比较键）
2. **无需有序遍历**
3. **超高并发需求**（XArray内置RCU）
```c
// XArray实现类似功能
void *xa_store(struct xarray *xa, unsigned long index, void *entry);
void *xa_load(struct xarray *xa, unsigned long index);
```

---

### 关键结论
1. **红黑树适用场景**：
   - VFS目录树（`dentry`缓存）
   - 进程调度器（CFS运行队列）
   - EPoll事件管理
   
2. **XArray优先场景**：
   - 文件页缓存（`address_space`）
   - 内存反向映射
   - UID到指针映射

> **迁移建议**：新代码中整数键映射优先采用XArray；复杂键/范围查询仍需红黑树。  
> **性能数据**：XArray在ext4文件系统中减少40%缓存操作耗时（内核5.15测试）