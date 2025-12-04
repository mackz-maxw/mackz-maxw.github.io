---
title: 代码随想录 | 刷题-动态规划13
categories: leetcode
lang: zh-CN
---

### 647. 回文子串 
首先是找到递推关系，对于字符串中下标i-j的字串，如果`s[i] == s[j]`则可以由`dp[i+1][j-1]`推出`dp[i][j]`
然后是遍历顺序，因为要先知道`dp[i+1][j-1]`，所以要从下往上，从左至右遍历
```cpp
class Solution {
public:
    int countSubstrings(string s) {
        vector<vector<bool>> dp(s.size(),vector<bool>(s.size(), false));
        int cnt = 0;
        for(int i = s.size()-1; i >= 0; i--){
            for(int j = i; j < s.size(); j++){
                if(s[i] != s[j]){
                    dp[i][j] = false;
                }else{
                    if(j - i <= 1){
                        dp[i][j] = true;
                        cnt++;
                    }else{
                        dp[i][j] = dp[i+1][j-1];
                        if(dp[i][j] == true)cnt++;
                    }
                }
            }
        }
        return cnt;
    }
};
```

### 516.最长回文子序列
和上一题思路类似
```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        vector<vector<int>>dp(s.size(), vector<int>(s.size(),0));
        for(int i = s.size()-1; i >= 0; i--){
            for(int j = i; j < s.size(); j++){
                if(s[i] != s[j]){
                    dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
                }else{
                    if(i == j){
                        dp[i][j] = 1;
                    }else if(i - j == 1){
                        dp[i][j] = 2;
                    }else{
                        dp[i][j] = dp[i+1][j-1]+2;
                    }
                }
            }
        }
        return dp[0][s.size()-1];
    }
};
```