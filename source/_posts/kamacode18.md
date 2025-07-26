---
title: 代码随想录 | 刷题-二叉树8
categories: leetcode
lang: zh-CN
---

### 669.修剪二叉搜索树
重点是理解当某个节点要删除时，返回的节点的逻辑
我的思路是觉得剪除根两边的孩子节点和只取一个子树中的部分节点应该分开讨论，看了题解发现部分代码是可以一起的
题解的思路也可以理解为先处理`root->val`自身，后处理子树

> 如果root（当前节点）的元素小于low的数值，那么应该递归右子树，并返回右子树符合条件的头结点。
> 如果root(当前节点)的元素大于high的，那么应该递归左子树，并返回左子树符合条件的头结点。
> 将下一层处理完左子树的结果赋给root->left，处理完右子树的结果赋给root->right。
> 最后返回root节点

```cpp
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(!root)return nullptr;
        if(root->val < low){
            TreeNode* right = trimBST(root->right, low, high);
            return right;
        }else if(root->val > high){
            TreeNode* left = trimBST(root->left, low, high);
            return left;
        }
        root->left = trimBST(root->left, low, high);
        root->right = trimBST(root->right, low, high);
        return root;
    }
};
```

### 108.将有序数组转换为二叉搜索树

#### 🔍 问题一：`cen` 取值错误
```cpp
int cen = (r - l) / 2;
```
这会导致**始终只取左半段的中点**，**没有加上 `l`**，导致选到的 `nums[cen]` 不是 `l` 到 `r` 区间的中点，而是从头算的。

🔧 应该写为：
```cpp
int cen = l + (r - l) / 2;
```

#### 🔍 问题二：递归边界判断不正确
```cpp
if((cen - l) > 1) ...
```
这样写会漏掉一层子节点，导致不完整构建树。例如当子区间只剩一个元素时（`cen - l == 1`），你就跳过了构建那一侧的节点。

🔧 正确的写法是直接递归调用，只要 `l <= r`：
```cpp
root->left = arrayToBST(nums, l, cen - 1);
root->right = arrayToBST(nums, cen + 1, r);
```
已经在函数开头判断了 `l > r` 就 `return nullptr;`，所以不需要重复判断。

最终解法：
```cpp
class Solution {
public:
    TreeNode* arrayToBST(vector<int>& nums, int l, int r){
        if(l > r)return nullptr;
        int cen = l + (r-l) / 2;
        TreeNode* root = new TreeNode(nums[cen]);
        root->left = arrayToBST(nums, l, cen-1);
        root->right = arrayToBST(nums, cen+1, r);
        return root;
    }
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        TreeNode* root = arrayToBST(nums, 0, nums.size()-1);
        return root;
    }
};
```

###  538.把二叉搜索树转换为累加树  
题解提示用双指针，给我吓一跳，用回溯其实是可以解决的
```cpp
class Solution {
public:
    int sumVal = 0;
    void cvt(TreeNode* root){
        if(!root)return;
        cvt(root->right);
        sumVal += root->val;
        root->val = sumVal;
        cvt(root->left);

    }
    TreeNode* convertBST(TreeNode* root) {
        cvt(root);
        return root;
    }
};
```