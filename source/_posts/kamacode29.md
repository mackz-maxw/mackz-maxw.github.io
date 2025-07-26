---
title: 代码随想录 | 刷题-动态规划2
categories: leetcode
lang: zh-CN
---

### 62.不同路径 
想清楚要怎么推导每个格子，可以怎么推导
```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        if(m == 1 || n == 1)return 1;
        vector<vector<int>> dp(m, vector<int>(n, 1));
        for(int i = 1;i<m;i++){
            for(int j = 1;j<n;j++){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
};
```

### 63. 不同路径 II 

#### 问题分析
1. **起点`(0,0)`未特殊处理**：
   - 当`i=0, j=0`时（起点），您的代码会进入`else`分支（因为不满足前三个条件）。
   - 在`else`分支中，它计算`dp[i][j] = dp[i-1][j] + dp[i][j-1]`，但`i-1 = -1`、`j-1 = -1`（**非法索引**），导致未定义行为（崩溃或错误结果）。

2. **边界条件不完整**：
   - 当起点有障碍物时，虽能设为0，但后续计算仍可能依赖无效索引。
   - 初始化`dp`为全1是冗余的，且可能掩盖问题（实际值会被覆盖）。

#### 修复后的代码
```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        vector<vector<int>> dp(m, vector<int>(n, 0));
        for(int i = 0; i<m;i++){
            for(int j = 0; j<n;j++){
                if(obstacleGrid[i][j] == 1){
                    dp[i][j] = 0;
                }else if(i == 0 && j == 0){
                    dp[i][j] = 1;
                }else if(i == 0){
                    dp[i][j] = dp[i][j-1];
                }else if(j == 0){
                    dp[i][j] = dp[i-1][j];
                }else{
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }
            }
        }
        return dp[m-1][n-1];
    }
};
```

### 343. 整数拆分 （可跳过）
我直接用贪心法，辗转减去3来写了
动态规划思路如下：

- 遍历 $j$（$1 \leq j < i$），比较 $(i - j) \times j$ 和 $dp[i - j] \times j$，取最大值。
- `j * (i - j)` 是把整数拆分为两个数相乘。
- `j * dp[i - j]` 是把整数拆分为两个及以上的数相乘。
- 如果用 `dp[i - j] * dp[j]`，则相当于强制把一个数拆成四份及以上。

递推公式为：
`dp[i] = max({dp[i], (i - j) * j, dp[i - j] * j});`

为什么还要比较 `dp[i]` 呢？  
因为每次计算 `dp[i]` 时，都要保证它是当前的最大值。
```cpp
class Solution {
public:
    int integerBreak(int n) {
        if(n <= 3)return n-1;
        int pd = 1;
        while(n > 3){
            if(n == 4){
                pd = pd * 4;
                n = 0;
            }else{
                n = n - 3;
                pd = pd * 3;
            }
        }
        if(n > 0){
            pd = pd * n;
        }
        return pd;
    }
};
```

### 96.不同的二叉搜索树 （可跳过）
```cpp
class Solution {
public:
    int numTrees(int n) {
        if(n <= 2)return n;
        vector<int> dp(n+1, 0);
        dp[0] = 1;// 方便计算
        dp[1] = 1;
        dp[2] = 2;
        for(int i = 3; i<=n ;i++){
            int cnt = 0;
            for(int j = 1; j<=i; j++){
                cnt += dp[j-1] * dp[i-j];
            }
            dp[i] = cnt;
        }
        return dp[n];
    }
};
```