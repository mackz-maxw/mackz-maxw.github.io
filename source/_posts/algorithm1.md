---
title: 刷题-初级算法：数组
categories: leetcode
lang: zh-CN
---

刷了有快一周leetcode题目了，感觉算法这块进度偏慢，一天可能刷一两道题就是极限了。

刷的是leetcode学习栏里的初级算法选题。有时候感觉自己的记忆力真是差到一定程度了，有好两道题刷完后才发现自己在算法竞赛入门里看过，就是记不起来，结果还是用了最笨的方法...

可以预感到自己学了有些方法，例如双指针，之后还需要适应...太难了！一步一步来吧

(以下所有题目来源力扣cn官网)

## 2/25-3/1

#### LC26. 删除有序数组中的重复项

给你一个 升序排列 的数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。

由于在某些语言中不能改变数组的长度，所以必须将结果放在数组nums的第一部分。更规范地说，如果在删除重复项之后有 k 个元素，那么 nums 的前 k 个元素应该保存最终结果。

将最终结果插入 nums 的前 k 个位置后返回 k 。

不要使用额外的空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

```cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int a=0;
        int b=nums.size();
        if(b<=1) return b;
        for(int i=0;i<(b-1);i++){
            if(nums[i]!=nums[i+1]){
                nums[a]=nums[i];
                a++;
            }
        }
        nums[a]=nums[b-1];
        return a+1;
    }
};
```

这里（重新）学到了双指针的思想，熟悉了下许久未见的cpp。选cpp语言来学其实只是因为自己以前选的好几本（算法跟opencv）教程都是cpp的，虽然听说cpp面试在语言特性上挺严格的，但是感觉比起另一个我学过一点的python，这个更能展现一些语言内部的设定吧，python总感觉隐去了不少细节，对我之后代码风格不是太有利。

这道题其实还好，后面有在原值上修改的题，就一定要用`vector<int>& nums`这种传引用的方式。

在写这道题的时候发现自己各种函数都快忘完了，`.size()/memset()/.length() `都不记得了，哎。

#### 122. 买卖股票的最佳时机 II

给定一个数组 prices ，其中 prices[i] 表示股票第 i 天的价格。

在每一天，你可能会决定购买和/或出售股票。你在任何时候 最多 只能持有 一股 股票。你也可以购买它，然后在 同一天 出售。
返回 你能获得的 最大 利润 。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int day=0;
        int count=0;
        bool st=false;
        if(prices.size()==1)return 0;
        for(int i=0;i<prices.size()-1;i++){
            if(prices[i]<prices[i+1]&&i+1!=prices.size()-1&&st==false){
                day=i;
                st=true;
            }
            if(prices[i]>prices[i+1]&&st==true){
                count+=prices[i]-prices[day];
                st=false;
            }
            if(prices[i]<=prices[i+1]&&i+1==prices.size()-1){
                if(st==true)count+=prices[i+1]-prices[day];
                if(st==false)count+=prices[i+1]-prices[i];
            }
        }
        return count;
    }
};
```

做这道题的时候我想到了要在下降的最低日买，上升的最高日卖，然后就捅了if窝......不过这几个条件想合并确实有点难吧。

然后我一想到这个解决方法就乐起来了，完全没想到自己之前在教程里看过一个更简单的解法，就是求每一个前后两日差，然后求其中正数的和。其实我思考题目的时候想到了是不是可以利用每一段买入到卖出可以拆解成这段时间每天都买入卖出，但是没想到啥好的利用方法。

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        for(int i=0;i<prices.size()-1;i++){
            prices[i]=prices[i+1]-prices[i];
        }
        int pf=0;
        for(int j=0;j<prices.size()-1;j++){
            if(prices[j]>0)pf+=prices[j];
        }
        return pf;
    }
};
```

哦对，还可以代码复用一下

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int pf=0;
        for(int i=0;i<prices.size()-1;i++){
            prices[i]=prices[i+1]-prices[i];
            if(prices[i]>0)pf+=prices[i];
        }
        return pf;
    }
};
```

这样看起来就简洁多了！需要注意的是最后一天没有别的天数来减（买了不卖也是亏），在第二个循环中不用加上。

#### 48.旋转图像

给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像。

```
输入：matrix = [[1,2,3],[4,5,6],[7,8,9]]
输出：[[7,4,1],[8,5,2],[9,6,3]]
```

这题显然是有一定的规律可以利用的，否则一边一边写旋转，代码有点太多了。看到题目强调矩阵这个概念，我就想到了可以把矩阵操作一下，比如转置之类的。

第一次做这题时，我想到可以先转置，再看看怎么调整。

```
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int tri = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0+tri; j < n; j++) {
                int tr=matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = tr;
            }
            tri++;
        }
        for (int i = 0; i < n; i++) {
            reverse(matrix[i].begin(),matrix[i].end());
        }
    }
};
```

果然，转置过后，每一行reverse一下就行了。不过需要注意的是转置时遍历一个三角即可，不然是转置两次。

后来重做了一遍，我是倒过来想的，先reverse列，再转置。

```
class Solution {
public:
	void rotate(vector<vector<int>>& matrix) {
		reverse(matrix.begin(), matrix.end());
		for (int i = 0; i < matrix.size(); i++) {
			for (int j = i; j < matrix[i].size(); j++) {
				int ori = matrix[i][j];
				matrix[i][j] = matrix[j][i];
				matrix[j][i] = ori;
			}
		}
	}
};
```

这次我学聪明了，知道遍历三角可以`j = i`，少几行代码。不过这样写使用内存居然比第一次多！我把这个想法也改成用`j = 0+tri`的形式（其他没变），内存使用变成和第一次一样了。这是为什么呢？

我想去stackoverflow上问问，就去了leetcode英文站看看英文题目描述，顺手再提交了两次题目，发现内存占用又一样了...但是我用最开始的写法时间更快。好吧，看来这或许不是我现阶段能了解的问题。

