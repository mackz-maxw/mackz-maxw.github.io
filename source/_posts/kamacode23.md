---
title: 代码随想录 | 刷题-贪心算法1
categories: leetcode
lang: zh-CN
---

### 455.分发饼干
这题题解上两个思路：

* 思路1：优先考虑饼干，小饼干先喂饱小胃口
* 思路2：优先考虑胃口，先喂饱大胃口

我这里优先考虑饼干
```cpp
class Solution {
public:
    int findC(vector<int>& g, vector<int>& s, int gStart, int sStart){
        int count = 0;
        if(gStart == g.size() || sStart == s.size())return count;
        for(int i = sStart; i<s.size();i++){
            if(s[i] >= g[gStart]){
                count += 1;
                count += findC(g, s, gStart+1, i+1);
                break;
            }
        }
        return count;
    }
    int findContentChildren(vector<int>& g, vector<int>& s) {
        sort(g.begin(), g.end());
        sort(s.begin(), s.end());
        int cnt = findC(g, s, 0, 0);
        return cnt;
    }
};
```

### 376. 摆动序列
按题解思路，因为题目要求的是最长摆动子序列的长度，所以只需要统计数组的峰值数量就可以了（相当于是删除单一坡度上的节点，然后统计长度）
这就是贪心所贪的地方，让峰值尽可能的保持峰值，然后删除单一坡度上的节点
```cpp
class Solution {
public:
    int wiggleMaxLength(vector<int>& nums) {
        int waveCnt = 1;
        vector<bool> isPos;
        if(nums.size() == 1)return waveCnt;
        for(int i = 1; i<nums.size();i++){
            if(nums[i-1] == nums[i])continue; // 需要跳过相等的情况，不计入后面的判断条件中
            if(nums[i-1] < nums[i]){
                isPos.push_back(true);
            }else if(nums[i-1] > nums[i]){
                isPos.push_back(false);
            }
            if(i >= 1 && !isPos.empty() && isPos.size() == 1)waveCnt++;
            if(!isPos.empty() && isPos.size() >= 2){
                if(isPos[isPos.size()-1] != isPos[isPos.size()-2])waveCnt++;
            }
        }
        return waveCnt;
    }
};
```

### 53.最大子序和
由题解可以得知
如果 -2 1 在一起，计算起点的时候，一定是从 1 开始计算，因为负数只会拉低总和，这就是贪心贪的地方！

* 局部最优：当前“连续和”为负数的时候立刻放弃，从下一个元素重新计算“连续和”，因为负数加上下一个元素“连续和”只会越来越小。
* 全局最优：选取最大“连续和”

区间的终止位置，其实就是如果 count 取到最大值了，及时记录下来

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int maxCnt = nums[0];
        int cnt = 0;
        for(int i=0; i<nums.size();i++){
            cnt = cnt + nums[i];
            if(cnt > maxCnt){// 要先赋值，再判断是否小于零，不然全是负数的情况会不正确
                maxCnt = cnt;
            }
            if(cnt < 0){
                cnt = 0;
                continue;
            }
        }
        return maxCnt;
    }
};
```