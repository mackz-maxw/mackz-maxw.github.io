---
title: 代码随想录 | 刷题-动态规划8
categories: leetcode
lang: zh-CN
---

### 121. 买卖股票的最佳时机
贪心方法：取最左最小值，取最右最大值
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1)return 0;
        int profit = 0;
        int i = 0;
        for(int j = 1; j<prices.size(); j++){
            if((prices[j] - prices[i])>profit)profit = prices[j] - prices[i];
            if(prices[j] < prices[i])i = j;
        }
        return profit;
    }
};
```
动态规划方法：每天保存两个数值
- 当天持有股票的最大值
  - 第i-1天就持有股票，那么就保持现状，所得现金就是昨天持有股票的所得现金 即：dp[i - 1][0]
  - 第i天买入股票，所得现金就是买入今天的股票后所得现金即：-prices[i]
- 当天不持有股票的最大值
  - 第i-1天就不持有股票，那么就保持现状，所得现金就是昨天不持有股票的所得现金 即：dp[i - 1][1]
  - 第i天卖出股票，所得现金就是按照今天股票价格卖出后所得现金即：prices[i] + dp[i - 1][0]

### 122.买卖股票的最佳时机II 
尝试一下动态规划方法
```cpp
class Solution {
public:
    void printdp(vector<vector<int>> dp){
        for(auto i: dp){
            for(auto j: i){
                cout<<j<<' ';
            }
            cout<<endl;
        }
    }
    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1)return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(2, 0));
        // printdp(dp);
        dp[0][1] -= prices[0];
        for(int i = 1; i < prices.size(); i++){
            dp[i][0] = max(dp[i-1][1]+prices[i], dp[i-1][0]);
            dp[i][1] = max(dp[i-1][1], dp[i-1][0]-prices[i]);
        }
        // printdp(dp);
        return dp[prices.size()-1][0];
    }
};
```

### 123.买卖股票的最佳时机III 
为什么“选择两个最大的上升区间”这种解决方式不正确：
由于交易次数限制，并不是所有上升区间都需要被单独考虑。有时一笔交易可能覆盖多个上升区间

本题建议使用动态规划，推导四个状态：
- 第一次持有股票
- 第一次不持有股票
- 第二次持有股票
- 第二次不持有股票
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.size() == 1)return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(5, 0));
        dp[0][1] -= prices[0];
        dp[0][3] -= prices[0];
        for(int i = 1; i<prices.size(); i++){
            dp[i][1] = max(dp[i-1][0]-prices[i], dp[i-1][1]);
            dp[i][2] = max(dp[i-1][1]+prices[i], dp[i-1][2]);
            dp[i][3] = max(dp[i-1][2]-prices[i], dp[i-1][3]);
            dp[i][4] = max(dp[i-1][3]+prices[i], dp[i-1][4]);
        }
        return dp[prices.size()-1][4];
    }
};
```