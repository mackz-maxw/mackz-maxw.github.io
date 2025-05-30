---
title: 代码随想录 | 刷题-二叉树5
categories: leetcode
lang: zh-CN
---

### 654.最大二叉树
```cpp
class Solution {
public:
    TreeNode* maxTree(vector<int>& nums, int l, int r){
        int maxNum = 0;
        int maxI = l;
        for(int i = l;i <= r;i++){
            if(nums[i] > maxNum){
                maxNum = nums[i];
                maxI = i;
            }
        }
        TreeNode* root = new TreeNode(maxNum);
        if(maxI > l)root->left = maxTree(nums, l, maxI-1);
        if(maxI < r)root->right = maxTree(nums, maxI +1, r);
        return root;
    }
    TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        TreeNode* res = maxTree(nums, 0, nums.size()-1);
        return res;
    }
};
```

### 617.合并二叉树 
```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if(!root1 && !root2)return nullptr;
        if(!root1)return root2;
        if(!root2)return root1;
        TreeNode* node = new TreeNode(root1->val + root2->val);
        node->left = mergeTrees(root1->left, root2->left);
        node->right = mergeTrees(root1->right, root2->right);
        return node;
    }
};
```

### 700.二叉搜索树中的搜索
不可以在`root->val == val`之前判定左右子树是否存在，会在搜索到叶子节点时提前结束
```cpp
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if(!root)return NULL;
        // cout<<"cur val"<<root->val<<endl;
        // if(!root->left && !root->right)return NULL; // --- 不可以在这里判定
        if(root->val == val){
            return root;
        }else if(root->val > val){
            return searchBST(root->left, val);
        }else{
            return searchBST(root->right, val);
        }
    }
};
```

### 98.验证二叉搜索树 
还是掉坑里了，其实验证二叉树中序遍历是否递增就好。当然也要注意最小值如何取
```cpp
class Solution {
public:
    TreeNode* pre = NULL;
    bool isValidBST(TreeNode* root) {
        if(!root)return true;
        bool left = isValidBST(root->left);
        if(pre != NULL && pre->val >= root->val)return false;
        pre = root;
        bool right = isValidBST(root->right);
        return left && right;
    }
};
```