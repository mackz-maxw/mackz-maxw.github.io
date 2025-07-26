---
title: 代码随想录 | 刷题-动态规划3
categories: leetcode
lang: zh-CN
---

###  416. 分割等和子集 

**动态规划的正确性**：
   - 每次迭代考虑一个新物品的所有可能组合
   - 在使用一维dp数组时，倒序遍历保证状态转移只依赖**上一轮**结果

原始思路主要问题：
```cpp
// 伪代码
int cnt = target;
for (遍历每个数字) {
    if (cnt >= 当前数字) cnt -= 当前数字; // 强制选择
    if (dp[cnt]) ... // 但dp从未更新！
}
```
**贪心选择错误**：强制按顺序选择数字，不能处理非连续选择
也可以使用布尔值判断：
```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int eq = 0;
        for(int n : nums){
            eq += n;
        }
        if(eq % 2 == 1){
            return false;
        }else{
            eq = eq / 2;
        }
        vector<bool> dp(eq+1, false);
        dp[0] = true;
        int cnt = eq;
        for(int n : nums){
            for(int e = eq; e>=n; e--){
                if(dp[e - n]){
                    dp[e] = true;
                }
                if(dp[eq])return true;
            }
        }
        return false;
    }
};
```