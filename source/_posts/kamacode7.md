---
title: 代码随想录 | 刷题-字符串
categories: leetcode
lang: zh-CN
---

### 344.反转字符串  
双指针法
```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        int h = s.size() / 2;
        int j = s.size() - 1;
        for(int i = 0;i<h;i++){
            char c = s[i];
            s[i] = s[j];
            s[j] = c;
            j--;
        }
    }
};
```

### 541. 反转字符串II  
主要还是注意语法的学习
```cpp
class Solution {
public:
    void rev(char* left, char* right){
        while(left< right){
            char c = *left;
            *left = *right;
            *right = c;
            left++;
            right--;
        }
    }
    string reverseStr(string s, int k) {
        int end = s.size() - 1;
        int p = 0;
        int n = 0;
        while(p<end){
            n = p + k - 1;
            if(n <= end){
                rev(&s[p], &s[n]);
            }else{
                rev(&s[p], &s[end]);
            }
            p = p+ 2* k;
        }
        return s;
    }
};
```

### 卡码网：54.替换数字  
题目描述：

> 给定一个字符串 s，它包含小写字母和数字字符，请编写一个函数，将字符串中的字母字符保持不变，而将每个数字字符替换为number。  
例如，对于输入字符串 "a1b2c3"，函数应该将其转换为 "anumberbnumbercnumber"。  
对于输入字符串 "a5b"，函数应该将其转换为 "anumberb"  
输入：一个字符串 s,s 仅包含小写字母和数字字符。  
输出：打印一个新的字符串，其中每个数字字符都被替换为了number  
样例输入：a1b2c3  
样例输出：anumberbnumbercnumber  
数据范围：1 <= s.length < 10000  

主要是学习字符串输入输出

```cpp
#include<iostream>
#include<string>
#include<sstream>
using namespace std;
int main(){
    string line, s;
    getline(cin, line);
    for(char c: line){
        if(c >= '0' && c <= '9'){
            s.append("number");
        }else{
            s.push_back(c);
        }
    }
    cout<<s<<endl;
}
```