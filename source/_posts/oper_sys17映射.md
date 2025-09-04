---
title: 操作系统基础 | 4.3 内核数据结构-映射
categories: os basic
lang: zh-CN
---

### 映射（Maps）  
#### 基本概念  
映射（又称关联数组）是由唯一键组成的集合，每个键关联一个特定值。键与值的关系称为**映射关系**，支持以下基本操作：  
- **添加**（Add）：插入键值对  
- **移除**（Remove）：删除指定键  
- **查找**（Lookup）：通过键获取值  

尽管哈希表是一种映射实现，但并非所有映射都基于哈希。映射也可使用**自平衡二叉搜索树**存储数据：  
- **哈希表**：平均时间复杂度更优（O(1)），但最坏情况为线性（O(n)）  
- **二叉搜索树**：最坏情况为对数复杂度（O(log n)），且支持有序遍历，无需哈希函数（仅需定义比较操作符）  

在Linux内核中，映射的特定实现称为`idr`（ID Radix Tree-旧版实现，现为XArray），专用于将唯一ID（UID）映射到指针。  

---

#### 初始化idr  
```c
#include <linux/idr.h>

struct idr id_huh;       // 静态定义  
idr_init(&id_huh);      // 初始化  
```

---

#### 分配UID  
##### 1. 预分配资源  
```c
int idr_pre_get(struct idr *idp, gfp_t gfp_mask);  
```  
- **功能**：必要时调整底层树结构，准备分配新UID  
- **参数**：  
  - `idp`：目标idr结构  
  - `gfp_mask`：内存分配标志（如`GFP_KERNEL`）  
- **返回值**：成功返回1，失败返回0（与其他内核函数相反！）  

##### 2. 分配UID并关联指针  
```c
int idr_get_new(struct idr *idp, void *ptr, int *id);  
```  
- **功能**：分配新UID，将其与`ptr`关联  
- **返回值**：  
  - 成功：返回0，UID存储在`id`中  
  - 失败：返回`-EAGAIN`（需重试`idr_pre_get`）或`-ENOSPC`（idr已满）  

##### 示例：分配UID  
```c
int id, ret;
do {
    if (!idr_pre_get(&id_huh, GFP_KERNEL)) 
        return -ENOSPC;
    ret = idr_get_new(&id_huh, ptr, &id); 
} while (ret == -EAGAIN);
```

##### 分配指定最小UID  
```c
int idr_get_new_above(struct idr *idp, void *ptr, int starting_id, int *id);  
```  
- **功能**：分配不小于`starting_id`的UID，确保UID单调递增  
```c
static int next_id = 1;  // 全局计数器
if (!idr_get_new_above(&id_huh, ptr, next_id, &id))
    next_id = id + 1;    // 更新下一个起始ID  
```

##### XArray方式 (Linux 4.2以后)
```c
/* 原子分配 (无需预分配) */
int xa_alloc(struct xarray *xa, unsigned int *id, 
             void *entry, struct xa_limit limit, gfp_t gfp);
/* 分配递增ID示例 */
unsigned int next_id = 1;
xa_alloc(&xa_huh, &next_id, ptr, XA_LIMIT(next_id, UINT_MAX), GFP_KERNEL);
// 成功后 next_id 自动递增
```

---

#### 查找与删除  
##### 查找UID  
```c
void *idr_find(struct idr *idp, int id);  
```  
- **返回值**：成功返回关联指针，失败返回`NULL`  
- **注意**：在分配UID时，禁止将`NULL`作为有效idr值映射，否则无法区分查找失败与有效`NULL`  

##### 移除UID  
```c
void idr_remove(struct idr *idp, int id);  
```  
- **注意**：无错误返回值，需调用者确保UID存在  

##### XArray方式
```c
/* 查找 */
void *xa_load(struct xarray *xa, unsigned long index);
/* 删除并返回删除项 */
void *xa_erase(struct xarray *xa, unsigned long index);
```

---

#### 销毁  
```c
void idr_destroy(struct idr *idp);                  // 释放未使用内存  
void idr_remove_all(struct idr *idp);               // 强制移除所有UID  
```  
**典型流程**：  
```c
idr_remove_all(&id_huh);  // 先清空所有映射  
idr_destroy(&id_huh);     // 再释放内存，确保所有idr内存被释放  
kfree(user_data_ptr);  // 释放实际业务数据
```

##### XArray方式
```c
/* 销毁并释放所有资源 */
void xa_destroy(struct xarray *xa);

/* 安全销毁流程示例 */
void module_exit(void)
{
    unsigned long id;
    void *entry;
    // 遍历释放关联资源
    xa_for_each(&xa_huh, id, entry) {
        xa_erase(&xa_huh, id);
        kfree(entry);
    }
    xa_destroy(&xa_huh); // 释放XArray管理内存
}
```

---

#### 关键注意事项  
1. **并发控制**：  
   - `idr_pre_get`无需加锁  
   - `idr_get_new`等操作需自旋锁保护（参见第9/10章）  
