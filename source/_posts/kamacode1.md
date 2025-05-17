---
title: 代码随想录 | 刷题-数组
categories: leetcode
lang: zh-CN
---

# 数组基础与算法实现

## 数组基本特性
- 数组下标从0开始
- 数组内存空间的地址是连续的
- 在C++中，二维数组在内存的空间地址是连续的

## C++常见数据类型长度
| 数据类型    | 长度（字节）               | 备注                          |
|------------|--------------------------|-------------------------------|
| char       | 1                        |                               |
| short      | 2                        |                               |
| int        | 4                        |                               |
| long       | 4或8                     | 取决于编译器和操作系统        |
| long long  | 8                        |                               |
| float      | 4                        |                               |
| double     | 8                        |                               |
| long double| 8或16                    | 取决于编译器和操作系统        |
| bool       | 1                        | 实际只使用1位                 |
| 指针类型   | 4或8                     | 取决于编译器和操作系统        |

## 算法实现

### 704. 二分查找

#### 左闭右闭区间实现
原本仅判断left == right，后来发现在处理仅两个元素，且target小于最小元素的数组时，这样写会导致left = 0同时right= -1,陷入死循环，更改判断条件后就解决了
```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        return searchid(0, nums.size()-1, target, nums);
    }
   
    int searchid(int left, int right, int target, vector<int>& nums) {
        int mid = (left + right) / 2;
        if(nums[mid] == target) {
            return mid;
        } else if(nums[mid] > target) {
            if(left >= right) return -1;
            return searchid(left, mid - 1, target, nums);
        } else {
            if(left >= right) return -1;
            return searchid(mid + 1, right, target, nums);
        }
    }
};
```

#### 左闭右开区间实现
```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        return searchid(0, nums.size(), target, nums);
    }
   
    int searchid(int left, int right, int target, vector<int>& nums) {
        int mid = (left + right - 1) / 2;
        if(nums[mid] == target) {
            return mid;
        } else if(nums[mid] > target) {
            if(left >= right) return -1;
            return searchid(left, mid, target, nums);
        } else {
            if(left >= right) return -1;
            return searchid(mid + 1, right, target, nums);
        }
    }
};
```

### 27. 移除元素
当target小于所有数时，right不断左移，left不变等于0，刚好结果应该是0。当target大于所有数时，right不会动，left不断右移直到nums.length，刚好等于结果。所以未找到时应返回left  

我的方法从数组的两端向中间遍历，虽然也是双指针方法，但是需要不少额外判断。下一次要定义快慢指针去解题，因为实际上不用把所有查找的元素移动到数组最后，所以直接跳过这些元素复制即可：
快指针：寻找新数组的元素 ，新数组就是不含有目标元素的数组
慢指针：指向更新 新数组下标的位置
```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        if(nums.empty()) return 0;
        int left = 0, right = nums.size()-1;
        while(left < right) {
            while(nums[left] != val && left < right) left++;
            while(nums[left] == val && left < right) {
                nums[left] = nums[right];
                nums[right] = val;
                right--;
            }
        }
        if(nums[left] != val) return left + 1;
        return left == 0 ? 0 : left;
    }
};
```

### 977. 有序数组的平方
这题我一开始总是想着在原数组中修改，于是想不出什么来。看了题解是新开一个数组，再左右指针遍历原数组，思路正确很快就做出来了：
```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> res(nums.size(), 0);
        int left = 0, right = nums.size()-1;
        int res_p = nums.size()-1;
        while(res_p >= 0) {
            if(abs(nums[left]) < abs(nums[right])) {
                res[res_p] = pow(nums[right], 2);
                right--;
            } else {
                res[res_p] = pow(nums[left], 2);
                left++;
            }
            res_p--;
        }
        return res;
    }
};
```

## C++常用数学函数
- `pow(x, y)`：计算x的y次方
- `sqrt(x)`：计算平方根
- `abs(x)`：计算绝对值
- `ceil(x)`：向上取整
- `floor(x)`：向下取整
- `round(x)`：四舍五入
- `max(x, y)`/`min(x, y)`：返回较大/较小值
