---
title: 代码随想录 | 刷题-哈希表2
categories: leetcode
lang: zh-CN
---

### 454. 四数相加II  
这道题目我没什么思路，看了题解是两组两组遍历，存储和的值和出现次数
```cpp
#include<unordered_map>
using namespace std;
class Solution {
public:
    int fourSumCount(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3, vector<int>& nums4) {
        unordered_map<int,int> m1;
        for(int a: nums1){
            for(int b: nums2){
                m1[a+b]++;
            }
        }
        int count = 0;
        for(int c: nums3){
            for(int d: nums4){
                int r = 0-(c+d);
                if(m1.find(r) != m1.end()){
                    count += m1[r];
                }
            }
        }
        return count;
    }
};
```

### 383. 赎金信  
```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        unordered_map<char, int> m;
        for(char c: magazine){
            m[c]++;
        }
        for(char cr: ransomNote){
            if(m.find(cr) == m.end())return false;
            m[cr]--;
            if(m[cr]<0)return false;
        }
        return true;
    }
};
```
### 15. 三数之和  
这题我的思路可能就是暴力解了，看了题解发现可以在for循环里同时利用下标和双指针来解决  
一开始想用set解决去重的问题，但是发现不排序的话去重还是有问题
这题的去重套路，初始化等要注意的地方还是很多的，这一遍刷题我也只是照着题解的意思敲了一遍，以后还是得多加练习
```cpp
#include<algorithm>
using namespace std;
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        sort(nums.begin(),nums.end());
        vector<vector<int>> v;
        for(int i = 0;i< nums.size();i++){
            // 遍历到第一个元素已经大于零，那么无论如何组合都不可能凑成三元组，可以剪枝
            if(nums[i]>0)return v;
            int left = i+1, right = nums.size()-1;
            // 错误去重第一个数方法，将会漏掉-1,-1,2 这种情况
            /*
            if (nums[i] == nums[i + 1]) {
                continue;
            }
            */
            if(i>0 && nums[i] == nums[i-1])continue;
            while(left < right && right < nums.size()){
                int sum = nums[i] + nums[left] + nums[right];
                if(sum == 0){
                    v.push_back({nums[i], nums[left], nums[right]});
                    // 去重逻辑应该放在找到一个三元组之后，对b 和 c去重
                    while(left < right && nums[left] == nums[left+1]){
                        left++;
                    }
                    while(left < right && nums[right] == nums [right-1]){
                        right--;
                    }
                    // 如果判断找到答案，双指针同时收缩
                    left++;
                    right--;
                }else if(sum < 0){
                    //否则单向收缩
                    left++;
                }else{
                    right--;
                }
            }
        }
        return v;
    }   
};
```

### 18. 四数之和  
还是看题解：

> 四数之和的双指针解法是两层for循环`nums[k] + nums[i]`为确定值，依然是循环内有left和right下标作为双指针，找出`nums[k] + nums[i] + nums[left] + nums[right] == target`的情况，三数之和的时间复杂度是 $O(n^2)$ ，四数之和的时间复杂度是 $O(n^3)$  
> 那么一样的道理，五数之和、六数之和等等都采用这种解法。  
> 对于15.三数之和双指针法就是将原本暴力 $O(n^3)$ 的解法，降为 $O(n^2)$ 的解法，四数之和的双指针解法就是将原本暴力 $O(n^4)$ 的解法，降为 $O(n^3)$ 的解法  

```cpp
#include<algorithm>
using namespace std;
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> v;
        if(nums.size()<4)return v;
        for(int i = 0;i<nums.size();i++){
            if(nums[i] >=0 && nums[i]> target)break;
            if(i>0 && nums[i-1] == nums[i])continue;
            for(int j = (i+1);j<nums.size();j++){
                // 第二级剪枝和去重，注意表达式变化
                if((nums[i] + nums[j])>=0 &&(nums[i] + nums[j])>target)break;
                if(j>(i+1) && nums[j-1] == nums[j])continue;
                int left = j+1, right = nums.size()-1;
                while(left < right){
                    long sum = (long)nums[i]+nums[j]+nums[left]+nums[right];
                    if(sum == target){
                        v.push_back({nums[i],nums[j],nums[left],nums[right]});
                        //先去重，再移动指针到下一个判定位置
                        while(left<right && nums[left] == nums[left+1])left++;
                        while(right>left && nums[right] == nums[right-1])right--;
                        right--;
                        left++;
                    }else if(sum > target){
                        right--;
                    }else{
                        left++;
                    }
                }
            }
        }
        return v;
    }
};
```