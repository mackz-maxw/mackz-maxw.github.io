---
title: 代码随想录 | 刷题-贪心算法3
categories: leetcode
lang: zh-CN
---

### 134. 加油站

#### 初始思路：
1. **先找一段正收益连续区间（油量盈余最大的起点）**
    * 用 `curMax` 累加正收益段，记录最大盈余时的起点 `maxSt`。
    * 如果遇到油不够的站点，就重置 `curMax` 并将 `start` 改为下一个站点。
    * 如果到最后一站没找完，循环一圈回来继续查找。
        
2. **再从 `maxSt` 模拟绕一圈**
    * 从 `maxSt` 出发，逐站加减油量，确认是否能绕完整圈。
    * 只要中途油量不足就返回 `-1`。

#### ⚠️ 存在的问题：
其实这种逻辑完善完善也是可以做出来的，题解中有，就是比较繁琐

#### ✅ 更优的做法（贪心 + 一次遍历）：
只要 **总油量 ≥ 总花费**，一定有解。解的起点是：当前累加油量一旦为负，就从下一个站点重启。
```cpp
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int tank = 0;
        int startInd = 0;
        int total = 0;
        for(int i = 0; i < gas.size(); i++){
            int store = gas[i] - cost[i];
            total += store;
            tank += store;
            if(tank < 0){
                startInd = i + 1;
                tank = 0;
            }
        }
        return total < 0 ? -1 : startInd;
    }
};
```

### 135. 分发糖果
这在leetcode上是一道困难的题目，其难点就在于贪心的策略，如果在考虑局部的时候想两边兼顾，就会顾此失彼。
题解采用了两次贪心的策略：
* 一次是从左到右遍历，只比较右边孩子评分比左边大的情况。
* 一次是从右到左遍历，只比较左边孩子评分比右边大的情况。
```cpp
class Solution {
public:
    int candy(vector<int>& ratings) {
        vector<int> candy;
        candy.push_back(1);
        for(int i = 1;i<ratings.size();i++){
            if(ratings[i] > ratings[i-1]){
                candy.push_back(candy[i-1] + 1);
            }else{
                candy.push_back(1);
            }
        }
        int total = 0;
        for(int i = (ratings.size()-1); i>0;i--){
            if(ratings[i-1] > ratings[i] && candy[i-1] <= candy[i]){
                candy[i-1] = candy[i] + 1;
            }
            total += candy[i];
        }
        total += candy[0];
        return total;
    }
};
```

### 860.柠檬水找零 
这题不用map,用两个整数表示5和10的钞票数也是可以的
```cpp
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        map<int, int> changes;
        changes[5] = 0;
        changes[10] = 0;
        for(int i = 0; i<bills.size();i++){
            int exc = bills[i] - 5;
            if(exc == 0){
                changes[5]++;
            }else if(exc == 5){
                changes[5]--;
                changes[10]++;
            }else if(exc == 15){
                if(changes[10] >= 1){
                    changes[10]--;
                    changes[5]--;
                }else{
                    changes[5] -= 3;
                }
            }
            if(changes[5] < 0)return false;
        }
        return true;
    }
};
```

###  406.根据身高重建队列 
看题解可知，当身高是按照从高到矮，第二个数字从小到大排列的时候，按顺序插入即可满足条件
```cpp
class Solution {
public:
    static bool cmp(const vector<int> a, const vector<int> b){
        if(a[0] > b[0]){
            return true;
        }else if(a[0] == b[0] && a[1] < b[1]){
            return true;
        }
        return false;
    }
    vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
        sort(people.begin(), people.end(), cmp);
        list<vector<int>> que;
        for(int i = 0; i<people.size();i++){
            int insPos = people[i][1];
            std::list<vector<int>>::iterator p = que.begin();
            while(insPos > 0){
                p++;
                insPos--;
            }
            que.insert(p, people[i]);
        }
        return vector<vector<int>>(que.begin(), que.end());
    }
};
```