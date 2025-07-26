---
title: 代码随想录 | 刷题-动态规划1
categories: leetcode
lang: zh-CN
---

###  509. 斐波那契数 
```cpp
class Solution {
public:
    int fib(int n) {
        if(n == 0)return 0;
        if(n == 1){
            return 1;
        }else if(n > 1 ){
            return fib(n-1)+fib(n-2);
        }
        return -1;
    }
};
```

### 70. 爬楼梯
递归超时了，得用数组算。如果用两个有效数字的数组重复计算还能再省空间
```cpp
class Solution {
public:
    int climbStairs(int n) {
        if(n <= 2){
            return n;
        }
        vector<int> dp(n+1, 1);
        dp[1] = 1;
        dp[2] = 2;
        for(int i = 3; i<=n;i++){
            dp[i] = dp[i-1] + dp[i-2];
        }
        return dp[n];
    }
};
```

###  746. 使用最小花费爬楼梯 
重点是跟着输出的dp数组看推导过程哪里有错
```cpp
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int stairs = cost.size();
        if(stairs == 1)return 0;
        if(stairs == 2)return min(cost[0], cost[1]);
        vector<int> dp(cost.size(), -1);
        dp[stairs-1] = cost[stairs-1];//back()
        dp[stairs-2] = cost[stairs-2];
        for(int i = (stairs-3); i>=0; i--){
            dp[i] = cost[i]+ min(dp[i+1], dp[i+2]);
        }
        // for(auto i: dp){
        //     cout<<i<<" ";
        // }
        return min(dp[0], dp[1]);
    }
};
```