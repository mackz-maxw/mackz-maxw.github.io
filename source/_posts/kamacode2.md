---
title: 代码随想录 | 刷题-数组2
categories: leetcode
lang: zh-CN
---

### 209.长度最小的子数组
不看题解的时候我的想法是在一个大循环里使用三个循环，先移动两个指针里右边指针的位置到和大于等于target，然后移动左指针到小于等于target,再开始移动右边指针。看了题解了解到可以在两个循环里操作两个指针完成上述操作，避免到处找边界条件。
实际写的时候还是犯了一些小错误，例如sum的累加初始值，sum应该在何时减少等等，今后在更改代码的时候还是要注意细节。
```cpp
#include <climits>
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left = 0, right = 0, sum = 0, k = INT_MAX;
        for(;right<nums.size();right++){
            sum += nums[right];
            int len;
            while(sum >= target){
                len = right - left + 1;
                if(len < k)k = len;
                sum -= nums[left];
                left++;
            }
            // printf("left: %d, right: %d, len: %d, k:%d\n", left, right,len, k);
        }
        return k == INT_MAX ? 0 : k;
    }
};
```
### 59.螺旋矩阵II
看到这道题目我的第一想法是整理出移动方向和当前数字所在位置i，j的关系。尝试写了一下感觉太复杂了，看了题解发现是每圈一次循环，每次循环处理四条边
自己尝试着写了一下发现被每次循环加减的边界绕进去了，还是题解的方式比较清晰。题解强调的区间问题也要注意，不仅仅是每条边的循环条件，每次循环终止如何过渡到下次循环开始也是问题。
```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> mat;
        for(int i = 0; i<n;i++){
            vector<int> row(n, 0);
            mat.push_back(row);
        }
        int row = 0, col = 0, p = 1, c = n / 2;
        int startx = 0, starty = 0, offset = 1;
        while(c--){
            row = starty;
            col = startx;
            for(;col<(n-offset);col++){
                mat[row][col] = p;
                p++;
            }
            for(;row<(n-offset);row++){
                mat[row][col] = p;
                p++;
            }
            for(;col>startx;col--){
                mat[row][col] = p;
                p++;
            }
            for(;row>starty;row--){
                mat[row][col] = p;
                p++;
            }
            startx++;
            starty++;
            offset++;
        }
        if(n % 2 == 1)mat[n/2][n/2] = p;
        return mat;
    }
};
```