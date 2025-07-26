---
title: 代码随想录 | 刷题-二叉树7
categories: leetcode
lang: zh-CN
---

### 235. 二叉搜索树的最近公共祖先 
相对二叉树的最近公共祖先简单多了
```cpp
#include <algorithm> // min 和 max
using namespace std;
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(!root)return nullptr;
        int rt = root->val;
        int minV = min(p->val, q->val);
        int maxV = max(p->val, q->val);
        if(rt>=minV && rt<=maxV){
            return root;
        }else if(rt < minV){
            return lowestCommonAncestor(root->right, p, q);
        }else{
            return lowestCommonAncestor(root->left, p,q);
        }
    }
};
```

### 701.二叉搜索树中的插入操作
```cpp
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        TreeNode* insNode = new TreeNode(val);
        if(!root)return insNode;
        if(root->val > val){
            root->left = insertIntoBST(root->left, val);
        }else{
            root->right = insertIntoBST(root->right, val);
        }
        return root;
    }
};
```

### 450.删除二叉搜索树中的节点
我的想法是：先找到应该删除的节点子树-找到它的左子树最右节点-找到最右节点的父节点-将最右节点的值替换给应该删除的节点并删除最右节点
题解虽然也类似地需要分情况讨论，但是简洁地多：

* 第一种情况：没找到删除的节点，遍历到空节点直接返回了
* 找到删除的节点
    * 第二种情况：左右孩子都为空（叶子节点），直接删除节点， 返回NULL为根节点
    * 第三种情况：删除节点的左孩子为空，右孩子不为空，删除节点，右孩子补位，返回右孩子为根节点
    * 第四种情况：删除节点的右孩子为空，左孩子不为空，删除节点，左孩子补位，返回左孩子为根节点
    * 第五种情况：左右孩子节点都不为空，则将删除节点的左子树头结点（左孩子）放到删除节点的右子树的最左面节点的左孩子上，返回删除节点右孩子为新的根节点。

```cpp
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        if(!root)return nullptr;
        if(root->val == key){
            if(!root->left)return root->right;
            if(!root->right)return root->left;
            TreeNode* minR = root->right;
            while(minR->left)minR = minR->left;
            minR->left = root->left;
            return root->right;
        }
        if(root->val > key){
            root->left = deleteNode(root->left, key);
        }else{
            root->right = deleteNode(root->right, key);
        }
        return root;
    }
};
```