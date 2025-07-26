---
title: 代码随想录 | 刷题-贪心算法5
categories: leetcode
lang: zh-CN
---

### 56. 合并区间  

#### 错误分析
1. **合并逻辑错误**（关键问题）：
   ```cpp
   intervals[i] = (intervals[i-1][0], intervals[i][1]); // 错误写法
   ```
   - **语法错误**：使用逗号运算符 `(a, b)` 实际返回 `b`，导致区间被错误赋值为 `[0, intervals[i][1]]`（起始点丢失）。
   - **逻辑错误**：合并时结束点未取两者最大值。正确做法：结束点应为 `max(intervals[i-1][1], intervals[i][1])`，否则可能丢失覆盖范围（如合并 `[1,5]` 和 `[2,3]` 会得到 `[1,3]` 而非 `[1,5]`）。

2. **重叠条件不精确**：
   ```cpp
   if(intervals[i-1][1] >= intervals[i][0]) // 边界情况处理不当
   ```
   - 题目要求：若前区间结束点 **等于** 后区间起始点（如 `[1,2]` 和 `[2,3]`），应合并为 `[1,3]`。当前条件 `>=` 正确，但合并操作错误导致结果异常。

3. **性能问题**：
   - **拷贝开销**：`cmp` 函数参数为传值（`vector<int> a, b`），应改为 **传引用**（`const vector<int>& a, b`）避免不必要的拷贝。
   - **额外空间**：使用 `incl` 标记数组非必需，经典解法可优化空间复杂度。

4. **边界处理缺失**：
   - 当输入为空时，应直接返回空数组，但代码未显式处理 `intervals.size() == 0` 的情况（依赖 `size <= 1` 返回，逻辑正确但不够清晰）。

#### 比较合并贪心解法（可以ac）
```cpp
class Solution {
public:
    static bool cmp(const vector<int>& a, const vector<int>& b) { // 传引用
        return a[0] < b[0] || (a[0] == b[0] && a[1] < b[1]);
    }
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), cmp);
        if (intervals.size() <= 1) return intervals;
        vector<bool> incl(intervals.size(), true);
        for (int i = 1; i < intervals.size(); i++) {
            if (intervals[i-1][1] >= intervals[i][0]) {
                // 正确合并：起始点取前一个，结束点取两者最大值
                intervals[i][0] = intervals[i-1][0];
                intervals[i][1] = max(intervals[i-1][1], intervals[i][1]);
                incl[i-1] = false;
            }
        }
        vector<vector<int>> result;
        for (int j = 0; j < intervals.size(); j++) {
            if (incl[j]) result.push_back(intervals[j]);
        }
        return result;
    }
};
```

#### 经典贪心解法思路
1. **按起始点排序区间**。
2. **动态合并**：
   - 初始化结果数组 `merged`。
   - 遍历每个区间：
     - 若 `merged` 为空 **或** 当前区间与 `merged` 最后一个区间不重叠（`current[0] > merged.back()[1]`），直接加入。
     - 否则，**更新 `merged` 最后一个区间的结束点**：`max(merged.back()[1], current[1])`。

#### 经典贪心解法
```cpp
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.empty()) return {};
        sort(intervals.begin(), intervals.end());
        vector<vector<int>> merged;
        for (const auto& interval : intervals) {
            if (merged.empty() || merged.back()[1] < interval[0]) {
                merged.push_back(interval);
            } else {
                merged.back()[1] = max(merged.back()[1], interval[1]);
            }
        }
        return merged;
    }
};
```

###  738.单调递增的数字 
题目要求小于等于N的最大单调递增的整数，那么拿一个两位的数字来举例。
例如：98，一旦出现`strNum[i - 1] > strNum[i]`的情况（非单调递增），首先想**让`strNum[i - 1]--`，然后`strNum[i]`给为9**，这样这个整数就是89，即小于98的最大的单调递增整数。
> 这一点如果想清楚了，这道题就好办了。

此时是从前向后遍历还是从后向前遍历呢？
从前向后遍历的话，遇到`strNum[i - 1] > strNum[i]`的情况，让`strNum[i - 1]`减一，但此时如果`strNum[i - 1]`减一了，可能又小于`strNum[i - 2]`。
这么说有点抽象，举个例子，数字：332，从前向后遍历的话，那么就把变成了329，此时2又小于了第一位的3了，真正的结果应该是299。
那么**从后向前遍历**，就可以重复利用上次比较得出的结果了，从后向前遍历332的数值变化为：332 -> 329 -> 299

#### **处理逻辑易错**
- 当发现 `digits[i-1] < digits[i]` 时，代码只修改了**当前低位**（设为9）和**当前高位**（减1），但**没有处理更高位的影响**
  - 高位减1后可能破坏与更前高位的单调性（例：`100` 的处理）
  - 未将**所有受影响低位**置为 `9`（例：`100` 的个位未置9）
- **示例验证**（输入 `n=100`）：
  分解 digits = [0, 0, 1]  # 个位=0, 十位=0, 百位=1
  遍历 i=1：0==0 → 跳过
  遍历 i=2：0<1 → 十位置9, 百位减1 → digits=[0,9,0]
  结果 = 0*10⁰ + 9*10¹ + 0*10² = 90  # 预期应为99

#### **数字组合计算易错**
- **错误代码**：`res += 10 * (i-1) * digits[i-1]`
- **问题**：
  - 错误使用乘法 `10*(i-1)` 而非指数 `10^(i-1)`
  - 导致计算结果完全错误（如个位贡献 `10*0*digits[0]=0`）
- **正确计算**：应使用 `pow(10, i-1) * digits[i-1]`

> 💡 **黄金法则**：需要修改数字的每一位时，优先考虑 `to_string` → 字符串处理 → `stoi` 流程，比数学运算更直观可靠！

```cpp
class Solution {
public:
    int monotoneIncreasingDigits(int n) {
        string str = to_string(n);
        int flag = str.size();
        for(int i = (str.size() - 1);i>0;i--){
            if(str[i-1] > str[i]){
                flag = i;
                str[i-1]--;
            }
        }
        for(int i = 0; i<str.size();i++){
            if(i >= flag){
                str[i] = '9';
            }
        }
        return stoi(str);
    }
};
```

###  968.监控二叉树 
其实我的思路和题解的想法已经很接近了，但是总结地没有那么到位，导致缺漏数量的情况

#### 确定遍历顺序
在二叉树中可以使用后序遍历也就是左右中的顺序，这样就可以在回溯的过程中从下到上进行推导了。
```cpp
int traversal(TreeNode* cur) {
    // 空节点，该节点有覆盖
    if (终止条件) return ;
    int left = traversal(cur->left);    // 左
    int right = traversal(cur->right);  // 右
    逻辑处理                            // 中
    return ;
}
```

#### 返回状态类型
* 0：该节点不在摄像头覆盖范围
* 1：本节点有摄像头
* 2：本节点在摄像头覆盖范围

递归的终止条件应该是遇到了空节点，此时应该返回2（有覆盖），这样就可以在叶子节点的父节点放摄像头了。
```cpp
// 空节点，该节点有覆盖
if (cur == NULL) return 2;
```
完整代码：
```cpp
class Solution {
public:
    int cmr = 0;
    int coverStat(TreeNode* root){
        if(!root)return 2;
        int left = coverStat(root->left);
        int right = coverStat(root->right);
        if(left == 2 && right == 2){
            return 0;
        }
        if(left == 0 || right == 0){
            cmr++;
            return 1;
        }
        if(left == 1 || right == 1){
            return 2;
        }
        return -1;
    }
    int minCameraCover(TreeNode* root) {
        int rcv = coverStat(root);
        if(rcv == 0)cmr++;
        return cmr;
    }
};
```