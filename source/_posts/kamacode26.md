---
title: 代码随想录 | 刷题-贪心算法4
categories: leetcode
lang: zh-CN
---

### 452. 用最少数量的箭引爆气球

#### 💡 题解逻辑如下：
* 如果当前气球的 **起点 > 上一个交集的最右端点**，说明它**不能和之前的箭共用**，需要再发一支箭。
    * 更新 `end` 为当前气球的 `end`，建立新的交集窗口。
        
* 否则，当前气球和前面的箭是**重叠的**，它们能一起被射中。
    * 更新 `end = min(end, points[i][1])` 是为了**缩小交集范围**，因为未来能共用这支箭的气球，必须跟所有已合并气球都有交集。
```cpp
class Solution {
public:
    static bool cmp(vector<int> a, vector<int> b){
        return a[0] < b[0];
        
    }
    int findMinArrowShots(vector<vector<int>>& points) {
        sort(points.begin(), points.end(), cmp);
        int arr = 1;
        if(points.size() == 0)return 0;
        int end = points[0][1];
        for(int i = 1; i<points.size();i++){
            if(end < points[i][0]){
                end = points[i][1];
                arr++;
            }else{
                end = min(end, points[i][1]);
            }
        }
        return arr;
    }
};
```

### 435. 无重叠区间 

#### 思路步骤：
1. **排序区间**（预处理）：
   - 使用自定义比较器 `cmp` 对所有区间进行排序。
   - 排序规则：**按起始点升序**（优先比较 `intervals[i][0]`）。若起始点相同，则**按结束点升序**（比较 `intervals[i][1]`）。
   - 按结束点排序效率可以更高

2. **初始化关键变量**：
   - 最大不重叠区间计数器 `intv = 1`（至少包含第一个区间）。
   - 当前覆盖的结束点 `end = intervals[0][1]`（以第一个区间的结束点作为初始基准）。

3. **遍历处理每个区间**（从第二个区间开始，即 `i = 1`）：
   - **检查重叠**：比较当前区间起始点 `intervals[i][0]` 与当前结束点 `end`。
     - **情况1：有重叠**（`intervals[i][0] < end`）：
       - 更新 `end = min(end, intervals[i][1])`（关键贪心选择：取最小结束点）。
       - **不增加计数器** `intv`（因为重叠区间不能同时保留，这里隐含选择了结束点更小的区间以正确计算无重叠区间个数）。
     - **情况2：无重叠**（`intervals[i][0] >= end`）：
       - 更新 `end = intervals[i][1]`（将结束点设为当前区间的结束点）。
       - 增加计数器 `intv++`（表示成功添加一个新的不重叠区间）。

4. **计算结果**：
   - 需要移除的最小区间数 = 总区间数 `intervals.size()` - 最大不重叠区间数 `intv`。

```cpp
class Solution {
public:
    static bool cmp(vector<int> a, vector<int>b){
        if(a[0] < b[0]){
            return true;
        }else if(a[0] == b[0] && a[1] < b[1]){
            return true;
        }
        return false;
    }
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), cmp);
        // for(auto i: intervals){
        //     cout<<i[0]<<" "<<i[1]<<", ";
        // }
        int intv = 1;
        int end = intervals[0][1];
        if(intervals.size() <= 1)return 0;
        for(int i = 1; i< intervals.size();i++){
            if(intervals[i][0] < end){
                end = min(end, intervals[i][1]);
            }else{
                end = intervals[i][1];
                intv++;
            }
        }
        return intervals.size() - intv;
    }
};
```

###  763.划分字母区间 
由题解：
在遍历的过程中相当于是要找每一个字母的边界，如果找到之前遍历过的所有字母的最远边界，说明这个边界就是分割点了。此时前面出现过所有字母，最远也就到这个边界了。
可以分为如下两步：

* 统计每一个字符最后出现的位置
* 从头遍历字符，并更新字符的最远出现下标，如果找到字符最远出现位置下标和当前下标相等了，则找到了分割点

```cpp
class Solution {
public:
    vector<int> partitionLabels(string s) {
        int hash[27] = {0};
        for(int i = 0;i<s.size();i++){
            hash[s[i] - 'a'] = i;
        }
        vector<int> result;
        int right = 0;
        int end = 0;
        for(int i = 0;i<s.size();i++){
            right = max(right, hash[s[i] - 'a']);
            if(right == i){
                result.push_back(right - end +1);
                right++;
                end = right;
            }
        }
        return result;
    }
};
```