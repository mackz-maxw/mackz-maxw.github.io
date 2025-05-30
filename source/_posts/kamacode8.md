---
title: 代码随想录 | 刷题-字符串2
categories: leetcode
lang: zh-CN
---

### 151.翻转字符串里的单词
需要注意反转的过程，不能想当然
```cpp
class Solution {
public:
    void rev(char* left, char* right){
        while(left < right){
            char c = *left;
            *left = *right;
            *right = c;
            left++;
            right--;
        }
    }
    string reverseWords(string s) {
        string ss;
        string res;
        for(int i = s.size() - 1; i>=0;i--){
            if(s[i] != ' '){
                // cout<<"s[i]:"<<s[i]<<endl;
                ss.push_back(s[i]);
            }else{
                if(!ss.empty()){
                    rev(&ss[0], &ss[ss.size() - 1]);
                    res.append(ss);
                    res.push_back(' ');
                    ss = "";
                }
            }
        }
        if(!ss.empty() && ss[ss.size()-1] != ' '){
            rev(&ss[0], &ss[ss.size() - 1]);
            res.append(ss);
        }
        if(res[res.size() - 1] == ' ')res.pop_back();
        return res;
    }
};
```

### 卡码网：55.右旋转字符串
一步一步输出看结果，总算是做对了
贴一下题目：

> 字符串的右旋转操作是把字符串尾部的若干个字符转移到字符串的前面。给定一个字符串 s 和一个正整数 k，请编写一个函数，将字符串中的后面 k 个字符移到字符串的前面，实现字符串的右旋转操作。  
例如，对于输入字符串 "abcdefg" 和整数 2，函数应该将其转换为 "fgabcde"。   
输入共包含两行，第一行为一个正整数 k，代表右旋转的位数。第二行为字符串 s，代表需要旋转的字符串。  

```cpp
#include<iostream>
#include<string>
using namespace std;
int main(){
    int k;
    cin>>k;
    string s;
    cin.ignore();
    getline(cin, s);
    // cout<<"k:"<<k<<" s:"<<s<<endl;
    string ss;
    int len = s.size();
    for(int i = (len - 1); i>=(len-k);i--){
        ss.push_back(s[i]);
        s.pop_back();
    }
    // cout<<"ss:"<<ss<<" s:"<<s<<endl;
    char* left = &ss[0];
    char* right = &ss[ss.size() - 1];
    while(left < right){
        char c = *left;
        *left = *right;
        *right = c;
        left++;
        right--;
    }
    ss.append(s);
    cout<<ss;
}
```

### 28. 实现 strStr()
使用KMP算法求最长相等前后缀的过程，实际上是在：

- 后缀指针是外层循环
- 若前后缀判断不相等，前缀指针可以一直往前跳跃至上一个判断不相等的位置，直到长度计数归零或者找到相等位置。如果找到相等位置，则参考下一条将计数加一
- 重复利用之前已经判断相等的前后缀，如果一直相等，则长度每次都加一

```cpp
class Solution {
public:
    void getNext(vector<int> &nextTable, string s){
        nextTable.push_back(0);
        int j = 0;
        for(int i = 1;i<s.size();i++){
            while(j > 0 && s[i] != s[j]){
                j = nextTable[j - 1];
            }
            if(s[i] == s[j]){
                j++;
            }
            nextTable.push_back(j);
        }
    }
    int strStr(string haystack, string needle) {
        vector<int> n;
        getNext(n, needle);
        for(int i:n){
            cout<<i;
        }
        cout<<endl;
        int j = 0;
        for(int i = 0;i<haystack.size();i++){
            while(j>0 && haystack[i] != needle[j]){
                j = n[j-1];
            }
            if(haystack[i] == needle[j]){
                if(j == needle.size() - 1)return i-j;
                j++;
            }
        }
        return -1;
    }
};
```

### 459.重复的子字符串
关键是数学证明：最长相等前后缀不包含的子串的长度 可以被 字符串s的长度整除，那么不包含的子串 就是s的最小重复子串
```cpp
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        vector<int> nextT;
        nextT.push_back(0);
        int j= 0;
        for(int i = 1;i<s.size();i++){
            while(j>0 && s[i] != s[j]){
                j = nextT[j-1];
            }
            if(s[i] == s[j]){
                j++;
            }
            nextT.push_back(j);
        }
        int cnt = 0;
        for(int k = 0;k<nextT.size();k++){
            if(nextT[k] == 0){
                cnt++;
            }else{
                break;
            }
        }
        int len = s.size();
        if(nextT.back()!=0 && (len % (len - j) == 0))return true;
        return false;
    }
};
```