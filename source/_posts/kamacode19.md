---
title: 代码随想录 | 刷题-回溯算法1
categories: leetcode
lang: zh-CN
---

### 77. 组合
虽然runtime离谱地高，但是俺居然自己寻思出了怎么回溯，可喜可贺：
```cpp
class Solution {
    public:
        vector<vector<int>> res;
        void comb(vector<int> curNums, int l, int r,int k){
            if(curNums.size() == k){
                res.push_back(curNums);
                return;
            }
            // for(int i = l; i<= r;i++){// 能想明白l和r的关系我已经很不容易了
            for(int i = l; i<= r-(k-curNums.size())+1;i++){ 
                // 👆 看了题解发现我想不明白的地方（i最大取多少）应该每次循环重新算而不是代入递归，用于剪枝
                curNums.push_back(i);
                comb(curNums, i+1, r, k);
                curNums.pop_back();// 最重要的一句
            }
            return;
        }
        vector<vector<int>> combine(int n, int k) {
            vector<int> cur;
            comb(cur, 1, n, k);
            return res;
        }
};
```
这种写法的时间复杂度**不是** `O(n * 2^n)`

#### 🚩实际代码做的事情：

这段代码实现的是 **从 `1` 到 `n` 中选择 `k` 个数的所有组合**，即 **组合数 C(n, k)**。

所以：

#### ✅ 正确时间复杂度是：

$O(C(n, k) \cdot k)$

解释如下：

* 一共会生成 $C(n, k)$ 个组合，每个组合长度是 `k`
    
* 每次 `curNums` 达到长度 `k` 就复制一次 `curNums` 到结果 `res` 中
    
* 每次复制 `k` 个元素，因此时间复杂度为 $O(k)$
    
* 总时间复杂度为：
$O(C(n, k) \cdot k)$

#### ❓那为什么会看到 `O(n * 2^n)` 的说法？

这个复杂度常出现在**子集枚举**问题（power set）中，比如“所有子集”或“子集和”等问题，使用类似回溯结构但**允许选择/不选择每个元素**，即二叉树结构：

```cpp
for (int i = l; i <= r; i++) { 
   // include or skip i —> 2^n branches
}
```
选出来的每个子集的**长度**是**从1到n**的，而本题组合问题选出来的组合长度固定为**K**


### 216.组合总和III 
和上题基本思想差不多，多了一些限制条件
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void comb(vector<int> nums, int k, int l, int n){
        if(k == 0){
            if(n == 0){
                result.push_back(nums);
                return;
            }else{
                return;
            }
        }
        int r = min(9, n);
        for(int i = l; i<=r;i++){
            nums.push_back(i);
            comb(nums, k-1,i+1, n-i);
            nums.pop_back();
        }
        return;
    }
    vector<vector<int>> combinationSum3(int k, int n) {
        vector<int> nums;
        comb(nums,k,1,n);
        return result;
    }
};
```

### 17.电话号码的字母组合 
注意终止条件是索引数加一
```cpp
class Solution {
public:
    const string digToStr[10] = {
        "",
        "",
        "abc",
        "def",
        "ghi",
        "jkl",
        "mno",
        "pqrs",
        "tuv",
        "wxyz"
    };
    vector<string> result;
    void comb(string str, string digits, int ind){
        if(ind == digits.size()){
            result.push_back(str);
            return;
        }
        int d = digits[ind] - '0';
        ind++;
        string s = digToStr[d];
        for(char c : s){
            str.push_back(c);
            comb(str, digits, ind);
            str.pop_back();
        }
        return;
    }
    vector<string> letterCombinations(string digits) {
        string str;
        if(digits.empty())return result;
        comb(str, digits, 0);
        return result;
    }
};
```