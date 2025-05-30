---
title: 代码随想录 | 刷题-二叉树2
categories: leetcode
lang: zh-CN
---

今天找到了一个将chatGPT内容markdown化的插件（chrome插件商店搜索chatGPT to Markdown），这样整理每天的错题就方便直观多了

### 堆对象 vs 栈对象 初始化
在 **C++ 中，`struct` 和 `class` 在语法上几乎没有区别**，主要区别只有两点：

1. 默认访问权限：`struct` 默认是 `public`，`class` 默认是 `private`
    
2. 默认继承权限：`struct` 默认是 `public` 继承，`class` 默认是 `private` 继承
    

👉 所以：**`struct` 并不“必须”用 `new` 来创建对象！**

| 写法 | 内存位置 | 生命周期 | 示例 |
| --- | --- | --- | --- |
| `Class c;` | 栈 (stack) | 自动管理 | `class` 或 `struct` 都可以 |
| `Class* c = new Class();` | 堆 (heap) | 需要手动 `delete` | `class` 或 `struct` 都可以 |

**在需要动态分配的场景使用new（如链表节点）**
    
```cpp
struct Node {
    int val;
    Node* next;
};

Node* head = new Node(); // ✅ 堆上分配，适合动态数据结构
```

### 102. 二叉树的层序遍历
昨天的层序遍历练一练
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode*> lev;
        vector<vector<int>> res;
        if(!root)return res;
        lev.push(root);
        while(!lev.empty()){
            int len = lev.size();
            vector<int> levVar;
            while(len > 0){
                len--;
                TreeNode* cur = lev.front();
                lev.pop();
                levVar.push_back(cur->val);
                if(cur->left)lev.push(cur->left);
                if(cur->right)lev.push(cur->right);
            }
            res.push_back(levVar);
        }
        return res;
    }
};
```

### 226.翻转二叉树

#### ❌ 问题一：错误创建了无用的 `TreeNode`

```cpp
TreeNode* rightT = new TreeNode();
if(root->right) rightT = root->right;
```

在这里无论 `root->right` 是否为空，都会创建一个新的 `TreeNode` 实例。但实际上你只需要记录 `root->right` 的原始指针。**没有 delete 掉那个多余的 new TreeNode()**，会造成内存泄漏。在递归中创建新的 `TreeNode`，可能无限递归，造成 **stack overflow**。

要删除通过 `new TreeNode()` 动态创建的对象，只需要调用 `delete` 并传入该指针即可。例如：

```cpp
TreeNode* node = new TreeNode();
// 使用 node 做一些操作
delete node; // 释放内存
node = nullptr; // 避免悬空指针（可选但推荐）
```

#### ✅ 注意事项：

1. **每一个 `new` 都要对应一个 `delete`**，否则会导致内存泄漏。
    
2. 删除后最好将指针设为 `nullptr`，防止以后误访问已释放的内存。
    
3. 如果你创建的是数组，用 `new[]`，则要用 `delete[]`。
    

例如：

```cpp
int* arr = new int[10];
delete[] arr;
```

#### ❌ 问题二：判断逻辑混乱，递归不完整

```cpp
if(root->left){
    root->right = invertTree(root->left);
}
if(rightT != nullptr){
    root->left = invertTree(rightT);
}
```

如果 `root->left` 是 `nullptr`，就跳过了对 `root->right` 的赋值

正确的做法是把翻转和赋值分隔开来：
```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if(!root)return root;
        TreeNode* rightT = invertTree(root->right);
        TreeNode* leftT = invertTree(root->left);
        root->left = rightT;
        root->right = leftT;
        return root;
    }
};
```

### 101. 对称二叉树
别忘了判定值是否一致
```cpp
class Solution {
public:
    bool isSym(TreeNode* left, TreeNode* right){
        if(!left && !right)return true;
        if((!left && right) || (!right && left))return false;
        if(left->val != right->val)return false;
        bool isIn = isSym(left->right, right->left);
        bool isOut = isSym(left->left, right->right);
        return isIn && isOut;
    }
    bool isSymmetric(TreeNode* root) {
        if(!root)return true;
        return isSym(root->left, root->right);
    }
};
```

### 104.二叉树的最大深度

* 二叉树节点的深度：指从根节点到该节点的最长简单路径边的条数或者节点数（取决于深度从0开始还是从1开始）
* 二叉树节点的高度：指从该节点到叶子节点的最长简单路径边的条数或者节点数（取决于高度从0开始还是从1开始）

注意边界条件对代码的影响

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        int maxD = 0;
        if(root){
            maxD++;
        }else{
            return maxD;
        }
        int maxL = root->left ? maxDepth(root->left) : 0;
        int maxR = root->right ? maxDepth(root->right) : 0;
        maxD += maxL > maxR ? maxL : maxR;
        return maxD;
    }
};
```

### 111.二叉树的最小深度
迭代法：
```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        int minD = 0;
        if(!root)return minD;
        minD++;
        if(!root->left && !root->right)return minD;
        if(!root->left && root->right){
            return minD + minDepth(root->right);
        }else if(root->left && !root->right){
            return minD + minDepth(root->left);
        }else{
            int minL = minDepth(root->left);
            int minR = minDepth(root->right);
            minD += minL > minR ? minR : minL;
            return minD;
        }
    }
};
```