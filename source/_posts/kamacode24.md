---
title: 代码随想录 | 刷题-贪心算法2
categories: leetcode
lang: zh-CN
---

###  122.买卖股票的最佳时机II  
和之前的贪心算法题目一样，注意到局部最佳和整体的关系
```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int prof = 0;
        for(int i = 0; i < prices.size(); i++){
            if(i > 0){
                int left = prices[i] - prices[i-1];
                if(left > 0)prof += left;
            }
        }
        return prof;
    }
};
```

###  55. 跳跃游戏 
```cpp
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int maxi = 0;
        for(int i = 0;i<nums.size();i++){
            if(i < nums.size()-1 && nums[i] == 0 && maxi <= i)return false;
            int fari = i + nums[i];
            maxi = max(maxi, fari);
        }
        if(maxi >= nums.size() - 1){
            return true;
        }else{
            return false;
        }
    }
};
```

###  45.跳跃游戏II 

#### **最开始的思路：**
* 定义了一个 `minJumps` 数组，其中 `minJumps[i]` 表示到达位置 `i` 所需的最少跳跃次数。
* 初始用 `minJumps[i] = i`，默认每一步都跳一步。
* 然后你从前向后遍历每个位置 `i`，对于它可以跳到的最远位置 `reach = i + nums[i]`，尝试用 `minJumps[i] + 1` 来更新 `minJumps[reach]`。
* 最后返回的是 `minJumps[n - 1]`。
    
#### ❌ **问题和可改进点：**
1. **只更新了 `reach`，没更新从 `i+1` 到 `reach` 的所有中间位置**：
    * 你只更新了 `minJumps[reach]`，但跳到 `[i+1, reach]` 的每一个点其实都可能是通过 `i` 达到的，因此都可能能更新为 `minJumps[i] + 1`。
3. **时间复杂度仍是 `O(n^2)` 的最坏情况**：
    * 因为如果要更新 `[i+1, reach]` 中的所有点需要嵌套循环。

#### ✅ **题解思路遍历过程**：
每次跳跃的起点在哪里不重要，关键是这次最远跳跃是否可以次数内完成
* 对每个位置 `i`，更新 `far = max(far, i + nums[i])`，表示当前范围内下一跳能跳到的最远位置。
* 当遍历到边界 `i == beg` 时，说明当前跳跃结束，需要再跳一次，并更新新的边界 `beg = far`。
* 如果新的 `beg` 已经到达或超过数组末尾，则可以提前 `break`。

```cpp
class Solution {
public:
    int jump(vector<int>& nums) {
        int minJump = 0;
        int beg = 0;
        int far = 0;
        if(nums.size() == 1)return 0;
        for(int i = 0; i<nums.size();i++){
            far = max(far, i + nums[i]);
            if(i == beg){
                minJump++;
                beg = far;
                if(beg >= nums.size()-1)break;
            }
        }
        return minJump;
    }
};
```

###  1005.K次取反后最大化的数组和  

####  **最开始的思路**
用了一个递归函数 `lsum(nums, k)` 来处理问题，思路如下：
1. **遍历数组找出最小值 `minN` 及其下标 `minI`**；
2. **将 `nums[minI]` 取反**（也就是对当前最小值执行一次取反操作）；
3. **继续递归调用 `lsum(nums, k - 1)`**；
4. **递归终止条件**：`k == 0`，此时返回整个数组的和；
5. **优化点尝试**：当数组中最小值已经 ≥ 0，尝试用 `k % 2` 代替多余的取反操作（只取反一次或不取反）以减少递归次数。

#### ❌ **当前实现存在的问题：**
1. **每次递归都重新遍历整个数组，效率低**：
    * 时间复杂度接近 `O(k * n)`，最坏情况下可能是 `10^5 * 10^3`，会超时。
2. **递归开销大**：
    * 每次递归都复制 `nums` 数组（因为是传引用再递归），可能带来严重的性能瓶颈。
3. **贪心策略不够高效**：

#### ✅ **推荐的优化思路：**
采用**排序 + 贪心策略**来完成任务，流程如下：
1. **先将数组按绝对值从大到小排序**；
2. **从左到右遍历数组，把负数取反，直到 k 用完或者没有负数了**；
3. **如果还有剩余的 k 且是奇数，说明还有一次取反机会，把当前最小值（绝对值最小的数）再取反一次**；
4. 最后返回数组的总和。

```cpp
class Solution {
public:
    int largestSumAfterKNegations(vector<int>& nums, int k) {
        sort(nums.begin(), nums.end(), [](int a, int b){return abs(a) > abs(b);});
        for(int i = 0; i<nums.size(); i++){
            if(k == 0)break;
            if(nums[i] < 0){
                nums[i] = 0 - nums[i];
                k--;
            }
        }
        if(k % 2 == 1)nums[nums.size()-1] = 0 - nums[nums.size()-1];
        int sum = 0;
        for(int n: nums){
            sum += n;
        }
        return sum;
    }
};
```