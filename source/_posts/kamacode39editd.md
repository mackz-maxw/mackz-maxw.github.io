---
title: 代码随想录 | 刷题-动态规划12
categories: leetcode
lang: zh-CN
---

### 115.不同的子序列
这题我想到要对于t的每个字符，一在s里匹配到，就从两串的下一个字符开始往后匹配。这么一看感觉开始递归了，不太动态规划。

题解的递推公式如下：
这一类问题，基本是要分析两种情况
- `s[i - 1]` 与 `t[j - 1]`相等
- `s[i - 1]` 与 `t[j - 1]` 不相等

当`s[i - 1]` 与 `t[j - 1]`相等时，`dp[i][j]`可以有两部分组成。
- 一部分是用`s[i - 1]`来匹配，那么个数为`dp[i - 1][j - 1]`。即不需要考虑当前s子串和t子串的最后一位字母，所以只需要 `dp[i-1][j-1]`。
- 一部分是不用`s[i - 1]`来匹配，个数为`dp[i - 1][j]`。
当`s[i - 1]` 与 `t[j - 1]`不相等时，`dp[i][j]`只有一部分组成，不用`s[i - 1]`来匹配（就是模拟在s中删除这个元素），即：`dp[i - 1][j]`

注意上限长度且所有字符一样的s和t，会使得dp超`long long`的限制，此时使用`uint64_t`
```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        vector<vector<uint64_t>> dp(t.size()+1, vector<uint64_t>(s.size()+1,0));
        // dp[0][0] = 1;
        for(int i=0; i<s.size();i++){
            if(s[i] == t[0])dp[0][i] = 1;
        }
        for(int i=1; i<=t.size(); i++){
            for(int j=1; j<=s.size(); j++){
                if(s[j-1] == t[i-1]){
                    dp[i][j] = dp[i-1][j-1] + dp[i][j-1];
                    // if(i == 1)dp[i][j] = max(dp[i][j], 1);
                }else{
                    dp[i][j] = dp[i][j-1];
                }
            }
        }
        // for(auto c:dp){
        //     for(int c_i : c){
        //         cout<<c_i<<' ';
        //     }
        //     cout<<endl;
        // }
        return dp[t.size()][s.size()];
    }
};
```

### 583. 两个字符串的删除操作
和上一题有些类似的思路
```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size()+1, vector<int>(word2.size()+1,0));
        for(int i = 1; i <= word1.size(); i++)dp[i][0] = i;
        for(int j = 1; j <= word2.size(); j++)dp[0][j] = j;
        for(int i = 1; i <= word1.size(); i++){
            for(int j = 1; j <= word2.size(); j++){
                if(word1[i-1] == word2[j-1]){
                    dp[i][j] = dp[i-1][j-1];
                }else{
                    dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + 1;
                    dp[i][j] = min(dp[i][j], dp[i-1][j-1]+2);
                }
            }
        }
        // for(auto line : dp){
        //     for(auto i : line){
        //         cout<<i<<' ';
        //     }
        //     cout<<endl;
        // }
        return dp[word1.size()][word2.size()];
    }
};
```

### 72. 编辑距离
跟上两题差不多的思路，差点就写对了
```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size()+1, vector<int>(word2.size()+1, 0));
        for(int i = 0; i < word1.size(); i++)dp[i+1][0] = (i+1);
        for(int i = 0; i < word2.size(); i++)dp[0][i+1] = (i+1);
        // dp[0][0] = 1;
        for(int i = 1; i <= word1.size(); i++){
            for(int j = 1; j <= word2.size(); j++){
                if(word1[i-1] != word2[j-1]){
                    dp[i][j] = min(dp[i-1][j]+1, dp[i][j-1]+1);
                    dp[i][j] = min(dp[i][j], dp[i-1][j-1]+1);
                }else{
                    dp[i][j] = dp[i-1][j-1];
                }
            }
        }

        // for(auto line : dp){
        //     for(int i : line){
        //         cout << i << ' ';
        //     }
        //     cout << endl;
        // }
        return dp[word1.size()][word2.size()];
    }
};
```