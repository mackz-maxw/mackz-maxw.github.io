---
title: 代码随想录 | 刷题-单调栈2
categories: leetcode
lang: zh-CN
---

### 42. 接雨水  
求雨水高度，需要弹出当前池底值，再求两边最小：min(凹槽左边高度, 凹槽右边高度) - 凹槽底部高度
```cpp
class Solution {
public:
    int trap(vector<int>& height) {
        int water = 0;
        stack<int> st;
        st.push(0);
        for(int i = 1; i < height.size(); i++){
            while(!st.empty() && height[i] > height[st.top()]){
                int mid = st.top();
                st.pop();
                if(!st.empty()){
                    int h = min(height[i], height[st.top()]) - height[mid];
                    int w = i - st.top()-1;
                    water += h * w;
                }
            }
            st.push(i);
        }
        return water;
    }
};
```

### 84. 柱状图中最大的矩形
这题我初步想法是对于每个柱，求以它为高度的最大矩形。但是具体怎么用类似前后缀表的方法优化查询，我有点没思路。
看了题解反应过来还是要用单调栈求区间的宽和高，同时因为我们要弹出一个元素来获取左边元素的下标，为了头尾元素能顺利出栈，需要在前后都加入0
```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int> st;
        int max_h = 0;
        heights.insert(heights.begin(), 0);
        heights.push_back(0);
        st.push(0);
        for(int i = 1; i < heights.size(); i++){
            if(heights[i] >= heights[st.top()]){
                st.push(i);
            }else{
                while(!st.empty() && heights[i] < heights[st.top()]){
                    int mid_i = st.top();
                    st.pop();
                    int left_i = st.top();
                    int w = i - left_i - 1;
                    int h = heights[mid_i];
                    max_h = max(max_h, w * h);
                }
                st.push(i);
            }
        }
        return max_h;
    }
};
```