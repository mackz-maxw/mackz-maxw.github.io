---
title: 代码随想录 | 刷题-二叉树
categories: leetcode
lang: zh-CN
---

### 144.二叉树的前序遍历

#### 递归法
注意边界条件，root为空指针时不再向下迭代叶子节点
```cpp
class Solution {
public:
    void preTree(TreeNode* root, vector<int>& vec){
        if(root){
            vec.push_back(root->val);
            preTree(root->left, vec);
            preTree(root->right, vec);
        }
    }
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> result;
        preTree(root, result);
        return result;
    }
};
```

#### 迭代法
注意栈先入先出的特性对入栈顺序的影响
```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        stack<TreeNode*> st;
        vector<int> result;
        if(!root)return result;
        st.push(root);
        while(!st.empty()){
            TreeNode* rt = st.top();
            st.pop();
            result.push_back(rt->val);
            if(rt->right)st.push(rt->right);
            if(rt->left)st.push(rt->left);
        }
        return result;
    }
};
```

### 145.二叉树的后序遍历

#### 递归法
```cpp
class Solution {
public:
    void postTree(TreeNode* root, vector<int>& vec){
        if(!root)return;
        postTree(root->left, vec);
        postTree(root->right, vec);
        vec.push_back(root->val);
    }
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        postTree(root, result);
        return result;
    }
};
```

#### 迭代法
注意栈先入先出的特性对入栈顺序的影响
```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> result;
        if(!root)return result;
        stack<TreeNode*> st;
        st.push(root);
        while(!st.empty()){
            TreeNode* cur = st.top();
            st.pop();
            result.push_back(cur->val);
            if(cur->left)st.push(cur->left);
            if(cur->right)st.push(cur->right);
        }
        reverse(result.begin(),result.end());
        return result;
    }
};
```

### 94.二叉树的中序遍历

#### 递归法
```cpp
class Solution {
public:
    void inTree(TreeNode* root, vector<int>& vec){
            if(!root)return;
            inTree(root->left, vec);
            vec.push_back(root->val);
            inTree(root->right, vec);
            
        }
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        inTree(root, result);
        return result;
    }
};
```

#### 迭代法
写答案的时候一直在想，因为想处理左子树方便，我想stack里面初始没有node，这样该如何定义while的初始条件？看了题解发现可以`while (cur != nullptr || !NodeStack.empty())`（但是还是写了麻烦的写法）  
一直没想明白递归到弹出第一个左叶子之后如何避免在弹出中间节点时再把左叶子压栈，看了题解发现`cur = cur->right;`此时cur为空指针，直接跳过下一个while即可
```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        stack<TreeNode*> NodeStack;
        vector<int> result;
        if(!root)return result;
        NodeStack.push(root);
        TreeNode* cur = root;
        while(!NodeStack.empty()){
            while(cur && cur->left != nullptr){
                cur = cur->left;
                NodeStack.push(cur);
            }
            cur = NodeStack.top();
            result.push_back(cur->val);
            NodeStack.pop();
            if(cur->right){
                NodeStack.push(cur->right);
            }
            cur = cur->right;
        }
        return result;
    }
};
```