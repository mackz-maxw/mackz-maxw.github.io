---
title: 代码随想录 | 刷题-动态规划6
categories: leetcode
lang: zh-CN
---

### 322. 零钱兑换 
```cpp
#include <limits>
#include <algorithm> // 包含 std::min 和 std::max
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount+1, std::numeric_limits<int>::max());// 每个amount需要的最少coin数
        // 第一遍遍历需要把coins中有的coin数置1，所以dp[0] = 0方便+1
        dp[0] = 0;
        // 针对每个amount, 遍历减去coin看哪一种最少
        for(int i = 0; i<=amount; i++){
            for(int j = 0; j< coins.size();j++){
                if((i - coins[j]) >= 0 && dp[i - coins[j]] < std::numeric_limits<int>::max()){
                    dp[i] = min(dp[i], dp[i - coins[j]] + 1);
                }
            }
        }
        return dp[amount] == std::numeric_limits<int>::max() ? -1 : dp[amount];
    }
};
```

### 279.完全平方数 

### 139.单词拆分 
