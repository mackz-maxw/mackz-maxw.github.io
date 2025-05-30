---
title: 代码随想录 | 刷题-哈希表
categories: leetcode
lang: zh-CN
---
### 数据结构

在C++中，set 和 map 分别提供以下三种数据结构，其底层实现以及优劣如下表所示：

| 集合               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| --- | --- | ---- | --- | --- | --- | --- |
| std::set           | 红黑树   | 有序     | 否               | 否           | O(log n) | O(log n) |
| std::multiset      | 红黑树   | 有序     | 是               | 否           | O(logn)  | O(logn)  |
| std::unordered_set | 哈希表   | 无序     | 否               | 否           | O(1)     | O(1)     |

std::unordered_set底层实现为哈希表，std::set 和std::multiset 的底层实现是红黑树，红黑树是一种平衡二叉搜索树，所以key值是有序的，但key不可以修改，改动key值会导致整棵树的错乱，所以只能删除和增加。

| 映射               | 底层实现 | 是否有序 | 数值是否可以重复 | 能否更改数值 | 查询效率 | 增删效率 |
| --- | --- | --- | --- | --- | --- | --- |
| std::map           | 红黑树   | key有序  | key不可重复      | key不可修改  | O(logn)  | O(logn)  |
| std::multimap      | 红黑树   | key有序  | key可重复        | key不可修改  | O(log n) | O(log n) |
| std::unordered_map | 哈希表   | key无序  | key不可重复      | key不可修改  | O(1)     | O(1)     |


std::unordered_map 底层实现为哈希表，std::map 和std::multimap 的底层实现是红黑树。同理，std::map 和std::multimap 的key也是有序的

### 242.有效的字母异位词  

基本上都是语法问题了，比如`m[c]`自动初始化（int则为0），pair的first和second都是成员变量
```cpp
#include<unordered_map>
#include<string>
using namespace std;
class Solution {
public:
    bool isAnagram(string s, string t) {
        unordered_map<char, int> m;
        for(char c: s){
            m[c]++;
        }
        for(char c: t){
            if(!m[c])return false;
            m[c]--;
            if(m[c]<0)return false;
        }
        for(pair<char, int> p: m){
            if(p.second>0)return false;
        }
        return true;
    }
};
```
### 349. 两个数组的交集

绞尽脑汁想半天如果结果有重复数字怎么办，结果是去重的。。。以后看题要更仔细呀！  
题解里使用set的特性去重也挺巧思的
```cpp
#include<unordered_set>
using namespace std;
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> m(nums1.begin(), nums1.end());
        unordered_set<int> r;
        for(int n: nums2){
            if(m.find(n)!=m.end()){
                r.insert(n);
            }
        }
        return vector<int>(r.begin(),r.end());
    }
};
```
### 202. 快乐数

思路差不多了，还是有代码实现的问题，例如`to_string(int)`的用法,如何char转int等
```cpp
#include<unordered_set>
#include<string>
using namespace std;
class Solution {
public:
    bool isHappy(int n) {
        unordered_set<int> nums;
        if(n == 1)return true;
        while(nums.find(n) == nums.end()){
            nums.insert(n);
            string s = to_string(n);
            int sum = 0;
            for(char c:s){
                int num = c - '0';
                sum += num*num;
            }
            if(sum == 1)return true;
            n = sum;
        }
        return false;
    }
};
```
### 1. 两数之和

看到题目例子的时候我就觉得数组如果有两个一样的值，用哈希表就比较麻烦。先存数组所有值再判断会出现索引相等的问题。使用一个循环，先判断是否已有互补值，再存当前值进map，可以避免配对同一个元素
```cpp
#include<unordered_map>
using namespace std;
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> m;
        vector<int> r;
        for(int i = 0; i< nums.size();i++){
            int v = target - nums[i];
            if(m.find(v) != m.end()){
                r.push_back(i);
                r.push_back(m[v]);
                return r;
            }
            m[nums[i]] = i;
        }
        return r;
    }
};
```