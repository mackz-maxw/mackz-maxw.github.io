---
title: 代码随想录 | 刷题-回溯算法3
categories: leetcode
lang: zh-CN
---

### 93.复原IP地址
手动写了`to_string()`和`stoi()`,发现字符串给整反了，改过来了也发现效率不如用现有函数
这题需要注意的限制条件确实很多
```cpp
class Solution {
public:
    vector<string> result;
    string pathToIp(vector<int> path){
        string ip;
        for(int i = 0;i<path.size();i++){
            ip += to_string(path[i]);
            if(i != (path.size()-1))ip.push_back('.');
        }
        return ip;
    }
    void resIp(string s, vector<int> path, int div){
        if(path.size() == 4){
            if(div == s.size())result.push_back(pathToIp(path));
            return;
        }
        for(int i = div; i<s.size();i++){
            string cur = s.substr(div, i-div+1);
            if(cur.size() > 3)continue;
            int curN = stoi(cur);
            if(cur.size()>1 && cur[0] == '0')continue;
            if(curN > 255)continue;
            path.push_back(curN);
            resIp(s, path, i+1);
            path.pop_back();
        }
    }
    vector<string> restoreIpAddresses(string s) {
        if (s.size() > 12) return result;
        vector<int> path;
        resIp(s, path, 0);
        return result;
    }
};
```

### 78. 子集
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void ssets(vector<int>& nums, vector<int> path, int left){
        for(int i = left; i < nums.size();i++){
            // if(i>left && nums[i] == nums[i-1])continue;
            path.push_back(nums[i]);
            result.push_back(path);
            ssets(nums, path, i+1);
            path.pop_back();
        }
    }
    vector<vector<int>> subsets(vector<int>& nums) {
        vector<int> path;
        result.push_back(path); // 带上空集
        ssets(nums, path, 0);
        return result;
    }
};
```

### 90.子集II
和上一篇中的40题一样，题解的used数组和`i>left`去重都可以，不过这回似乎`i>left`去重runtime也不低
```cpp
class Solution {
public:
    vector<vector<int>> result;
    void ssets(vector<int>& nums, vector<int> path, int left){
        for(int i = left; i < nums.size();i++){
            if(i>left && nums[i] == nums[i-1])continue;
            path.push_back(nums[i]);
            result.push_back(path);
            ssets(nums, path, i+1);
            path.pop_back();
        }
    }
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        vector<int> path;
        result.push_back(path); // 带上空集
        sort(nums.begin(), nums.end());
        ssets(nums, path, 0);
        return result;
    }
};
```