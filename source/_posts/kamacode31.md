---
title: 代码随想录 | 刷题-动态规划4
categories: leetcode
lang: zh-CN
---

### 01背包问题中一维和二维 dp 的遍历顺序

#### 一维 dp：**倒序遍历容量**

- **原因**：倒序遍历可以保证每个物品只被放入一次。
- **原理**：每次更新 `dp[j]` 时，使用的是上一轮（未加入当前物品）的状态，避免重复加入同一个物品。
- **举例**：  
  - 物品0，重量1，价值15  
  - 正序遍历：`dp[2] = dp[1] + 15 = 30`（物品0被加了两次，错误！）
  - 倒序遍历：`dp[2] = dp[1] + 15 = 15`，`dp[1] = dp[0] + 15 = 15`（每个容量只加一次物品0）

#### 二维 dp：**正序遍历容量**

- **原因**：二维 dp 的 `dp[i][j]` 都是通过上一层 `dp[i-1][j]` 计算的，本层不会覆盖上一层，所以不会重复加入物品。
- **原理**：每一层代表一个物品，状态转移只依赖于上一层，天然保证每个物品只选一次。


### 1049. 最后一块石头的重量 II 

- 目标：将所有石头分成两组，使两组总重量尽量接近，最后剩下的石头重量就是两组重量差的绝对值。
- 转化为**01背包问题**：每个石头只能选一次，背包容量为所有石头重量和的一半，尽量让一组的重量最大且不超过容量。

```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum = 0;
        int hlf_sum = 0;
        for(int s: stones){
            sum += s;
        }
        hlf_sum = sum / 2; // 注意hlf_sum / 2是向下取整的

        vector<int> dp(1501, 0); // 一定要保证dp[j]是可以比较的，所以我们选择重量为j
        for(int i = 0; i<stones.size();i++){
            for(int j = hlf_sum; j>=stones[i]; j--){
                dp[j] = max(dp[j], dp[j - stones[i]] + stones[i]);
            }
        }
        
        return sum - dp[hlf_sum] * 2;
    }
};
```

### 494. 目标和 

该问题可以转化为一个子集和问题：在数组中找到一些数字，使它们的和等于 `(target + sum(nums)) / 2`（记为 `bagSize`），每个数字只能使用一次。以下是详细步骤：

1. **问题转化**：
   - 设加法总和为 `x`，则减法总和为 `sum - x`。
   - 根据条件 `x - (sum - x) = target`，解得 `x = (target + sum) / 2`。
   - 问题转化为：在 `nums` 中选取若干数字，使它们的和等于 `bagSize`，求选取方案数。

2. **边界条件**：
   - 若 `abs(target) > sum`，无解。
   - 若 `(target + sum)` 为奇数，无解（`bagSize` 必须是整数）。

3. **动态规划**：
   - 使用一维数组 `dp[j]` 表示装满容量 `j` 的背包的方案数。
   - 初始化 `dp[0] = 1`（空集方案）。
   - 遍历每个数字，逆序更新 `dp` 数组（避免重复使用数字）。

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        // 回溯法容易超时
        int sum = 0;
        for(int n:nums){
            sum += n;
        }
        int sz = (target + sum) / 2;
        if((target + sum) % 2 == 1)return 0;
        if(abs(target) > sum)return 0;
        vector<int> dp(sz+1, 0);
        dp[0] = 1;
        for(int i = 0; i<nums.size();i++){
            for(int j = sz; j >= nums[i]; j--){
                dp[j] += dp[j-nums[i]];
            }
        }
        return dp[sz];
    }
};
```

#### 代码解释
1. **初始化**：
   - `dp[0] = 1`：表示容量为0的背包有1种方案（不选任何数字）。
   - 其他位置初始化为0。

2. **遍历更新**：
   - 对每个数字 `num`，从 `bagSize` 到 `num` 逆序遍历。
   - 更新公式：`dp[j] += dp[j - num]`，表示：
     - **不选 `num`达成j**：方案数保持 `dp[j]`。
     - **选 `num`达成j**：方案数加上 `dp[j - num]`（剩余容量 `j - num` 的方案数）。

3. **示例**：
   - `nums = [1, 1, 1, 1, 1]`, `target = 3`：
     - `total = 5`, `bagSize = (3 + 5) / 2 = 4`。
     - `dp` 数组初始为 `[1, 0, 0, 0, 0]`。
     - 遍历每个 `1` 后，`dp[4] = 5`（5种组合方式）。

#### 复杂度
- **时间复杂度**：O(n × bagSize)，其中 `n` 为数组长度。
- **空间复杂度**：O(bagSize)。

### 474.一和零  
本题容易被看成多重背包问题，识别清楚问题本质就容易做了
```cpp
class Solution {
public:
    vector<int> count(string s){
        vector<int> cnt;
        int zeroes = 0, ones = 0;
        for(char c:s){
            if(c == '0')zeroes++;
            if(c == '1')ones++;
        }
        cnt.push_back(zeroes);
        cnt.push_back(ones);
        return cnt;
    }
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
        for(int i = 0; i<strs.size();i++){
            vector<int> cnt = count(strs[i]);
            if(cnt[0] > m || cnt[1] > n){
                continue;
            }
            for(int a = m; a >= cnt[0]; a--){
                for(int b = n; b >= cnt[1]; b--){
                    dp[a][b] = max(dp[a][b], dp[a-cnt[0]][b-cnt[1]] + 1);
                }
            }
        }
        return dp[m][n];
    }
};
```