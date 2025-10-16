---
title: 代码随想录 | 刷题-动态规划11
categories: leetcode
lang: zh-CN
---

### 1143.最长公共子序列 
按照动态规划，每次比较成功就加一，取所有值最大的思路是错的，这种会把乱序但是相同的字符也算进去。
于是我想了半天怎么先循环以i,j为右下角的正方形，本来想的是记忆化将i,j赋值为比较后相等的值，看了题解发现得在递推公式上作更改
因为字符中间可能会插入别的字符，所以ac,ace的比较结果和ac,aced的比较结果是一样的
```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        vector<vector<int>> dp(text1.size()+1, vector<int> (text2.size()+1, 0));
        for(int i = 1; i <= text1.size();i++){
            for(int j = 1; j <= text2.size(); j++){
                if(text1[i-1] == text2[j-1]){
                    dp[i][j] = dp[i-1][j-1]+1;
                }else{
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        return dp[text1.size()][text2.size()];
    }
};
```

### 1035.不相交的线
既然一个数只能连一根线，那么其实和上一题是一个意思了
```cpp
class Solution {
public:
    int maxUncrossedLines(vector<int>& nums1, vector<int>& nums2) {
        vector<vector<int>> dp(nums1.size()+1, vector<int> (nums2.size()+1, 0));
        for(int i = 1; i <= nums1.size(); i++){
            for(int j = 1; j <= nums2.size(); j++){
                if(nums1[i-1] == nums2[j-1]){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        return dp[nums1.size()][nums2.size()];
    }
};
```

### 53. 最大子序和
我想的是，`dp[i]`由`dp[i-1]`,`dp[i-1]+nums[i]`,`nums[i]`中的最大值决定，但是这样解会算出不连续的最大值

题解的推算法是这样的：
`dp[i]`只有两个方向可以推出来：
- `dp[i-1] + nums[i]`，即：`nums[i]`加入当前连续子序列和
- `nums[i]`，即：从头开始计算当前连续子序列和
再找每个的`dp[i]`最大值
```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        vector<int> dp(nums.size(), 0);
        dp[0] = nums[0];
        int res = nums[0];
        for(int i = 1; i < nums.size(); i++){
            dp[i] = max(dp[i-1]+nums[i], nums[i]);
            if(dp[i] > res)res = dp[i];
        }
        return res;
    }
};
```

### 392.判断子序列
常规做的话双指针法即可，按照动态规划来做是和第一题是差不多的思路
```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        vector<vector<int>> dp(s.size()+1, vector<int>(t.size()+1, 0));
        for(int i = 1; i <= s.size(); i++){
            for(int j = 1; j <= t.size(); j++){
                if(s[i-1] == t[j-1]){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = dp[i][j-1];
                }
            }
        }
        return dp[s.size()][t.size()] == s.size() ? true : false;
    }
};
```