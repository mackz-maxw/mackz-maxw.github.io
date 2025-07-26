---
title: 代码随想录 | 刷题-回溯算法4
categories: leetcode
lang: zh-CN
---

### 491.递增子序列
题解说这里用数组做哈希，效率会更高
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void findSS(vector<int>& nums, vector<int> path, int left){
        unordered_set<int> used; // 当前层去重
        for(int i = left; i<nums.size();i++){
            if(used.find(nums[i]) != used.end())continue;
            if(path.empty()){
                path.push_back(nums[i]);
                used.insert(nums[i]);
                findSS(nums, path, i+1);
                path.pop_back();
            }else if(!path.empty() && nums[i]>=path.back()){
                // if(i>left && nums[i-1] == nums[i])continue; // 重复元素不一定相邻
                path.push_back(nums[i]);
                result.push_back(path);
                used.insert(nums[i]);
                findSS(nums, path, i+1);
                path.pop_back();
            }
        }
        return;
    }
    vector<vector<int>> findSubsequences(vector<int>& nums) {
        vector<int> path;
        findSS(nums, path, 0);
        return result;
    }
};
```

### 46.全排列

1. **索引限制错误**：
   - 使用 `left` 参数限制选择范围，导致只能生成**索引递增的子序列**而非全排列
   - 例如输入 `[1,2,3]` 时，代码只能生成 `[1,2,3]`，无法生成 `[2,1,3]` 等排列

2. **缺少元素使用状态跟踪**：
   - 排列需要从**所有未使用元素**中选择，而不仅仅是当前索引后的元素
   - 没有记录哪些元素已被使用，导致：
     - 无法重新选择左侧元素
     - 无法避免重复使用同一元素

3. **递归逻辑错误**：
   - 每次递归都从 `i+1` 开始，使得路径只能向右扩展
   - 排列需要任意顺序组合，需能回头选择左侧元素

```cpp
class Solution {
public:
    vector<vector<int>> result;
    
    void perm(vector<int>& nums, vector<int>& path, vector<bool>& used) {
        if (path.size() == nums.size()) {
            result.push_back(path);
            return;
        }
        for (int i = 0; i < nums.size(); i++) {  // 必须遍历所有元素
            if (used[i]) continue;  // 跳过已用元素
            used[i] = true;
            path.push_back(nums[i]);
            perm(nums, path, used);  // 注意：这里递归时不限制索引
            path.pop_back();
            used[i] = false;  // 回溯
        }
    }
    
    vector<vector<int>> permute(vector<int>& nums) {
        vector<int> path;
        vector<bool> used(nums.size(), false);  // 添加使用状态数组
        perm(nums, path, used);
        return result;
    }
};
```

### 47.全排列 II
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void perm(vector<int>& nums, vector<int> path, vector<bool>& used){
        if(path.size() == nums.size()){
            result.push_back(path);
            // return;
        }
        for(int i = 0; i< nums.size(); i++){
            if(used[i] == true)continue;
            // used[i - 1] == true，说明同一树枝nums[i - 1]使用过
            // used[i - 1] == false，说明同一树层nums[i - 1]使用过
            // 👇同一层使用过则跳过
            if(i>0 && nums[i] == nums[i-1] && used[i-1] == false)continue;
            path.push_back(nums[i]);
            used[i] = true;
            perm(nums, path, used);
            path.pop_back();
            used[i] = false;
        }
    }
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        vector<int> path;
        vector<bool> used(nums.size(),false);
        sort(nums.begin(), nums.end());
        perm(nums, path, used);
        return result;
    }
};
```

### 51. N皇后

#### 问题分析：

1. **列起始位置错误**：
   ```cpp
   for(int i = l; i<n; i++)  // 错误：每行都应从0开始尝试所有列
   ```

2. **赋值操作符错误**：
   ```cpp
   li[i] == 1;  // 应该是 =，不是 ==
   ```

3. **皇后位置检查逻辑错误**：
   - 检查时使用`continue`导致死循环，应记录检查结果后设置皇后位置
```cpp
class Solution {
public:
    vector<vector<string>> result;
    void solve(vector<vector<int>>& cur, int n, int row) {
        if (row == n) {
            vector<string> solution;
            for (auto& line : cur) {
                string s;
                for (int p : line) {
                    s.push_back(p == 1 ? 'Q' : '.');
                }
                solution.push_back(s);
            }
            result.push_back(solution);
            return;
        }
        for (int col = 0; col < n; col++) {  // 每行都尝试所有列
            bool valid = true;
            // 检查正上方
            for (int r = 0; r < row; r++) {
                if (cur[r][col] == 1) {
                    valid = false;
                    break;
                }
            }
            if (!valid) continue;
            // 检查左上对角线
            int r = row - 1, c = col - 1;
            while (r >= 0 && c >= 0) {
                if (cur[r][c] == 1) {
                    valid = false;
                    break;
                }
                r--;
                c--;
            }
            if (!valid) continue;
            // 检查右上对角线
            r = row - 1;
            c = col + 1;
            while (r >= 0 && c < n) {
                if (cur[r][c] == 1) {
                    valid = false;
                    break;
                }
                r--;
                c++;
            }
            if (!valid) continue;
            // 放置皇后
            vector<int> new_row(n, 0);
            new_row[col] = 1;  // 正确赋值
            cur.push_back(new_row);
            // 递归下一行（列从0开始）
            solve(cur, n, row + 1);
            // 回溯
            cur.pop_back();
        }
    }
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<int>> cur;
        solve(cur, n, 0);  // 从第0行开始
        return result;
    }
};
```