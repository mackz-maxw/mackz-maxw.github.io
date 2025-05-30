---
title: 代码随想录 | 刷题-栈与队列2
categories: leetcode
lang: zh-CN
---

### 150. 逆波兰表达式求值
细节很重要，注意数字可能有多位，有负数；数学表达式操作前后数字顺序影响结果
```cpp
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<int> st;
        for(string ss: tokens){
            char ssEnd = ss[ss.size()-1];
            if(ssEnd <= '9' && ssEnd >= '0'){
                int val = 0;
                int times = 1;
                while(!ss.empty()){
                    if(ss[ss.size()-1] == '-'){
                        val = 0 - val;
                    }else{
                        int n = (ss[ss.size()-1] - '0');
                        val += n * times;
                        times = times * 10;
                    }
                    ss.pop_back();
                }
                st.push(val);
            }else{
                int n1 = st.top();
                st.pop();
                int n2 = st.top();
                st.pop();
                int opVal = 0;
                if(ssEnd == '+'){
                    opVal = n2 + n1;
                }else if(ssEnd == '-'){
                    opVal = n2 - n1;
                }else if(ssEnd == '*'){
                    opVal = n2 * n1;
                }else{
                    opVal = n2 / n1;
                }
                st.push(opVal);
            }
        }
        return st.top();
    }
};
```

### 239. 滑动窗口最大值
注意单调栈判断条件；top查询应在代码逻辑最后
```cpp
class MyQueue{
private:
    deque<int> dq;
public:
    MyQueue(){}
    void push(int val){
        while(!dq.empty() && val > dq.back()){
            dq.pop_back();
        }
        dq.push_back(val);
    }
    void pop(int val){
        if(!dq.empty() && dq.front() == val){
            dq.pop_front();
        }
    }
    int top(){
        return dq.front();
    }
};
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        MyQueue mq;
        vector<int> maxWin;
        for(int i = 0; i<nums.size();i++){
            mq.push(nums[i]);
            if(i>(k-1)){
                mq.pop(nums[i-k]);
            }
            if(i>=(k-1)){
                maxWin.push_back(mq.top());
            }
        }
        return maxWin;
    }
};
```

### 347.前 K 个高频元素
注意operator()一定要是public的，因为小顶堆每次弹出堆顶最小元素所以最后要逆序填入vector  
为什么priority_queue第三个参数要传入一个类？priority_queue 只需要知道这个类的类型（模板参数），它会自己构造一个比较器对象myComp comp 来比较元素
```cpp
class Solution {
public:
    class myComp{
        public:
        bool operator()(const pair<int, int>& lhs, const pair<int, int>&rhs){
            return lhs.second > rhs.second;// 插入元素判断为true时走入叶子节点，所以是小顶堆
        }
    };
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> mp;
        for(int n:nums){
            mp[n]++;
        }
        priority_queue<pair<int,int>, vector<pair<int,int>>, myComp> priQue;
        for(unordered_map<int, int>::iterator p = mp.begin();p!=mp.end();p++){
            priQue.push(*p);
            if(priQue.size()>k)priQue.pop();
        }
        vector<int> res(k);
        for(int i = k-1;i>=0;i--){
            res[i] = priQue.top().first;
            priQue.pop();
        }
        return res;
    }
};
```