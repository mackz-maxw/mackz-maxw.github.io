---
title: 代码随想录 | 刷题-单调栈1
categories: leetcode
lang: zh-CN
---

### 739. 每日温度
维护一个栈来记录未更新的数组值
`using xx = xxxx`仅可用于为现有变量创建别名，如果数组变量名太长请创建引用
```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        stack<int> st;
        vector<int> out_v(temperatures.size(), 0);
        st.push(0);
        for(int i = 1; i < temperatures.size(); i++){
            if(temperatures[i] <= temperatures[st.top()]){
                st.push(i);
            }else{
                while(!st.empty() && temperatures[i] > temperatures[st.top()]){
                    out_v[st.top()] = i - st.top();
                    st.pop();
                }
                st.push(i);
            }
        }
        return out_v;
    }
};
```

### 496.下一个更大元素 I 
和上一题很像的思路，但是需要借助两个数组都没有重复数字的假设构造map，使答案不超时
```cpp
#include<unordered_map>
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
        unordered_map<int, int>mp;
        for(int i = 0; i < nums1.size(); i++){
            mp[nums1[i]] = i;
        }
        vector<int> ng(nums1.size(), -1);
        stack<int> st;
        st.push(0);
        for(int i = 1; i < nums2.size(); i++){
            while(!st.empty() && nums2[i] > nums2[st.top()]){
                int ntop = nums2[st.top()];
                if(mp.find(ntop) != mp.end()){
                    ng[mp[ntop]] = nums2[i];
                }
                // cout<<endl;
                st.pop();
            }
            st.push(i);
        }
        return ng;
    }
};

```

### 503.下一个更大元素II 
我想的是找到最大的元素，从下一个数开始遍历一遍，不过看题解直接遍历两遍数组即可
```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        vector<int> res(nums.size(), -1);
        // int max_n = nums[0];
        // int max_i = 0;
        // for(int i = 0; i < nums.size(); i++){
        //     if(nums[i] > max_n){
        //         max_n = nums[i];
        //         max_i = i;
        //     }
        // }
        stack<int> st;
        st.push(0);
        for(int j = 1; j < nums.size()*2; j++){
            int n_i = j % nums.size();
            while(!st.empty() && nums[n_i] > nums[st.top()]){
                res[st.top()] = nums[n_i];
                st.pop();
            }
            st.push(n_i);
        }
        return res;
    }
};
```