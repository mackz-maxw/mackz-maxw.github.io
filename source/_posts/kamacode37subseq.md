---
title: 代码随想录 | 刷题-动态规划10
categories: leetcode
lang: zh-CN
---

### 300.最长递增子序列
这题我一开始想使用回溯找最长子序列，但是我发现需要记录的状态太多，回溯没办法解决；同时对于序列中的相同数字，如何记住遍历前后的状态也是个问题。
题解中这题使用的是动态规划，对于每一个索引求它所在的最长递增子序列，再求所有子序列中最长的那个。

- 外层循环：遍历序列中每一个索引`i`
- 内层循环：求从开头到`i`之前的数字`j`中最长的子序列，如果`nums[i] > nums[j]`，则开头到`i`所在处最长子序列为`j`所在处+1

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(), 1);
        int mx = 1;
        for(int i = 1; i<nums.size(); i++){
            for(int j = 0; j<i; j++){
                if(nums[i] > nums[j])dp[i] = max(dp[i], dp[j] + 1);
                // cout<<dp[i]<<' ';
            }
            mx = max(mx, dp[i]);
            // cout<<endl;
        }
        return mx;
    }
};
```

### 674. 最长连续递增序列
这题不要想得太复杂了就行
```cpp
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        int lng = 1;
        int max_l = 1;
        for(int i = 1; i < nums.size(); i++){
            if(nums[i] > nums[i-1]){
                lng++;
            }else{
                lng = 1;
            }
            max_l = max(max_l, lng);
        }
        return max_l;
    }
};
```

### 718. 最长重复子数组 
这题我想可不可以分别求两个数组的最长相等前后缀表，再找重复子数组。看了动规解法我发现，一个两数组比较表能解决的事情，用前后缀表似乎有点浪费？

动规解法中，`dp[i][j]`表示两个数组下标i-1，j-1个数进行比较，这样0行和0列默认初始化为0即可。如果表示下标i，j个数，则需要再初始化一下0行和0列
```cpp
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        int len = 0;
        vector<vector<int>> dp(nums1.size()+1, vector<int>(nums2.size()+1, 0));
        for(int i = 1; i < (nums1.size()+1); i++){
            for(int j = 1; j < (nums2.size()+1); j++){
                if(nums1[i-1] == nums2[j-1]){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }
                if(dp[i][j] > len)len = dp[i][j];
            }
        }
        // for(auto line:dp){
        //     for(int l:line){
        //         cout<<l<<' ';
        //     }
        //     cout<<endl;
        // }
        return len;
    }
};
```