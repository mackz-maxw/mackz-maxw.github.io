---
title: 代码随想录 | 刷题-回溯算法2
categories: leetcode
lang: zh-CN
---

### 39. 组合总和 
```cpp
#include<algorithm>
using namespace std;
class Solution {
public:
    vector<vector<int>> result;
    void combSum(vector<int> path, int left, vector<int>& candi, int t){
        if(t == 0){
            result.push_back(path);
            return;
        }else if(candi[0] > t){
            return;
        }
        for(int i = left;i< candi.size();i++){
            if(candi[i] <= t){
                path.push_back(candi[i]);
                combSum(path, i, candi, t-candi[i]);
                path.pop_back();
            }else{
                break;
            }
        }
        return;
    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) { // candidate不是递增的
        sort(candidates.begin(), candidates.end());
        vector<int> path;
        combSum(path, 0, candidates, target);
        return result;
    }
};
```

#### ✅ **Introsort**（IntroSort）

C++ 中的 `std::sort()`（定义在 `<algorithm>` 中）默认使用的是一种由 **三种排序算法组合而成的混合排序算法**

#### 🔧 Introsort 的组成：

1. **快速排序（Quicksort）**：在数据量大、分布均匀时使用，速度快，期望时间复杂度是 O(n log n)。
    
2. **堆排序（Heapsort）**：当递归深度超过一定阈值（防止最坏情况 O(n²)）时退化为堆排序，保证最坏时间复杂度也是 O(n log n)。
    
3. **插入排序（Insertion Sort）**：在小数据范围（如 <16）时使用，性能优于快速排序。


### 40.组合总和II 
`i > left && candi[i] == candi[i-1]`和题解中创建candidate长度的`i > 0 && candidates[i] == candidates[i - 1] && used[i - 1] == false`都可以达到剪去重复，代码AC的效果
如果使用`i > left && candidates[i] == candidates[i - 1] && used[i - 1] == false`可以提高代码效率

注意题解中used数组只用来剪去同一层的重复数字，所以递归调用后`used[i]`要设置回`false`
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void comb(vector<int>& candi, int left, vector<int> path, int t){
        if(t == 0){
            result.push_back(path);
            return;
        }else if(t < 0){
            return;
        }
        for(int i = left; i< candi.size();i++){
            if(candi[i] > t)return;
            if(i > left && candi[i] == candi[i-1])continue;
            path.push_back(candi[i]);
            comb(candi, i+1, path, t-candi[i]);
            path.pop_back();
        }
        return;
    }
    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<int> path;
        comb(candidates, 0, path, target);
        return result;
    }
};
```

### 131.分割回文串  
如题解所说，难点在于认识到切割问题和组合问题类似——每个位置是否“切”——每种切法就是一条树枝，还有求是否是回文串部分的优化
```cpp
class Solution {
public:
    vector<vector<bool>> pal;
    void isPal(string s){
        pal.resize(s.size(), vector<bool>(s.size(), false));
        for(int i = (s.size()-1); i>=0;i--){
            for(int j = i; j<= (s.size()-1);j++){
                if(j == i){
                    pal[i][j] = true;
                }else if((j - i) == 1){
                    pal[i][j] = s[i] == s[j] ? true : false;
                }else{
                    pal[i][j] = (s[i] == s[j] && pal[i+1][j-1]);
                }
            }
        }
    }
    vector<vector<string>> result;
    void partPal(vector<string> path, string s, int div){
        if(div == s.size()){
            result.push_back(path);
            return;
        }
        for(int i = div; i < s.size();i++){
            if(pal[div][i] == true){
                path.push_back(s.substr(div, i-div+1));
                partPal(path, s, i+1);
                path.pop_back();
            }else{
                continue;
            }
        }
        return;
    }
    vector<vector<string>> partition(string s) {
        vector<string> path;
        isPal(s);
        partPal(path, s, 0);
        return result;
    }
};
```