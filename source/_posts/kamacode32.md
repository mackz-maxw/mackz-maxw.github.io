---
title: 代码随想录 | 刷题-动态规划5
categories: leetcode
lang: zh-CN
---

###  518. 零钱兑换 II  
```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        vector<uint64_t> dp(amount + 1, 0); // 防止相加数据超int
        dp[0] = 1;
        // 第一轮循环应该将所有j = coins[i]置1
        // 想清楚这一点，就可以明确组合数的状态转移方程和排列数不一样
        for(int i = 0; i<coins.size(); i++){
            int coin = coins[i];
            for(int j = coin; j<(amount+1); j++){
                dp[j] += dp[j - coins[i]];
            }
            // cout<<"coins[i]="<<coins[i]<<endl;
            // for(int d:dp)cout<<d<<' ';
            // cout<<endl;
        }
        
        return dp[amount];
    }
};
```

###  377. 组合总和 Ⅳ  
如果求组合数就是外层for循环遍历物品，内层for遍历背包，因为物品处理顺序固定。
如果求排列数就是外层for遍历背包，内层for循环遍历物品，因为每个容量下都重新扫描所有物品。
```cpp
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        vector<int> dp(target + 1, 0);
        dp[0] = 1;
        for(int j = 0;j<=target;j++){
            for(int i = 0; i<nums.size();i++){
                if((j - nums[i])>=0 && dp[j] <= INT_MAX - dp[j - nums[i]]){
                    dp[j] += dp[j - nums[i]];
                }
            }
            // for(auto d:dp)cout<<d<<' ';
            // cout<<endl;
        }
        return dp[target];
    }
};
```

### 70. 爬楼梯 （进阶）
```cpp
class Solution {
public:
    int climbStairs(int n) {
        vector<int> dp(n+1, 0);
        dp[0] = 1;
        for(int i = 0; i<=n; i++){
            for(int j = 1; j<=2; j++){
                if((i-j) >= 0){
                    dp[i] += dp[i - j];
                }
            }
        }
        return dp[n];
    }
};
```