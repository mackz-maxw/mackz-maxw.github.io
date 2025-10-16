---
title: 代码随想录 | 刷题-动态规划9
categories: leetcode
lang: zh-CN
---

### 188.买卖股票的最佳时机IV
买卖股票三的扩展版，从最多两次交易扩充到k次交易。注意买入和卖出某个数量的股票都要单列一列dp，并采用不同的状态转移方式。
```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {
        vector<vector<int>> dp(prices.size(), vector<int>(2*k+1, 0));
        for(int i = 1; i<=2*k; i++){
            if(i % 2 == 1)dp[0][i] -= prices[0];
        }
        if(prices.size() == 1)return 0;
        for(int i = 1; i < prices.size(); i++){
            for(int j = 1; j<=2*k; j+=2){
                dp[i][j] = max(dp[i-1][j-1] - prices[i], dp[i-1][j]);
            }
            for(int j = 2; j<=2*k; j+=2){
                // printf("index of prices: %d, sold out stock id: %d\n", i, j);
                dp[i][j] = max(dp[i-1][j-1] + prices[i], dp[i-1][j]);
            }
        }
        return dp[prices.size()-1][2*k];
    }
};
```

### 309.最佳买卖股票时机含冷冻期
我这道题还是写复杂了。其实不规定买入次数，就不需要每次买卖都开一个状态了，每天都计算持有/非持有状态即可，还可以省空间复杂度
我采用的思路延续了每次买入都有两个不同状态的方案，当计算卖出的时候往前比较两天，就是需要注意第二天持有股票状态的递推公式的特殊情况（选择第二天买入还是第一天买入）。其实采用仅四个状态也可以：
状态一：持有股票状态（今天买入股票，或者是之前就买入了股票然后没有操作，一直持有）
不持有股票状态，这里就有两种卖出股票状态
- 状态二：保持卖出股票的状态（两天前就卖出了股票，度过一天冷冻期。或者是前一天就是卖出股票状态，一直没操作）
- 状态三：今天卖出股票
状态四：今天为冷冻期状态，但冷冻期状态不可持续，只有一天！
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1)return 0;
        int status = prices.size()*2;
        vector<vector<int>> dp(prices.size(), vector<int>(status+1, 0));
        for(int i = 1; i <= status; i += 2){
            dp[0][i] -= prices[0];
        }
        for(int i = 1; i < prices.size(); i++){
            for(int j = 1; j <= status; j += 2){
                // printf("prices: %d, hold stock cnt: %d\n", i, j);
                if(i >= 3){
                    dp[i][j] = max(dp[i-2][j-1] - prices[i], dp[i-1][j]);
                }else{
                    dp[i][j] = max(-prices[i], dp[i-1][j]);
                }
            }
            for(int j = 2; j <= status; j += 2){
                dp[i][j] = max(dp[i-1][j-1] + prices[i], dp[i-1][j]);
            }
        }
        // for(auto l: dp){
        //     cout<<"prices line ";
        //     for(auto i: l){
        //         cout<<i<<' ';
        //     }
        //     cout<<endl;
        // }
        return dp[prices.size()-1][status];
    }
};
```

### 714.买卖股票的最佳时机含手续费
这题延续 *122.买卖股票的最佳时机II* 的思路即可
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        if(prices.size() == 1)return 0;
        int status = prices.size()*2;
        vector<vector<int>> dp(prices.size(), vector<int>(2, 0));
        dp[0][1] -= prices[0];
        for(int i = 1; i< prices.size(); i++){
            dp[i][0] = max(dp[i-1][1]+prices[i]-fee, dp[i-1][0]);
            dp[i][1] = max(dp[i-1][0]-prices[i], dp[i-1][1]);
        }
        // for(int i = 0; i<dp.size(); i++){
        //     for(int j = 0; j<dp[0].size(); j++){
        //         cout<<dp[i][j]<<' ';
        //     }
        //     cout<<endl;
        // }
        return dp[prices.size()-1][0];
    }
};
```