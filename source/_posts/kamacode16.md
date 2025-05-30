---
title: 代码随想录 | 刷题-二叉树6
categories: leetcode
lang: zh-CN
---

### 530.搜索树的最小绝对差
遇到在二叉搜索树上求什么最值啊，差值之类的，就把它想成在一个有序数组上求最值，求差值，这样就简单多了
尝试自己写遍历，虽然有很多莫名其妙的值，但是居然写对了
```cpp
class Solution {
public:
    TreeNode* maxNode = nullptr;
    int absd  = -1;
    int getMinimumDifference(TreeNode* root) {
        if(!root)return -1;
        int left = -1;
        int right = -1;
        if(root->left)left = getMinimumDifference(root->left);
        if(maxNode != nullptr){
            int maxd = root->val - maxNode->val;
            if(absd == -1){
                absd = maxd;
            }else if(maxd < absd){
                absd = maxd;
            }
        }
        maxNode = root;
        if(root->right)right = getMinimumDifference(root->right);
        return absd;
    }
};
```

### 501.二叉搜索树中的众数
如果我自己写，就遍历数组取最大值好了。看题解发现可以中序遍历，按这种方法找：
弄一个指针指向前一个节点，这样每次cur（当前节点）才能和pre（前一个节点）作比较。
而且初始化的时候pre = NULL，这样当pre为NULL时候，我们就知道这是比较的第一个元素。
如果 频率count 等于 maxCount（最大频率），当然要把这个元素加入到结果集中（以下代码为result数组），  
频率count 大于 maxCount的时候，不仅要更新maxCount，而且要清空结果集（以下代码为result数组），因为结果集之前的元素都失效了。

注意计数和判定最大值要分开，并且确定好是判定当前节点
```cpp
class Solution {
public:
    TreeNode* pre = nullptr;
    int cnt = 0;
    int cntMax = 0;
    void traverse(vector<int> &res, TreeNode* root){
        if(!root)return;
        traverse(res, root->left);
        if(!pre){
            cnt++;
        }else if(pre->val == root->val){
            cnt++;
        }else{ //  if(pre->val != root->val)
            cnt = 1;
        }
        if(cnt == cntMax){
            res.push_back(root->val);
        }else if(cnt > cntMax){
            res.clear();
            cntMax = cnt;
            res.push_back(root->val);
        }
        pre = root;
        traverse(res, root->right);
        return;
    }
    vector<int> findMode(TreeNode* root) {
        vector<int> res;
        traverse(res, root);
        return res;
    }
};
```

### 236. 二叉树的最近公共祖先
我的解法就是暴力解，找到p和q的路径并比较路径中不同的节点，前一个就是公共最小祖先了
看了题解发现还可以回溯：

> 遇到这个题目首先想的是要是能自底向上查找就好了，这样就可以找到公共祖先了。
> 那么二叉树如何可以自底向上查找呢？
> 回溯啊，二叉树回溯的过程就是从底到上。
> 后序遍历（左右中）就是天然的回溯过程，可以根据左右子树的返回值，来处理中节点的逻辑。

搞清楚什么时候返回root也很重要
```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(!root || root == p || root == q)return root;
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);
        if(left != nullptr && right != nullptr)return root;
        if(left == nullptr)return right;
        return left;
    }
};
```