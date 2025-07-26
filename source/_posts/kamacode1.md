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
在C++中，声明数组和向量（vector）有多种方式。以下分别详细说明：

## 数组声明

### **数组（Array）的声明方式**
数组是固定大小的连续内存序列，大小在编译时确定。

1. **指定大小（不初始化）**  
   元素值未定义（随机值）：
   ```cpp
   int arr1[5];  // 5个int类型元素，值未初始化
   ```

2. **指定大小并完全初始化**  
   显式初始化所有元素：
   ```cpp
   int arr2[3] = {10, 20, 30};  // 初始化所有元素
   ```

3. **指定大小并部分初始化**  
   未显式初始化的元素设为0：
   ```cpp
   int arr3[5] = {1, 2};  // [1, 2, 0, 0, 0]
   ```

4. **自动推断大小**  
   编译器根据初始化列表确定数组大小：
   ```cpp
   int arr4[] = {1, 2, 3};  // 大小自动推断为3
   ```

5. **统一初始化（C++11）**  
   省略等号，使用花括号：
   ```cpp
   int arr5[]{4, 5, 6};  // 大小推断为3
   ```

6. **多维数组**  
   声明二维/三维数组：
   ```cpp
   int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};
   ```

---

### **向量（Vector）的声明方式**
向量是动态数组（标准库容器），大小可在运行时改变，需包含头文件 `<vector>`。

1. **空向量**  
   创建不含元素的空向量：
   ```cpp
   std::vector<int> vec1;  // 空向量
   ```

2. **指定大小**  
   创建含 `n` 个默认初始化元素的向量（如 `int` 初始化为0）：
   ```cpp
   std::vector<int> vec2(5);     // 5个0: [0, 0, 0, 0, 0]
   ```

3. **指定大小和初始值**  
   创建含 `n` 个指定值的元素：
   ```cpp
   std::vector<int> vec3(4, 100);  // 4个100: [100, 100, 100, 100]
   ```

4. **通过初始化列表（C++11）**  
   直接初始化元素值：
   ```cpp
   std::vector<int> vec4 = {7, 8, 9};  // 列表初始化
   std::vector<int> vec5{10, 20, 30};   // 直接花括号初始化
   ```

5. **通过迭代器范围**  
   复制另一个容器的部分或全部元素：
   ```cpp
   std::vector<int> src = {1, 2, 3, 4};
   std::vector<int> vec6(src.begin(), src.end());  // 复制整个src
   std::vector<int> vec7(src.begin(), src.begin() + 2);  // 复制前2个元素: [1, 2]
   ```

6. **拷贝构造**  
   复制另一个向量的内容：
   ```cpp
   std::vector<int> original = {5, 6, 7};
   std::vector<int> vec8(original);  // 拷贝: [5, 6, 7]
   ```

7. **移动构造（C++11）**  
   高效转移另一个向量的资源（原向量被置空）：
   ```cpp
   std::vector<int> temp = {9, 10};
   std::vector<int> vec9(std::move(temp));  // vec9接管资源，temp变为空
   ```

8. **多维向量**  
   声明二维向量（向量嵌套）：
   ```cpp
   std::vector<std::vector<int>> matrix = {{1, 2}, {3, 4}, {5, 6}};
   ```

---

### **关键区别**
| **特性**       | **数组**                          | **向量**                          |
|----------------|----------------------------------|----------------------------------|
| **大小**       | 固定（编译时确定）                | 动态（运行时可调整）             |
| **内存管理**   | 栈或静态内存                     | 堆内存（自动管理）               |
| **作为参数传递** | 退化为指针                       | 可安全传递（保持大小信息）       |
| **越界访问**   | 未定义行为（可能崩溃）           | 使用`at()`会抛出异常             |
| **功能扩展**   | 无内置方法                       | 支持`push_back()`、`size()`等方法|

---

### **示例代码**
```cpp
#include <iostream>
#include <vector>

int main() {
    // 数组示例
    int arr[] = {1, 2, 3};  // 自动推断大小
    std::cout << "数组大小: " << sizeof(arr)/sizeof(arr[0]) << "\n"; // 输出3

    // 向量示例
    std::vector<int> vec = {4, 5, 6};
    vec.push_back(7);  // 动态添加元素
    std::cout << "向量大小: " << vec.size();  // 输出4
    return 0;
}
```
根据需求选择：**数组**适用于固定大小且性能敏感的场景；**向量**则提供灵活性和安全性，适合大多数动态需求。

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
