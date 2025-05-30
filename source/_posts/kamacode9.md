---
title: 代码随想录 | 刷题-栈与队列
categories: leetcode
lang: zh-CN
---

### 232.用栈实现队列
我的想法是一旦pop或者peek就把数据导到输出栈，输出后再一个一个push回去。看了题解有更简捷的方式：

> 在push数据的时候，只要数据放进输入栈就好，但在pop的时候，操作就复杂一些，输出栈如果为空，就把进栈数据全部导入进来（注意是全部导入），再从出栈弹出数据，如果输出栈不为空，则直接从出栈弹出数据就可以了。
最后如何判断队列为空呢？如果进栈和出栈都为空的话，说明模拟的队列为空了。

```cpp
#include<stack>
using namespace std;
class MyQueue {
private:
    stack<int> inStack;
    stack<int> outStack;
public:
    MyQueue() {
        
    }
    
    void push(int x) {
        inStack.push(x);
    }
    
    int pop() {
        if(outStack.empty()){
            while(!inStack.empty()){
                outStack.push(inStack.top());
                inStack.pop();
            }
        }
        int r = outStack.top();
        outStack.pop();
        return r;
    }
    
    int peek() {
        int r = this->pop();
        outStack.push(r);
        return r;
    }
    
    bool empty() {
        return inStack.empty() && outStack.empty();
    }
};
```

### 225. 用队列实现栈
用一个队列实现的方法比较简洁。注意队列和栈两种数据结构的函数用法不一样
```cpp
#include<queue>
using namespace std;
class MyStack {
private:
    queue<int> q;
public:
    MyStack() {
        
    }
    
    void push(int x) {
        q.push(x);
    }
    
    int pop() {
        int s = q.size() - 1;
        while(s>0){
            s--;
            int a = q.front();
            q.pop();
            q.push(a);
        }
        int r = q.front();
        q.pop();
        return r;
    }
    
    int top() {
        int r = this->pop();
        q.push(r);
        return r;
    }
    
    bool empty() {
        return q.empty();
    }
};
```

### 20. 有效的括号
注意一下几种可能遇到的错误情况即可
```cpp
#include<stack>
using namespace std;
class Solution {
public:
    bool isValid(string s) {
        stack<char> pairP;
        for(char c: s){
            if(c == '(' || c == '[' || c=='{'){
                pairP.push(c);
            }else if(c == ')' || c == ']' || c == '}'){
                if(pairP.empty())return false;
                char lPair = pairP.top();
                if( (c == ')' && lPair == '(') || (c == ']' && lPair == '[')||(c == '}' && lPair == '{')){
                    pairP.pop();
                }else{
                    return false;
                }
            }
        }
        if(!pairP.empty())return false;
        return true;
    }
};
```

### 1047. 删除字符串中的所有相邻重复项
有了前面的经验，这一题就相对简单了
```cpp
class Solution {
public:
    string removeDuplicates(string s) {
        stack<char> adjStack;
        string r;
        for(char c: s){
            if(!adjStack.empty() && adjStack.top() == c){
                adjStack.pop();
            }else{
                adjStack.push(c);
            }
        }
        while(!adjStack.empty()){
            char c = adjStack.top();
            adjStack.pop();
            r.push_back(c);
        }
        if(r.empty())return r;
        int left = 0;
        int right = r.size() - 1;
        while(left<right){
            char e = r[left];
            r[left] = r[right];
            r[right] = e;
            left++;
            right--;
        }
        return r;
    }
};
```