---
title: 代码随想录 | 刷题-动态规划7
categories: leetcode
lang: zh-CN
---

### 198.打家劫舍 
```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        vector<int> dp(nums.size()+1, 0);
        dp[1] = nums[0];
        for(int i = 2; i <= nums.size(); i++){
            dp[i] = max(dp[i-2]+nums[i-1], dp[i-1]);
        }
        return dp[nums.size()];
    }
};
```

### 213.打家劫舍II 
三种情况的动态规划：
- 不考虑头尾元素
- 考虑头元素，不考虑尾元素
- 考虑尾元素，不考虑头元素

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if(nums.size() == 1)return nums[nums.size()-1];
        int res_front = rob_range(nums, 0, nums.size()-2);
        int res_back = rob_range(nums, 1, nums.size()-1);
        int res = max(res_front, res_back);
        if(nums.size() >= 3)res = max(res,rob_range(nums, 1, nums.size()-2));
        return res;
    }
    int rob_range(vector<int>& nums, int start, int end){
        vector<int> dp(nums.size()+1, 0);
        if(start == end)return nums[start];
        dp[start] = nums[start];
        dp[start+1] = max(nums[start], nums[start+1]);
        if((start+1) == end)return dp[start+1];
        for(int i = start+2; i<=end; i++){
            dp[i] = max(dp[i-2]+nums[i], dp[i-1]);
        }
        return dp[end];
    }
};
```

### 337.打家劫舍III 
直接使用递归，计算抢当前节点还是不抢当前节点时，对相同的子树进行了多次独立的递归调用。会超时，推荐记忆化：
利用递归，每层返回一个保存当前节点偷还是不偷的数组
```cpp
// struct TreeNode{
//     int val;
//     TreeNode* left;
//     TreeNode* right;
//     TreeNode(int v) : val(v), left(nullptr), right(nullptr){}
// };
class Solution {
public:
    vector<int> rob_tree(TreeNode* root){
        if(root == nullptr)return vector<int>{0, 0};
        vector<int> left = rob_tree(root->left);
        vector<int> right = rob_tree(root->right);

        // ↓不抢当前节点时，既可以选择抢子节点，也可选择不抢，可能会有跳过两个节点抢更优的情况
        int next_h = max(left[0], left[1])+max(right[0], right[1]);
        int this_h = root->val+left[0]+right[0];
        
        return {next_h, this_h};
    }
    int rob(TreeNode* root) {
        vector<int> res = rob_tree(root);
        return max(res[0], res[1]);
    }
};
```